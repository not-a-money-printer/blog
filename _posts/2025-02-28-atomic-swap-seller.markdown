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

下载最新版本的 Monero 二进制文件

[https://github.com/monero-project/monero/releases/latest](https://github.com/monero-project/monero/releases/latest)

{% highlight bash %}
cd ~/asb
wget https://downloads.getmonero.org/cli/monero-linux-x64-v0.18.1.0.tar.bz2
{% endhighlight %}

提取 monero-wallet-rpc 二进制文件:
{% highlight bash %}
tar xvf monero-linux-x64-v0.18.1.0.tar.bz2
sudo chmod +x monero-x86_64-linux-gnu-v0.18.1.0/monero-wallet-rpc
sudo cp monero-x86_64-linux-gnu-v0.18.1.0/monero-wallet-rpc /usr/local/bin
rm monero-linux-x64-v0.18.1.0.tar.bz2
rm -rf monero-x86_64-linux-gnu-v0.18.1.0
{% endhighlight %}

验证二进制文件是否正常工作：
`monero-wallet-rpc --version`

# 设置tor daemon:

如果您使用的是 Debian，只需运行以下命令来安装和启动 Tor：

{% highlight bash %}
sudo apt-get install tor
sudo systemctl enable tor
sudo systemctl start tor
{% endhighlight %}

如果您使用的是 Ubuntu，请按照官方文档使用 Tor 提供的仓库：

[https://support.torproject.org/apt/tor-deb-repo/](https://support.torproject.org/apt/tor-deb-repo/)

配置好 Tor 仓库后，运行以下命令来安装和启动 Tor：

{% highlight bash %}
sudo apt install tor deb.torproject.org-keyring
sudo systemctl enable tor
sudo systemctl start tor
{% endhighlight %}

如果您使用的是 CentOS/RHEL，请按照官方文档使用 Tor 提供的仓库：

[https://support.torproject.org/rpm/](https://support.torproject.org/rpm/)

配置好 Tor 仓库后，运行以下命令来安装和启动 Tor：

{% highlight bash %}
sudo yum install tor
sudo systemctl enable tor
sudo systemctl start tor
{% endhighlight %}

## 使用ufw 加固系统：

使用ufw禁止除了必要端口之外的访问，如果你不知道什么是ufw可以点击这里[https://landchad.net/ufw](https://landchad.net/ufw)

执行以下命令以设置防火墙

{% highlight bash %}
#Deny all non-explicitly allowed ports
sudo ufw default deny incoming
sudo ufw default allow outgoing

#Allow SSH access
sudo ufw allow ssh

#Allow the default ASB ports (remove the following two lines if running exclusively over Tor, as they are not needed)
sudo ufw allow 9939/tcp
sudo ufw allow 9940/tcp

#Enable UFW
sudo ufw enable
{% endhighlight %}

## 配置工具

# 设置asb的用户和相关目录

我们要为两个工具和相关目录设置专门的用户

{% highlight bash %}
#Create a system user and group to run asb and monero-wallet-rpc as
sudo addgroup --system asb
sudo adduser --system asb --home /var/lib/asb

#Create necessary directories for the asb tools
sudo mkdir /var/run/asb
sudo mkdir /var/log/asb
sudo mkdir /etc/asb

#Set permissions for new directories
sudo chown asb:asb /var/run/asb
sudo chown asb:asb /var/log/asb
sudo chown -R asb:asb /etc/asb

{% endhighlight %}

# monero-wallet-rpc systemd 配置

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

- `external_addresses`必须是一个公网或者暗网可以访问到的地址

如果你想让你的asb节点只能通过tor访问，那么你需要先启动一次asb,然后复制显示的`/onion3/`地址，将其将`external_address`更改为这个地址：

`external_addresses = ["/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9939", "/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9940"]`

如果你使用ipv4并且没有配置DNS，那么这样设置参数：

`external_addresses = ["/ip4/5.9.120.18/tcp/9939", "/ip4/5.9.120.18/tcp/9940/ws"]`

如果你使用DNS，这样配置：
external_addresses = ["/dns4/swap.sethforprivacy.com/tcp/9939", "/dns4/swap.sethforprivacy.com/tcp/9940/ws"]

如果你熟悉高级DNS配置，那么可以研究如何使用`/dnsaddr`格式，可以参考文档：[   https://github.com/multiformats/multiaddr/blob/master/protocols/DNSADDR.md](https://github.com/multiformats/multiaddr/blob/master/protocols/DNSADDR.md)

- `min_buy_btc` 决定了你可以接受的最小比特币量

- `max_buy_btc` 决定了你可以接受的最大比特币量

- `ask_spread` 决定了你的利润大小（在市价的基础上你加价的百分比），0.05即加价5%，0.1即加价10%

- `electrum_rpc_url`， 指向你自己的electrum 服务器地址，或者一个你信任的服务器地址

现在创建asb程序的配置文件，示例如下：
{% highlight bash %}
[data]
dir = "/etc/asb"

[network]
listen = ["/ip4/0.0.0.0/tcp/9939", "/ip4/0.0.0.0/tcp/9940/ws"]
rendezvous_point = "/dns4/discover.unstoppableswap.net/tcp/8888/p2p/12D3KooWA6cnqJpVnreBVnoro8midDL9Lpzmg8oJPoAGi7YYaamE"
# Example external_addresses: 
external_addresses = ["/onion3/example.onion/tcp/9939", "/onion3/example.onion/tcp/9940/ws"]

[bitcoin]
electrum_rpc_url = "ssl://electrum.blockstream.info:50002"
target_block = 3
network = "Mainnet"

[monero]
wallet_rpc_url = "http://127.0.0.1:18083/json_rpc"
network = "Mainnet"

[tor]
control_port = 9051
socks5_port = 9050

[maker]
min_buy_btc = 0.0005
max_buy_btc = 0.001
ask_spread = 0.05
price_ticker_ws_url = "wss://ws.kraken.com/"

{% endhighlight %}

最后重启systemd服务以应用配置文件

`sudo systemctl daemon-reload`

# 洋葱路由配置

