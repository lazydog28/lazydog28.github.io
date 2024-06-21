---
title: "QQ机器人 Mirai 搭建及签名服务器"
date: 2023-07-26T23:49:32+08:00
tags:
  - tool
categories:
  - tool
---
>本文将简单介绍在Linux平台如何基于mirai搭建 QQ 机器人 以及当前必须的签名服务器
> 
> mirai开源仓库 https://github.com/mamoe/mirai
> 
> 签名服务插件开源仓库 https://github.com/cssxsh/fix-protocol-version
> 
> 签名服务开源仓库 https://github.com/fuqiuluo/unidbg-fetch-qsign
## 1. mcl启动器安装
>Mirai Console 启动器（简称 MCL）

下载地址：https://github.com/iTXTech/mcl-installer/releases
请根据自己的系统选择对应的版本，本文以Linux为例，下载 `mcl-installer-ae9f946-linux-amd64` 文件
![1690386922797.png](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/1690386922797.png)
我们这里创建文件夹 mirai 并将文件下载到该文件夹内，将文件重命名为 `mcl-installer`
```shell
mkdir mirai
cd mirai
wget https://github.com/iTXTech/mcl-installer/releases/download/ae9f946/mcl-installer-ae9f946-linux-amd64
mv mcl-installer-ae9f946-linux-amd64 mcl-installer
```
赋予刚才下载的文件可执行权限
```shell
chmod +x mcl-installer
```
执行安装
```shell
./mcl-installer
```
![1690387234459.png](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/1690387234459.png)
如果你当前系统内没有Java环境，请输入`Y`安装Java环境(有Java环境也建议安装避免出现乱七八糟的问题)
![1690387340472.png](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/1690387340472.png)
Java环境安装完成后，会让你输入是否下载 `iTXTech MCL` 这个就是我们所需的，请输入`Y`
![1690387373842.png](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/1690387373842.png)
至此我们的mcl启动器安装完成，接下来我们就可以使用mcl启动器了。
```shell
./mcl
```
> 退出mcl命令行的命令为：exit

启动器会帮你准备运行环境，下载和更新 Mirai 核心。你也可以使用启动器下载一些插件（见下文）。


## 2. mirai运行环境介绍

第一次运行 `mcl.cmd` 时会初始化运行环境。下表说明了各个文件夹的用途。

MCL 只是启动器，没有机器人功能。MCL 支持从远程仓库下载插件，并启动 Mirai Console 终端版（`mirai-console-terminal`）。

