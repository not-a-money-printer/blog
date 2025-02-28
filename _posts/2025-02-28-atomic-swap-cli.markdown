---
layout: post
title:  "如何通过原子交换使用比特币购买门罗币"
date:   2025-02-28 05:49:26 +0000
categories: jekyll update
---
此教程为seth for privacy的博客的翻译，但是原文中很多内容已经过时，所以进行了重写

原文链接:[https://sethforprivacy.com/archives/bitcoin-monero-atomic-swaps/](https://sethforprivacy.com/archives/bitcoin-monero-atomic-swaps/)

# 介绍：

你是否持有有不干净历史的比特币呢？或者你不希望在使用比特币时暴露自己的身份，那么将比特币原子交换为门罗币将会是你的完美选择。

去中心化的交易方式才是加密货币真正的未来。原子交换是一种不可审查的也无法被禁止的交易方式，整个过程完全匿名，因为原子交换不需要信任任何中间机构。

在经过漫长的众筹与开发后，门罗币社区开发出了它们自己的交换通道，unstoppable swap。你可以在交换过程中全程使用洋葱网络以保持完全的匿名，并且因为没有第三方介入，你也不需要提交任何身份信息。在这个教程中我会展示如何使用比特币购买门罗币，或者售卖比特币以获得门罗币。


若想要更深入的了解这个协议，可以参考以下链接：

[https://localmonero.co/knowledge/monero-atomic-swaps](https://localmonero.co/knowledge/monero-atomic-swaps)

[https://www.monerooutreach.org/stories/monero-atomic-swaps.html](https://www.monerooutreach.org/stories/monero-atomic-swaps.html)

[https://github.com/comit-network/xmr-btc-swap](https://github.com/comit-network/xmr-btc-swap)

[https://comit.network/blog/2020/10/06/monero-bitcoin/](https://comit.network/blog/2020/10/06/monero-bitcoin/)


# 准备工作：

原开发者comit network已经不再维护这个项目，现在这个项目已经转为社区维护，这里是要用到的程序的下载链接:
[https://github.com/UnstoppableSwap/core/releases](https://github.com/UnstoppableSwap/core/releases)

整个交换过程涉及到两个程序,asb和swap，这两个程序都是基于命令行运行的，如果你觉得命令行太难用可以参考下半段的图形版本教程

asb是用于提供门罗币侧的流动性的，如果你只是想要用比特币购买门罗币，你只需要了解和下载swap即可。
另外整个教程都是基于假设你的操作系统是linux,但是windows和mac应该区别不大

下载swap程序:

1.在github下载最新的swap二进制文件，[https://github.com/UnstoppableSwap/core/releases](https://github.com/UnstoppableSwap/core/releases)
我这里下载的是`swap_1.0.0-rc.13_Linux_x86_64.tar`

2.解压文件

`tar xvf swap_1.0.0-rc.13_Linux_x86_64.tar`

3.测试一下swap能否正常运行

`./swap --version`


# 进行交易：
1.通过rendezvous point来获取卖家列表（rendezvous point你可以理解为一个卖家可以张贴自己广告的广告牌），并选择你的卖家：

`./swap list-sellers --rendezvous-point <swap_point>`

这是gui程序使用的rendezvous point:

`/dns4/discover.unstoppableswap.net/tcp/8888/p2p/12D3KooWA6cnqJpVnreBVnoro8midDL9Lpzmg8oJPoAGi7YYaamE`

`/dns4/eratosthen.es/tcp/7798/p2p/12D3KooWAh7EXXa2ZyegzLGdjvj1W4G3EXrTGrf6trraoT1MEobs`

这个命令会显示正在售卖门罗币的卖家，以及他们的交易额度和价格

你需要记录下卖家的网络地址

![seller_network_address](/blog/assets/monero-atomic-swap/seller.png)

2.开始交易：
`./swap buy-xmr --receive-address <你的门罗币地址> --change-address <你的比特币退款地址> --seller <卖家网络地址>`

如果出现权限问题例如获取lock失败直接sudo运行即可解决

你要在seller参数后输入你刚才获取到的卖家网络地址

![trade_init](/blog/assets/monero-atomic-swap/trade_init.png)

务必确保正确的输入比特币和门罗币地址

3.向入金地址打入比特币，确保你打入的比交换额度稍微多一点，这样你才有足够的手续费来完成整个交换

比特币入金地址会在交易发起成功后展示

![deposit](/blog/assets/monero-atomic-swap/deposit.png)

4.喝一杯咖啡，等待门罗币到帐，整个交换过程会通过日志呈现 ：-）

# 如果交换失败了怎么办！
如果交换失败了，尽快采取以下措施:
1.先抢救一下，尝试恢复交换:

`./swap resume --swap-id <SWAP ID>`

2.如果恢复失败，执行取消和退款指令

`./swap cancel-and-refund --swap-id <SWAP ID>`

额外信息：
交换价格从kraken交易所自动获取，asb侧的门罗币卖家会卖的稍微贵一点以获得利润

你的比特币退款地址应该是一个从没有使用过的新地址以保护你的隐私

你的比特币会在交易失败时返还，使用`--withdraw-btc`来提款

你应该在每次交易使用不同的monero subaddress来确保隐私

交易需要比特币主网确认两次，门罗币主网确认十次，所以完成交换需要一点耐心。如果你实在需要中断交换，可以使用swap的resume功能，但是最好不要这么做

# 图形版UI教程：

如果操作命令行程序对于你来说过于困难，你可以考虑使用名为Unstoppable swap的GUI程序，web端现在已经弃用。

1.在官方网站下载unstoppable swap，链接是:`https://unstoppableswap.net/`

2.运行这个程序 `./UnstoppableSwap_1.0.0-rc.11_amd64.AppImage`

3.稍等片刻，让程序下载好需要的内容以及建立洋葱网络，这个GUI程序使用自己内置的洋葱客户端，所以非常遗憾在Whonix下运行会出现tor over tor 的问题而无法使用

![gui_client](/blog/assets/monero-atomic-swap/gui_client.png)

4.在daemon准备好后就可以选择交易伙伴进行交易了，程序默认选择最便宜的交易方，如果你有别的需求可以自己更换交易方

![ready](/blog/assets/monero-atomic-swap/ready.png)

5.之后填入你的门罗币收款地址，GUI程序自带内部比特币钱包，在交换失败时会退回到这个钱包里，之后就可以开始交换了

![swap_start](/blog/assets/monero-atomic-swap/swap_start.png)

6.最后转入你的比特币，如果没有任何意外发生，那么你将会在一段时候后收到你的门罗币

![btc_deposit](/blog/assets/monero-atomic-swap/btc_deposit.png)


