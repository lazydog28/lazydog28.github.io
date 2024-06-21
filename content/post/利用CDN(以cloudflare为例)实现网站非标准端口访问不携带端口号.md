---
title: "利用CDN(以cloudflare为例)实现网站非标准端口访问不携带端口号"
date: 2023-07-17T16:26:01+08:00
tags:
  - cloudflare
  - CDN
categories:
  - web
---

# 利用CDN(以cloudflare为例)实现网站非标准端口访问不携带端口号

## 前置条件

* 已有网站，运行端口为非标准端口，例如http协议端口为81而非80。
* cloudflare的账号（免费计划即可，无需付费）
* 自有域名

## Cloudflare 前置设置

Cloudflare网址： https://dash.cloudflare.com/

进入Cloudflare注册登录后点击添加站点。

![image-20230713165208275](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/image-20230713165208275.png)

按照网站提示成功添加站点后添加DNS解析记录（代理状态应设置成已代理），要使用 Cloudflare，请确保已更改权威 DNS 服务器或名称服务器。这些服务器是分配的 Cloudflare 名称服务器。

> 更改域名DNS解析服务器请前往你的域名注册商处修改，更改DNS服务器有一定延迟，正常2~24小时内完成。

![image-20230713165427952](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/image-20230713165427952.png)

**Cloudflare DNS服务器**

| 类型 | 值                        |
| :--- | :------------------------ |
| NS   | adaline.ns.cloudflare.com |
| NS   | guss.ns.cloudflare.com    |

## 网站设置

把网站运行起来，这部分不再赘述毕竟每个人网站不一样。下图为我的网站运行端口号为81，如果Cloudflare DNS设置无误，当前已经可以通过域名+端口号访问。

![1689238045441.png](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/1689238045441.png)

## Cloudflare 设置

进入Cloudflare -规则-源规则界面，点击`创建规则`

![image-20230713170844479](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/image-20230713170844479.png)

规则名称可以随意填写，字段选择主机名，运算符可以是等于或者其他运算符，我这里因为是只希望对 `meetrainbow.eu.org` 这个host生效，所以选择的是等于，值填写你的网站host。然后下方勾选重写到，值填写你的网站端口号。

> 如果你的网站是http协议网站且`Cloudflare-SSL/TLS-概述`中SSL/TLS 加密模式为灵活或者关闭那么到这一步就可以不携带端口号直接访问网站了。

![image-20230713171228980](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/image-20230713171228980.png)如果不携带端口号直接访问网站提示 SSL握手失败，请核实`Cloudflare-SSL/TLS-概述`页面是否选择了完全、完全（严格)规则，如果勾选了上述规则那么Cloudflare认为Cloudflare与你的源服务器之间的传输协议为https协议，那么传输端口为非https端口。修复该问题可以将`Cloudflare-SSL/TLS-概述`页面的规则改为关闭或者灵活。亦或修改你的网站配置文件，设置好网站证书，并且指定端口传输协议为ssl。

![image-20230713172353329](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/image-20230713172353329.png)