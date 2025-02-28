---
layout: post
title:  "如何在原子交换中出售门罗币（高级）"
date:   2025-02-28 05:49:26 +0000
categories: jekyll update
---
# 介绍：
本教程面向高级用户，如果你对于交换协议以及asb这个程序没有充分的理解，请先在testnet上进行测试。

这个教程中我将展示如果作为一个卖方，通过asb程序即automatic swap backend（自动交易后台）售卖门罗币以获得比特币。

# 背景知识：
我会尝试以一种尽可能简短的方式讲述我自己学习使用asb的经历，首先你可以参考官方文档来了解一些背景知识。

[https://github.com/comit-network/xmr-btc-swap/blob/master/docs/asb/README.md](https://github.com/comit-network/xmr-btc-swap/blob/master/docs/asb/README.md)

关于软件的具体操作细节在这一节:

[https://github.com/comit-network/xmr-btc-swap/blob/master/docs/asb/README.md#setup-details](https://github.com/comit-network/xmr-btc-swap/blob/master/docs/asb/README.md#setup-details)

若你想了解交易协议的具体内容，请参考这一片文章:
[https://comit.network/blog/2020/10/06/monero-bitcoin](https://comit.network/blog/2020/10/06/monero-bitcoin)

如果上面的文章对于你来说太长了，他的概括版本在这里：
[https://comit.network/blog/2020/10/06/monero-bitcoin/#long-story-short](https://comit.network/blog/2020/10/06/monero-bitcoin/#long-story-short)

# 隐私保护:

在使用这个程序时请全程使用洋葱网络，这样可以避免交易ID与ip地址关联，尤其是当你在家里运行这个程序时。并且注意加固你的服务器，必要暴露没有必要的端口在公网上。

如果你因为任何原因要与第三方共享，记得在提交前删除swap id,transaction id与ip地址。

[在本地运行你自己的门罗币节点](https://sethforprivacy.com/guides/run-a-monero-node/)

如果可能，在本地运行你自己的比特币节点与[electrumX server](https://electrumx-spesmilo.readthedocs.io/en/latest/)

你很有可能会收到被标记的比特币，因此不要让这些比特币与你的交易所账户产生任何关联，请参考这个链接以更好的处理被污染的比特币：[Bitcoin privacy](https://bitcoiner.guide/privacy/)

# 需要的工具：

这个教程假设你已经准备好了如下内容：

你已经有了一个主机可以被搭建为服务器（最好是一个云服务器）

你已经设置好了主机的远程访问

如果要使用DNS,你已经购买并调试好了域名

你已经找到了可以信任的门罗币节点，或者你在本地有自己的节点。你可以使用seth for privacy 的节点：[Monero node](https://sethforprivacy.com/about/#high-performance-monero-nodes)

你准备好了用于出售的门罗币

你很熟悉该如何使用门罗币

你了解了比特币的隐私防护，并且熟知如何处理被污染的比特币

## 设置环境

这个教程假设你在使用Linux，但是windows和Mac区别应该不大，我这里使用了 asb_0.11.0_Linux_x86_64版本

# 第一步是准备好asb程序：

1.创建一个专用的目录

`mkdir ~/asb`
`cd ~/asb`

2.下载程序

访问Github下载`https://github.com/comit-network/xmr-btc-swap/releases/latest`

3.解压缩文件

`tar xvf asb_0.11.0_Linux_x86_64.tar`

`rm asb_0.11.0_Linux_x86_64.tar`

`sudo chmod +x asb`

`sudo cp asb /usr/local/bin/`

4.运行一下程序确保能正确工作

`asb --version`

# 设置monero-wallet-rpc:

monero-wallet-rpc 是asb用于连接门罗币区块链的工具，同时也会通过这个工具管理资金，以及进行交易所需要的签名和转账

设置这个服务最简单的方式就是复制和保存我提供的systemd 配置文件，并将其保存在`/etc/systemd/system/monero-wallet-rpc.service`

`sudo nano /etc/systemd/system/monero-wallet-rpc.service`

如果你没有在本地运行门罗币节点，记得将127.0.0.1:18089换成你自己的节点链接，例如`node.sethforprivacy.com:18089`

{% highlight bash %}
[Unit]
Description=Monero Wallet RPC (Mainnet)
After=network.target

[Service]
# Process management
####################

Type=forking
PIDFile=/var/run/asb/monero-wallet-rpc.pid
ExecStart=/usr/local/bin/monero-wallet-rpc --pidfile /var/run/asb/monero-wallet-rpc.pid --daemon-host 127.0.0.1:18089 --rpc-bind-port 18083 --disable-rpc-login --wallet-dir /etc/asb --detach --log-file /var/log/asb/monero-wallet-rpc.log
Restart=on-failure
RestartSec=30

# Directory creation and permissions
####################################

# Run as asb:asb
User=asb
Group=asb

# /run/asb
RuntimeDirectory=asb
RuntimeDirectoryMode=0710

# /var/lib/asb
StateDirectory=asb
StateDirectoryMode=0710

# /var/log/asb
LogsDirectory=asb
LogsDirectoryMode=0710

# /etc/asb
ConfigurationDirectory=asb
ConfigurationDirectoryMode=0710

# Hardening measures
####################

# Provide a private /tmp and /var/tmp.
PrivateTmp=true

# Mount /usr, /boot/ and /etc read-only for the process.
ProtectSystem=full

# Deny access to /home, /root and /run/user
ProtectHome=true

# Disallow the process and all of its children to gain
# new privileges through execve().
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target

{% endhighlight %}

# Automated swap broker(ASB) systemd 配置文件

配置方式同上，创建`/etc/systemd/system/asb.service`，然后将以下内容粘贴到文件里

{% highlight bash %}
[Unit]
Description=Automated swap broker (ASB)
After=network.target monero-wallet-rpc.service

[Service]
# Process management
####################

Type=simple
ExecStart=/usr/local/bin/asb --config /etc/asb/config.toml start
StandardOutput=append:/var/log/asb/asb.log

# Directory creation and permissions
####################################

# Run as asb:asb
User=asb
Group=asb

# /var/log/asb
LogsDirectory=asb
LogsDirectoryMode=0710

# /etc/asb
ConfigurationDirectory=asb
ConfigurationDirectoryMode=0710

# Hardening measures
####################

# Provide a private /tmp and /var/tmp.
PrivateTmp=true

# Mount /usr, /boot/ and /etc read-only for the process.
ProtectSystem=full

# Deny access to /home, /root and /run/user
ProtectHome=true

# Disallow the process and all of its children to gain
# new privileges through execve().
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
{% endhighlight %}

# ASB配置文件

这个配置文件决定了asb会如何运行，所以按照你想要的方式来调节参数

这是一些你必须改变的关键参数：
`external_addresses`是买方