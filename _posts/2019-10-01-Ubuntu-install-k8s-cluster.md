---
layout: post
title: Ubuntu 安装k8s集群
subtitle: Ubuntu 安装k8s使用集群
date:       2019-10-01 12:00:00
author:     "Archer"
categories: 
tags:
    - ubuntu
    - k8s
---

### Ubuntu 安装k8s集群

#### 一、环境准备
- docker环境
```text
curl -fsSL get.docker.com -o get-docker.sh

sudo sh get-docker.sh --mirror Aliyun
```

```text
vim /etc/docker/daemon.json
```

```text
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com"
  ]
}
```

```text
sudo systemctl daemon-reload && sudo systemctl restart docker
```
- k8s环境
```text
apt-get update && apt-get upgrade && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  

apt-get update
apt-get install -y kubelet kubeadm kubectl
```

> 官方源：
```text
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add 
sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni
```

### 二、禁用swap
> 对于禁用 `swap` 内存，具体原因可以查看Github上的Issue: [Kubelet/Kubernetes should work with Swap Enabled](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fkubernetes%2Fkubernetes%2Fissues%2F53533)

```text
# 编辑/etc/fstab文件，注释掉引用swap的行
sudo swapoff -a
# 测试：输入top 命令，若 KiB Swap一行中 total 显示 0 则关闭成功

# 永久关闭：
sudo vim /etc/fstab
# 注释掉swap那一行,reboot
```

### 三、配置master节点cgroup driver
查看 Docker 使用的是 cgroup driver:
 ```text
docker info | grep -i cgroup
WARNING: No swap limit support
Cgroup Driver: systemd
```

而 kubelet 使用的 cgroupfs 为system，不一致故有如下修正：
```text
sudo vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

加上如下配置：
```text
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs"
```

或者：
```text
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=cgroupfs --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"
```

重启 kubelet
```text
systemctl daemon-reload
systemctl restart kubelet
```

### 四、拉取镜像

利用kubeadm查看需要拉取的镜像：
```text
kubeadm config images list
```

编写脚本 `pull.sh` 从阿里云拉取需要的镜像
```text
for i in `kubeadm config images list`; do 
  imageName=${i#k8s.gcr.io/}
  docker pull registry.aliyuncs.com/google_containers/$imageName
  docker tag registry.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
  docker rmi registry.aliyuncs.com/google_containers/$imageName
done;
```

### 五、初始化集群master
```text
kubeadm init --apiserver-advertise-address=192.168.17.132 --pod-network-cidr=10.244.0.0/16
```

配置环境：
```text
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

配置网络模型flannel:
```text
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
> 因为用的kubuctl版本是v1.16.0，所以这里kube-flannel.yml需要修改一个值
```text
# 下载 kube-flannel.yml
wget https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml

# 修改"cniVersion": "0.2.0"为 "cniVersion": "0.3.1"

# 运行
kubectl apply -f kube-flannel.yml
```


或者weavenet:
```text
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

或者calico (需要init时–pod-network-cidr=192.168.0.0/16):
```text
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

### 六、拷贝 $HOME/.kube/config 至子节点
```text
# 
mkdir -p $HOME/.kube
vim mkdir -p $HOME/.kube/config
```

### 扩展
- CentOS / RHEL / Fedora 配置k8s环境
```text
sudo bash -c 'cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF' && \
sudo setenforce 0 && \
sudo yum install -y kubelet kubeadm kubectl && \
sudo systemctl enable kubelet && sudo systemctl start kubelet
```

- 报错一：
```text
# 执行kubeadm reset后，重新kubeadm init
kubectl get nodes 
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: 
verification error" while trying to verify candidate authority certificate "kubernetes")
```

解决办法：
```text
# 重点就在 `rm -rf $HOME/.kube`
rm -rf $HOME/.kube
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```