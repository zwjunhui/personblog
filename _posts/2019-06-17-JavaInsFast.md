---
layout: post
title: SpringBoot2.0集成FastDFS
date: 2019-06-17
tag: SpringBoot
---

### SpringBoot2.0集成FastDFS

**FastDFS 架构包括 Tracker server 和 Storage server。 客户端请求 Tracker server 进行文件上传、下载，通过 Tracker server 调度最终由 Storage server 完成文件上传和下载。**

**Tracker server 作用是负载均衡和调度，通过 Tracker server 在文件上传时可以根据一些策略找到 Storage server 提供文件上传服务。可以将 tracker 称为追踪服务器或调度服务器。** 

**Storage server 作用是文件存储，客户端上传的文件最终存储在 Storage 服务器上，Storageserver 没有实现自己的文件系统而是利用操作系统 的文件系统来管理文件。可以将storage称为存储服务器。**
![图片](/images/posts/fastdfs/856154-20171012180511480-692747720.png)