如果遇到启动器问题，请提交至 [iTXTech/mirai-console-loader](https://github.com/iTXTech/mirai-console-loader) 。

| 文件夹名称                | 用途                                |
| ------------------------- | ----------------------------------- |
| `data`                    | 存放插件的数据，一般不需要在意它们  |
| `config`                  | 存放插件的配置，可以打开并修改配置  |
| `logs`                    | 存放运行时的日志，日志默认保留 7 天 |
| `libs`                    | 存放 mirai-core 等核心库文件        |
| `plugins`                 | 存放插件                            |
| `plugin-libraries`        | 存放插件的库缓存                    |
| `plugin-shared-libraries` | 存放插件的公共库                    |
| `modules`                 | 存放启动器的拓展模块                |

> 可以在[这里](https://github.com/iTXTech/mirai-console-loader)查看 MCL 详细用法

## 3. mirai插件安装
刚刚装好的 Mirai Console 没有功能，功能将由插件提供。
### 使用 MCL 自动安装插件

### 如何安装官方插件

Mirai 官方提供两个插件：

- [chat-command](https://github.com/project-mirai/chat-command): 允许在聊天环境通过以 "/" 起始的消息执行指令（也可以配置前缀）
- [mirai-api-http](https://github.com/project-mirai/mirai-api-http)：提供 HTTP 支持，允许使用其他编程语言的插件

打开命令行 (Windows 系统在文件夹按住 Shift 单击鼠标右键，点击 "在此处打开 PowerShell")， 可以使用 MCL 自动安装这些插件，例如：

安装 mirai-api-http 的 2.x 版本：

```
./mcl --update-package net.mamoe:mirai-api-http --type plugin --channel maven-stable
```



安装 chat-command：

```
./mcl --update-package net.mamoe:chat-command --type plugin --channel maven-stable
```



注意：插件有多个频道，`--channel maven-stable` 表示使用从 `maven` 更新的 `stable`（稳定）的频道。不同的插件可能会设置不同的频道， 具体需要使用哪个频道可参考特定插件的说明 (很多插件会单独说明要如何安装它们, 因此不必过多考虑)。

详细文档：[MCL 命令行参数](https://github.com/iTXTech/mirai-console-loader/blob/master/cli.md)

### 常用的插件

- [chat-command](https://github.com/project-mirai/chat-command): 不安装此插件不能在聊天环境中执行命令
- [mirai-api-http](https://github.com/project-mirai/mirai-api-http): 提供 HTTP 支持，允许使用其他编程语言的插件
- [mirai-silk-converter](https://github.com/project-mirai/mirai-silk-converter): 可以自动将 `wav`, `mp3` 等格式转换为语音所需格式 `silk`
- [LuckPerms-Mirai](https://github.com/Karlatemp/LuckPerms-Mirai): 高级权限组插件，适合权限分配模型比较复杂的情况，并且可以提供网页UI的权限编辑器 (指令 `lp editor`)
- [mirai-login-solver-sakura](https://github.com/KasukuSakura/mirai-login-solver-sakura): 验证处理工具，主要是为了优化和方便处理各种验证码

## 安装签名服务
>当前签名服务是必须的，否则无法正常使用QQ机器人（无法登录QQ，会报错）
> 
> 签名服务是独立于mirai的，所以我们需要单独安装签名服务
> 
> 签名服务与mirai的qsign api对接需要安装插件

### 1. 安装签名服务插件
插件下载地址：https://github.com/cssxsh/fix-protocol-version/releases/
![1690388238783.png](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/1690388238783.png)
进入plugins文件夹，下载 `fix-protocol-version-1.x.xx.mirai2.jar` 文件
```shell
cd plugins
wget https://github.com/cssxsh/fix-protocol-version/releases/download/v1.9.10/fix-protocol-version-1.9.10.mirai2.jar
cd ..
```
启动 `mcl` 以初始化插件
```shell
./mcl
```
成功启动以后再退出`mcl`命令行
此时mirai文件夹内就会有`KFCFactory.json`该配置文件，我们待会需要修改该配置文件，暂时先不动。
![1690388629077.png](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/1690388629077.png)

### 2. 安装签名服务
签名服务下载地址：https://github.com/fuqiuluo/unidbg-fetch-qsign/releases
![1690388754937.png](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/1690388754937.png)
下载 `unidbg-fetch-qsign-1.x.x-onejar.zip` 压缩包并解压，我们在 `mirai` 文件夹内创建文件夹 `qsign` 
```shell
mkdir qsign
cd qsign
wget https://github.com/fuqiuluo/unidbg-fetch-qsign/releases/download/1.1.7/unidbg-fetch-qsign-1.1.7-onejar.zip
unzip unidbg-fetch-qsign-1.1.7-onejar.zip
```
解压后会有一个 `unidbg-fetch-qsign-1.1.6` 文件夹
进入`unidbg-fetch-qsign-1.1.6` 文件夹内的`txlib`文件夹，里面有多个版本的配置文件，我们选择其中一个版本，这里以`8.9.68`为例
```shell
cd unidbg-fetch-qsign-1.1.7/txlib/8.9.68
```
该文件夹内有`config.json`文件，我们需要修改该文件，将端口号port及密钥key修改为你想设置的。
![1690389170757.png](https://hermes981128.oss-cn-shanghai.aliyuncs.com/ImageBed/1690389170757.png)
修改完成后，返回`unidbg-fetch-qsign-1.1.6` 文件夹，启动服务
```shell
cd ../..
bash bin/unidbg-fetch-qsign --basePath=txlib/8.9.68
```
`basePath`指定我们刚才修改的配置文件所在的文件夹。
当前签名服务就启动了。
### 3. 修改KFCFactory.json
回到mirai文件夹，修改`KFCFactory.json`文件，删除所有配置，将以下配置粘贴进去
```json
{
  "8.9.68": {
    "base_url": "http://127.0.0.1:8082",
    "type": "fuqiuluo/unidbg-fetch-qsign",
    "key":"720521"
  }
}
```
`base_url`为签名服务的地址，`key`为签名服务的密钥，与刚才设置的密钥一致。
`type`固定设置为`fuqiuluo/unidbg-fetch-qsign`
现在所有的配置都已经完成了，我们可以启动机器人了。

