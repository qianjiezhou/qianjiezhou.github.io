---
layout: post
title: nginx-quic文档
date: 2020-09-29 23:18 +0800
last_modified_at: 2020-10-01 01:08:25 +0800
tags: nginx,QUIC,HTTP3
grammar_cjkRuby: true
toc:  true
---

## 目录
一.介绍
二.安装
三.配置
四.客户端
五.故障排除
六.贡献


### 一.介绍
这是nginx对于[QUIC](https://datatracker.ietf.org/doc/html/rfc9000)/[HTTP3](https://datatracker.ietf.org/doc/html/draft-ietf-quic-http)的实验性支持。

该代码是在一个单独的[QUIC](https://hg.nginx.org/nginx-quic)分支中开发的。目前，它是基于nginx主线1.21.x上。官方会将最新的nginx版本定期合并到这个分支。该项目代码库与nginx使用相同的BSD许可证。

该代码目前处于测试阶段，不应用于生产。作者正在改进HTTP/3支持，目标是将其集成到主要的nginx代码库中。

现在的工作进度：

目前我们通过最终的RFC文档支持IETF-QUIC草案29。不支持较早的草案，因为它们具有不兼容的有线格式。

nginx可以通过QUIC响应HTTP/3请求，并且在上传和下载大文件时不会出错。

+ 成功完成握手
+ 一个端点可以更新秘钥并且它的对端可以正确响应
+ 0-RTT数据正在接受和处理
+ 使用TLS Resume Ticket建立连接
+ 包含重试数据包的握手成功完成
+ 流数据开始交换和确认
+ H3业务处理成功
+ 一个或两个端点将事项插入动态表，随后从标题块中引用他们
+ 版本协商数据包被发送到版本未知的客户端
+ 检测丢失的数据包并正确地重传
+ 客户端可能会迁移到新地址

尚未支持的功能：

+ [quic-recovery](https://datatracker.ietf.org/doc/html/rfc9002)中指定的显式拥塞通知（Explicit Congestion Notification,ECN）
+ 与[旋转比特位](https://www.kancloud.cn/kancloud/http3-explained/1395029)（Spin Bit）连接成功并且使值开始旋转
+ 结构化日志

由于代码是实验性的并且仍在开发中，很多工作可能无法按照预期完成，例如：
+ 流控制机制是基本的，旨在避免占用CPU，并使简单的交互成为可能
+ 并非所有协议要求都得到严格遵守；为了简化初始实现，省略了一些检查

### 二.安装

您将需要一个提供QUIC支持的[BroingSSL](https://boringssl.googlesource.com/boringssl/)库

{% highlight js linenos %}
$ hg clone -b quic https://hg.nginx.org/nginx-quic
$ cd nginx-quic
$ ./auto/configure --with-debug --with-http_v3_module       \
                   --with-cc-opt="-I../boringssl/include"   \
                   --with-ld-opt="-L../boringssl/build/ssl  \
                                  -L../boringssl/build/crypto"
$ make
{% endhighlight %}

配置nginx时，可以使用以下新配置选项启用QUIC和HTTP/3：

{% highlight js linenos %}
  --with-http_v3_module     - enable QUIC and HTTP/3
  --with-http_quic_module   - enable QUIC for older HTTP versions
  --with-stream_quic_module - enable QUIC in Stream
{% endhighlight %}
