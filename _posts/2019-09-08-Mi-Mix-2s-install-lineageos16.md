---
layout: post
title: Mi Mix 2s 安装 第三方ROM lineageos16
subtitle: '小米Mix2s安装lineageos16'
date:       2019-09-08 12:00:00
author:     "Archer"
categories: Android
tags:
    - android
    - rom
---

### Mi Mix 2s 安装 第三方ROM lineageos16

#### 一、前置条件，确认电脑安装了 `adb` 和 `fastboot`
1. 安装 `adb` 和 `fastboot` ，下载 [Windows zip](https://dl.google.com/android/repository/platform-tools-latest-windows.zip)
2. 解压 `platform-tools` 并且设置系统 `Path` 环境变量
3. 确认 `adb` 命令，链接手机，设置开发者选项 USB调试
```text
# 查看android设备
adb devices
```
4. 确认 `fastboot` 命令，手机进入 fastboot模式，输入命令：
```text
# 查看fastboot设备
fastboot devices
```

#### 二、解锁bootloader
1. 在开发者选项中，解锁状态中设备绑定小米账号（大约需要3天才能正式解锁）
2. 申请解锁，下载官方解锁工具，[小米官方解锁网站](http://en.miui.com/unlock/)
3. 手机同时按住 音量下键+电源键，进入fastboot模式，再使用小米解锁工具解锁

#### 三、fastboot模式安装第三方recovery
1. 下载 [TWRP](https://dl.twrp.me/polaris),我这里下载的是[twrp-3.3.1-0-polaris.img](https://dl.twrp.me/polaris/twrp-3.3.1-0-polaris.img.html)
2. 命令行输入 `adb reboot bootloader`，先检验设备是否进入fastboot模式，`fastboot devices`
3. 刷入 recovery 到设备
```text
fastboot flash recovery <recovery_filename>.img
```
4. 重启进入 recovery验证安装
```text
fastboot boot <recovery_filename>.img
```

#### 四、在recovery里安装LineageOS
1. 下载 [LineageOS install package](https://download.lineageos.org/polaris)
2. 可以额外下载谷歌全套，[MindTheGapps (mirror), OpenGApps](http://opengapps.org/?api=9.0&variant=nano),安装Gapps在安装lineageos后面，稍后讲详细步骤
3. 进入 fastboot ，点击 `Wipe`，点击 `Format Data` 格式化数据
4. 进入上一级菜单，点击 `Advanced Wipe`，然后选中 `Cache ` 和 `System` 分区擦除，点击 `Swipe to Wipe`
5. 在主菜单上选择 `Advanced`,`ADB Sideload`，确认开始sideload
6. 命令行 `adb sideload filename.zip`
7. 安装lineageos完成后，把下载好的Gapps包传至手机某个目录，返回主菜单，点击 `Install` ，`install zip` ，选中安装包 `Apply update`
8. 命令行 `adb reboot`，完成


#### 五、Q&A
1. **Q**:WIFI 链接后显示已连接无网络
**A**: 进入开发者选项 一律不使用`HDCP检查`，确认勾选启用WLAN详细日志记录功能，拨号 `*#*#4636#*#*` , 进入 `Wi-Fi information` --> `Wi-Fi status`,
能看到wifi的ping主机是www.google.com，国内连不上是正常的，浏览器搜索换下国内网址就可以

2. **Q**:pixel设置向导屡次停止运行
**A**: 
```text
# 命令行运行
adb root
adb remount
adb shell
pm disable com.google.android.setupwizard
```
然后进入应用权限，把所有google相关的app的权限全部打开
```text
# 新命令行运行
pm enable com.google.android.setupwizard
```
然后进入显示系统，然后在下面找到设置向导（setup wizard），打开里面两个权限
```text
am start -n com.google.android.setupwizard/.SetupWizardTestActivity
```

3. **Q**:去掉lineageos wifi叉号
**A**:
```text
adb shell "settings put global captive_portal_http_url http://captive.lineageos.org.cn/generate_204";
adb shell "settings put global captive_portal_https_url https://captive.lineageos.org.cn/generate_204";
```
然后打开飞行模式，关闭飞行模式，即可去掉叉号或叹号


