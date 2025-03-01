---
layout: post
title:  "如何在原子交换中出售门罗币（高级）"
date:   2025-02-28 05:49:26 +0000
categories: jekyll update
---

此教程为seth for privacy的博客的翻译，但是原文中很多内容已经过时，所以进行了重写，另外我删除了一些我认为的原作者过于啰嗦的部分，因为本文应该假设读者已经具有了一定的Linux基础知识，如果你不知道最基本的命令行常识，你就不应该尝试本教程，否则出现意外后找回资金对于你来说将会是一个非常痛苦的过程

原文链接:[https://sethforprivacy.com/archives/run-an-atomic-swap-provider-advanced/](https://sethforprivacy.com/archives/run-an-atomic-swap-provider-advanced/)


# 介绍：
本教程面向高级用户，如果你对于交易协议以及asb这个程序没有充分的理解，请先在testnet上进行测试。

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

- 在使用这个程序时请全程使用洋葱网络，这样可以避免交易ID与ip地址关联，尤其是当你在家里运行这个程序时。并且注意加固你的服务器，不要将非必要端口暴露与公网上。

- 如果你因为任何原因要与第三方共享日志，记得在提交前删除swap id,transaction id与ip地址等敏感信息。

- [在本地运行属于你自己的门罗币节点](https://sethforprivacy.com/guides/run-a-monero-node/)

- 如果可能，在本地运行你自己的比特币节点与[electrumX server](https://electrumx-spesmilo.readthedocs.io/en/latest/)

- 你很有可能会收到被标记的比特币，因此不要让这些比特币与你的交易所账户产生任何关联，请参考这个链接以更好的处理被污染的比特币：[Bitcoin privacy](https://bitcoiner.guide/privacy/)

# 需要的工具：

这个教程假设你已经准备好了如下内容：

- 你已经有了一个可以被做为服务器的主机（最好是一个云服务器）

- 你经设置好了主机的远程访问

- 如果要使用DNS,你已经购买并调试好了域名

- 你已经找到了可以信任的门罗币节点，或者你在本地有自己的节点。你可以使用seth for privacy 的节点：[Monero node](https://sethforprivacy.com/about/#high-performance-monero-nodes)

- 你准备好了用于出售的门罗币

- 你很熟悉该如何使用门罗币

- 你了解了比特币的隐私防护，并且熟知如何处理被污染的比特币

## 设置环境

这个教程假设你在使用Linux，但是windows和Mac区别应该不大，我这里使用了asb_1.0.0-rc.13版本。

# 第一步是准备好asb程序：

1.创建一个专用的目录:

`mkdir ~/asb`
`cd ~/asb`

2.下载程序:

访问Github下载`https://github.com/UnstoppableSwap/core/releases`

3.解压缩文件:

`tar xvf asb_1.0.0-rc.13_Linux_x86_64.tar `

`rm asb_1.0.0-rc.13_Linux_x86_64.tar `

`sudo chmod +x asb`

`sudo cp asb /usr/local/bin/`

4.运行一下程序确保能正确工作:

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

配置好 Tor 软件源后，运行以下命令来安装和启动 Tor：

{% highlight bash %}
sudo apt install tor deb.torproject.org-keyring
sudo systemctl enable tor
sudo systemctl start tor
{% endhighlight %}

如果您使用的是 CentOS/RHEL，请按照官方文档使用 Tor 提供的仓库：

[https://support.torproject.org/rpm/](https://support.torproject.org/rpm/)

配置好 Tor 软件源后，运行以下命令来安装和启动 Tor：

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

我们要为两个工具和相关目录设置专门的用户:

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

monero-wallet-rpc 是asb用于连接门罗币区块链的工具，同时也会通过这个工具管理资金，以及进行交易所需要的签名和转账。

设置这个服务最简单的方式就是复制我提供的systemd 配置文件，并将其保存在`/etc/systemd/system/monero-wallet-rpc.service`:

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

配置方式同上，创建`/etc/systemd/system/asb.service`，然后将以下内容粘贴到文件里:

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

这个配置文件决定了asb会如何运行，所以按照你想要的方式来调节参数。

这是一些你必须改变的关键参数：

- `external_addresses`必须是一个公网或者暗网可以访问到的地址

如果你想让你的asb节点只能通过tor访问，那么你需要先启动一次asb,然后复制显示的`/onion3/`地址，将其将`external_address`更改为这个地址：

`external_addresses = ["/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9939", "/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9940"]`

如果你使用ipv4并且没有配置DNS，那么这样设置参数：

`external_addresses = ["/ip4/5.9.120.18/tcp/9939", "/ip4/5.9.120.18/tcp/9940/ws"]`

如果你使用DNS，这样配置：
`external_addresses = ["/dns4/swap.sethforprivacy.com/tcp/9939", "/dns4/swap.sethforprivacy.com/tcp/9940/ws"]`

如果你熟悉高级DNS配置，那么可以研究如何使用`/dnsaddr`格式，可以参考文档：[   https://github.com/multiformats/multiaddr/blob/master/protocols/DNSADDR.md](https://github.com/multiformats/multiaddr/blob/master/protocols/DNSADDR.md)

- `min_buy_btc` 决定了你可以接受的最小比特币量

- `max_buy_btc` 决定了你可以接受的最大比特币量

- `ask_spread` 决定了你的利润大小（在市价的基础上你加价的百分比），0.05即加价5%，0.1即加价10%

- `electrum_rpc_url`， 指向你自己的electrum 服务器地址，或者一个你信任的服务器地址

这是一个asb配置文件示例，按照我刚才说来修改关键参数：
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

最后重启systemd服务以应用配置文件:

`sudo systemctl daemon-reload`

# 洋葱路由配置

为了能让asb正确的配置洋葱路由，你需要修改tor的配置文件`/etc/tor/torrc`，并且将asb加入到`debian-tor`用户组当中，然后重新启动tor。

1.使用编辑器打开`/etc/tor/torrc`，然后设置控制端口以允许asb控制洋葱路由器:

`sudo nano /etc/tor/torrc`

{% highlight bash %}
# Allow asb tool to configure hidden services
ControlPort 9051
CookieAuthentication 1
CookieAuthFileGroupReadable 1
{% endhighlight %}

2.将asb添加到`debian-tor`用户组，然后重启tor:

{% highlight bash %}
sudo adduser asb debian-tor
sudo systemctl restart tor
{% endhighlight %}

## 启动服务

# 启动monero-wallet-rpc和asb

因为已经配置过systemd服务，所以你可以使用systemd启动它们:

1.先启动`monero-wallet-rpc`

`sudo systemctl start monero-wallet-rpc`

2.然后启动asb

`sudo systemctl start asb`

# 如何重启服务

1.先重启`monero-wallet-rpc`

`sudo systemctl restart monero-wallet-rpc`

2.然后重启asb

`sudo systemctl restart asb`

## 给门罗币钱包打钱

在asb启动时它会给你一个门罗币地址，你需要向当中转入你要售卖的门罗币。

1.查询地址

`sudo grep monero_address /var/log/asb/asb.log`

2.保存地址，因为它不会被再次显示。你可以使用这个命令来生成入金地址的二维码

`qrencode "你的门罗币地址" -t ascii -o -`

如果qrencode没有被安装，你可以使用`sudo apt install qrencode`或者`sudo dnf install qrencode`来安装。

若你在初始化时忘记保存入金地址，你还可以使用以下指令向`monero-rpc-wallet`查询:

`curl http://127.0.0.1:18083/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"get_address","params":{"account_index":0,"address_index":[0]}}' -H 'Content-Type: application/json'`

3.向刚才保存的地址转账，但是请注意这个钱包没有密码保护，所以不要在里面放太多的钱

## 将你自己的asb节点公布

参考另一篇教程，下载GUI客户端，然后等待其就绪后在打开卖家列表

找到"add a new maker to public registry"，然后将你自己的网络地址和id填入其中:

![add_maker](/blog/assets/monero-atomic-swap/add_maker.png)

如果你不知道你的peer id,可以使用 `sudo grep peer_id /var/log/asb/asb.log`获得。

## 处理交易时遇到的问题

你需要时刻对日志保持关注，尤其是在头几次交易时以确保你的配置是正确的。

使用这个指令查看日志:

`sudo tail -f /var/log/asb/asb.log`

如果你在交易失败后看到了标记为`ERROR`的日志，你可以在Github对应的项目下查找相应的issue:[https://github.com/UnstoppableSwap/core](https://github.com/UnstoppableSwap/core)

如果没有相关的issue,你可以自己创建一个，并且要记得附带上以下信息:

- asb版本，使用`asb --version`获得

- 所有的和交换失败日志相邻的日志， 确保记得在发布前删除ip地址，swap id等敏感信息

- 你的操作系统信息

- 任何其它相关的信息

大部分问题都可以通过重启服务来解决，参考之前“重启服务”一节，但是在重启前最好收集日志。

## 在交易结束后提取比特币

在交易结束后，asb会将获得的比特币存储在内部钱包，当你想要提款时你需要停止asb服务，在提款后再次重启。

执行以下命令以提币:

{% highlight bash %}
sudo systemctl stop asb
asb withdraw-btc --address 你的比特币地址 --amount "提币金额 BTC"
sudo systemctl start asb
{% endhighlight %}

如果去掉amount参数，则会提走内部钱包所有的比特币:
{% highlight bash %}
sudo systemctl stop asb
asb withdraw-btc --address 你的比特币地址
sudo systemctl start asb
{% endhighlight %}

## 检查比特币与门罗币余额

检查当前两种币种余额最简单的方式就是停止服务然后运行`asb balance`指令:
{% highlight bash %}
sudo systemctl stop asb
asb balance
sudo systemctl start asb
{% endhighlight %}

你也可以在不停止asb的情况下通过monero-rpc来检查门罗币余额:

`curl http://127.0.0.1:18083/json_rpc -d '{"jsonrpc":"2.0","id":"0","method":"get_balance","params":{"account_index":0,"address_indices":[0]}}' -H 'Content-Type: application/json'`

## 升级工具

你应该时刻保持asb和monero-rpc处于最新而状态，运行以下指令以进行升级，你只需要把版本号替换成最新版本的即可。

# monero-wallet-rpc

{% highlight bash %}
cd ~/asb
wget https://downloads.getmonero.org/cli/monero-linux-x64-v0.18.3.4.tar.bz2
tar xvf monero-linux-x64-v0.18.3.4.tar.bz2
sudo chmod +x monero-x86_64-linux-gnu-v0.18.3.4/monero-wallet-rpc
sudo mv -f monero-x86_64-linux-gnu-v0.18.3.4/monero-wallet-rpc /usr/local/bin/
rm monero-linux-x64-v0.18.3.4.tar.bz2
rm -rf monero-x86_64-linux-gnu-v0.18.3.4
{% endhighlight %}

# asb

{% highlight bash %}
cd ~/asb
wget https://github.com/UnstoppableSwap/core/releases/download/1.0.0-rc.13/asb_1.0.0-rc.13_Linux_x86_64.tar
tar xvf asb_1.0.0-rc.13_Linux_x86_64.tar
rm asb_1.0.0-rc.13_Linux_x86_64.tar
sudo chmod +x asb
sudo mv -f asb /usr/local/bin/
{% endhighlight %}

## 高级配置选项

以下内容对于运行asb不是必须的，但是你可以参考：

你可以使用 /dnsaddr 格式来设置 external_address。

libp2p 是 COMIT 原子交换的网络基础架构，其中一个很酷的功能是能够使用统一地址来描述 ASB 的所有访问方式。通过一个单一地址，你可以使ASB通过IP地址，DNS和 Onionv3访问，然后你的买方就可以选择对于他们来说最方便的方式。

关于配置的详细文档请参考： [https://github.com/multiformats/multiaddr/blob/master/protocols/DNSADDR.md](https://github.com/multiformats/multiaddr/blob/master/protocols/DNSADDR.md)

要进行配置，你需要为您的域名添加TXT DNS记录，你的每一个想通过/dnsaddr宣传的地址都需要被添加。

1.通过listen和tor配置文件来配置你想要的访问方式，通常情况下使用默认listen选项，并且按照之前配置tor的教程操作即可。

2.像下面一样在域名服务商中添加包行_dnsadd的TXT记录，你需要根据你的onion地址和网络配置进行自定义。这个案例中我使用了namecheap，但是每一个域名提供商都应该会提供相似的功能。

![dnsadd_entries](/blog/assets/monero-atomic-swap/dnsaddr-entries.png)

{% highlight bash %}
dnsaddr=/ip4/5.9.120.18/tcp/9939/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW
dnsaddr=/ip4/5.9.120.18/tcp/9940/ws/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW
dnsaddr=/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9939/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW
dnsaddr=/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9940/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW
{% endhighlight %}

每一个条目都必须以`dnsaddr=`为开头，然后在结尾处像上面一样加上`/p2p/你的交易id`

3.使用dig命令验证 DNS 配置工作正常，`dig +short txt _dnsaddr.DOMAIN.NAME`应该返回相似的结果。
{% highlight bash %}
dig +short txt _dnsaddr.swap.sethforprivacy.com
"dnsaddr=/ip4/5.9.120.18/tcp/9939/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW"
"dnsaddr=/ip4/5.9.120.18/tcp/9940/ws/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW"
"dnsaddr=/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9939/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW"
"dnsaddr=/onion3/b4wfknratwn6rcpvpczs5pgtyyafedpcfjqnupr32qdfu63x6odql4id:9940/p2p/12D3KooWCPcfhr6e7V7NHoKWRxZ5zPRr6v5hGrVPhHdsftQk2DXW"
{% endhighlight %}

4.使用你注册的rendezvous point来检测你的asb节点是否能被检测为在线状态：
`./swap list-sellers --rendezvous-point /dnsaddr/swap.sethforprivacy.com/p2p/12D3KooWCULyZKuV9YEkb6BX8FuwajdvktSzmMg4U5ZX2uYZjHeu`

## 免责声明

这个软件对于我来说工作正常，并且已经可以在主网上使用，但是这个软件仍然在开发中，并且我不保障其绝对不会出错。交易理论上双方都能获得他们想要的币种，或者双方都可以得到退款，但是原子交换仍然是一个试验性交易方式。

我对于你因为各种意外导致的资金损失不负任何责任，但是我会在你遇到问题时尽力提供帮助。

## 结论

我希望这是一个很简单的如何售卖门罗币以帮助比特币卖家获得门罗币的教程，原子交换是一个非常重要的移除对于中心化交易所的信任，以及组织未来更多的政府监管的工具。所以我对于这项新技术可以运行良好而感到非常开心。

如果你有什么具体的问题需要帮助，请参考这个链接来联系我[https://sethforprivacy.com/about/#how-to-contact-me](https://sethforprivacy.com/about/#how-to-contact-me)