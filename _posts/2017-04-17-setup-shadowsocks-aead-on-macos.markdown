---
layout: post
title:  "Shadowsocks AEAD on macOS"
date:   2017-04-17 16:16:07 +0800
categories: shadowsocks
---
[AEAD][AEAD] 是 Shadowsocks 最新的协议标准，增强了应对发现和检测的强度。支持 AEAD 的客户端还没有，暂时只能使用 shadowsocks 的命令行版本使用。

原有连接流程：

`apps ->  Surge(buit-in ss module) - [old ciphers] -> remote-server -> internet`

新的连接模型：

`apps ->  Surge -> ss-local - [AEAD ciphers] -> remote-server -> internet`

因此，多出的部分实际上只是安装配置支持 AEAD 的 ss-local 客户端。步骤分为两步：

1. 安装 [Homebrew][Homebrew]（macOS 的安装包管理）
2. 安装&配置 [shadowsocks-libev][shadowsocks-libev]
3. 配置 Surge 使用 ss-local 本地代理

##  安装 Homebrew

> 注意：如果已经安装过 Homebrew，请跳过安装部分。通过运行 `brew doctor` 检查一下是否一切正常。

[Homebrew][Homebrew] 是 macOS 下必备的包管理解决方案，类似 Ubuntu 下的 `apt-get`。

Homebrew 需要 **Xcode Command Line Tools** 或 **Xcode**，如果你不是 iOS 或 macOS 的开发者就没有比较安装庞大的 Xcode，用 Xcode Command Line Tools 就够了。

### 安装 Xcode Command Line Tools

安装 Xcode Command Line Tools，在命令行中执行，按照提示下载安装：

    $ xcode-select --install

### 安装 Homebrew

安装 Homebrew，在命令行中执行：

    $ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

检查 `homebrew` 是否安装就绪，在命令行中执行：

    $ brew doctor
    Your system is ready to brew.

如果提示出现有问题，请根据提示修复。看到 `Your system is ready to brew.` 那 Homebrew 就算安装完成了。

## 安装&配置 shadowsocks-libev

shadowsocks-libev 从 3.x 版本开始才支持新的 AEAD，通过 Homebrew 安装非常简单，就一行命令：

`$ brew install shadowsocks-libev`

shadowsocks-libev 安装完成后，默认配置文件位置在 `/usr/local/etc/shadowsocks-libev.json`，编辑这个文件：

`$ nano /usr/local/etc/shadowsocks-libev.json`

修改部分配置文件内容：

    {
        "server":"remote_server",
        "server_port":993,
        "local_port":1080,
        "password":"shadowsocks_password",
        "timeout":3600,
        "method":"chacha20-ietf-poly1305",
        "fast_open":true
    }

用 Shadowsocks 服务提供方的资料，填写 `server`、`server_port` 和 `password`。method 中的 chacha20-ietf-poly1305 即是新的 AEAD 加密方式。

配置文件编辑完成后，就可以启动这个常驻的 ss-local 服务，在命令行中执行：

`$ brew services start shadowsocks-libev`

这条命令会让 shadowsocks-libev 在你登录系统的时候自启动，不需要每次开机通过命令行启动了。检测 ss-local 是否已经启动成功，通过命令行运行 `lsof -Pn -i4 | grep ss-local`。如果未来不再使用，可以通过 `brew services stop shadowsocks-libev` 关闭该服务。

## 配置 Surge 使用 ss-local 本地代理

进入 Surge 的 Dashboard，切换到 Proxy 页面，添加一个 Proxy，Name 随意（例如 local），Type 选择 `SOCKS5`，Server填写 `127.0.0.1`，端口填写 `1080`，用户名和密码为空。

将这个 Proxy 添加到一个正在使用 Proxy Group 中：双击 Proxy Group，然后将刚才创建的这个代理添加到该 Proxy Group 里。

在 Surge 的菜单中，点击 **Reload Configuration**，并在 Proxy 列表中选择刚刚创建好的 Proxy。
![](/assets/img/screenshot-2017-04-17.png)

至此，大功告成。

[AEAD]: https://shadowsocks.org/en/spec/AEAD-Ciphers.html
[shadowsocks-libev]: https://github.com/shadowsocks/shadowsocks-libev
[Homebrew]: https://brew.sh/
