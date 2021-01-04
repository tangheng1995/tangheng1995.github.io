---
layout: post
title: "HTTP PATCH 方法"
date: "2020-12-17 12:00:00"
category: http
tags: http
author: Archer
---
* content
{:toc}

## 介绍

PATCH 非幂等，但是能以幂等形式发送请求。




## 幂等性

幂等性，根据 rfc2616(Hypertext Transfer Protocol -- HTTP/1.1) 文档第 50 页底部对 Idempotent Methods 的定义：

> Methods can also have the property of "idempotence" in that (aside
   from error or expiration issues) the side-effects of N > 0 identical
   requests is the same as for a single request. The methods GET, HEAD,
   PUT and DELETE share this property. Also, the methods OPTIONS and
   TRACE SHOULD NOT have side effects, and so are inherently idempotent.

相同的请求执行多次和执行一次的副作用是一样的。可以看出，GET，HEAD，PUT，DELETE，OPTIONS 和 TRACE 方法都是幂等的。

## PATCH

根据 rfc5789 介绍PATCH：

> The PATCH method requests that a set of changes described in the request entity be applied to the resource identified by the Request-URI.  The set of changes is represented in a format called a "patch document" identified by a media type.  If the Request-URI does not point to an existing resource, the server MAY create a new resource, depending on the patch document type (whether it can logically modify a null resource) and permissions, etc.

> PATCH 方法请求将一组描述在请求实体里的更改应用到 Request-URI 标志的资源。这组更改以称为 "补丁文档" 的格式（该格式由媒体类型标志）表示，如果 Request-URI 未指向现有资源，服务器可能根据补丁文档的类型（是否可以在逻辑上修改空资源）和权限等来创建一个新资源。

所以可以知道 PATCH 请求中的实体是一组将要应用到实体的更改，而不是像 PUT 请求那样是要替换旧资源的实体，但是这并没有解决 PATCH 方法为什么不是幂等的问题。不着急，继续读，接下来就给出了 PUT 和 PATCH 的区别：

> The difference between the PUT and PATCH requests is reflected in the
   way the server processes the enclosed entity to modify the resource
   identified by the Request-URI.  In a PUT request, the enclosed entity
   is considered to be a modified version of the resource stored on the
   origin server, and the client is requesting that the stored version
   be replaced.  With PATCH, however, the enclosed entity contains a set
   of instructions describing how a resource currently residing on the
   origin server should be modified to produce a new version.  The PATCH
   method affects the resource identified by the Request-URI, and it
   also MAY have side effects on other resources; i.e., new resources
   may be created, or existing ones modified, by the application of a
   PATCH.

> PUT 和 PATCH 请求的区别体现在服务器处理封闭实体以修改 Request-URI 标志的资源的方式。在一个 PUT 请求中，封闭实体被认为是存储在源服务器上的资源的修改版本，并且客户端正在请求替换存储的版本。而对于 PATCH 请求，封闭实体中包含了一组描述当前保留在源服务器上的资源应该如何被修改来产生一个新版本的指令。PATCH 方法影响由 Request-URI 标志的资源，而且它也可能对其他资源有副作用；也就是，通过使用 PATCH，新资源可能被创造，或者现有资源被修改。

以上就是答案。可以理解为，PATCH 请求中的实体保存的是修改资源的指令，该指令指导服务器来对资源做出修改，所以不是幂等的。

举例：

```text
PATCH /user/123

[
    {"op":"add","salary":200}
]
```

op我add操作，实际上是对用户123的工资做增加200操作，如果多次请求，则做累加200操作，所以PATCH非幂等

## PATCH 能实现幂等

根据 rfc5789 介绍：

> A PATCH request can be issued in such a way as to be idempotent,
   which also helps prevent bad outcomes from collisions between two
   PATCH requests on the same resource in a similar time frame.
   Collisions from multiple PATCH requests may be more dangerous than
   PUT collisions because some patch formats need to operate from a
   known base-point or else they will corrupt the resource.  Clients
   using this kind of patch application SHOULD use a conditional request
   such that the request will fail if the resource has been updated
   since the client last accessed the resource.  For example, the client
   can use a strong ETag [RFC2616] in an If-Match header on the PATCH
   request.

> 可以以幂等的方式发出PATCH请求，这也有助于防止两个之间的冲突产生不良结果,PATCH在相似的时间范围内请求相同资源。

举例：

```text
PATCH /user/123

[
    {"op":"replace","salary":200}
]
```

op我replace操作，实际上是对用用户123的工资做替换200操作，请求多次，用户123工资一直为200，所以PATCH能做幂等操作

## 总结

PATCH含对资源的一些“一般更改”，因此我们不仅仅应进行典型的字段替换。如果资源用于计数器，则patch可以请求其增加，这显然不是idempotet。本质在于如何描述对资源的修改。

1. PATCH 是对资源的部分修改

2. PATCH 是非幂等，但是能实现幂等操作，看具体patch document操作指令

## 引用

* [1] [rfc](https://tools.ietf.org/html/rfc5789)
* [2] [REST API - PUT与PATCH的实例](https://www.itranslater.com/qa/details/2103002937847972864)
* [3] [为什么 HTTP PATCH 方法不是幂等的及其延伸](https://juejin.cn/post/6844903813799739399)
* [4] [Use of PUT vs PATCH methods in REST API real life scenarios](https://stackoverflow.com/questions/28459418/use-of-put-vs-patch-methods-in-rest-api-real-life-scenarios/39338329#39338329)
* [5] [MDN web-PATCH](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/PATCH)
