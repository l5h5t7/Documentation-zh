# 常见问题


#### 我是否需要开放端口?

这是 __可选的__, 你无须开放端口就可以浏览和使用 ZeroNet 站点. 
如果你想创建新的站点那么强烈建议你开放端口. 

在启动时 ZeroNet 会使用 [UPnP](https://zh.wikipedia.org/wiki/UPnP) 尝试在你的路由器上开放端口, 
如果这失败了你需要手动操作:

- 尝试访问你路由器的Web界面 [http://192.168.1.1](http://192.168.1.1)
或者 [http://192.168.0.1](http://192.168.0.1)
- 寻找 "开启 UPnP 支持" 或相似的选项然后重启 ZeroNet.

如果还是不行则尝试寻找"端口转发". 这对每个路由器都是不同的. [这是一个 YouTube 上的教程.](https://www.youtube.com/watch?v=aQXJ7sLSz14) 要开放的端口是 15441.


---


#### ZeroNet 是匿名的吗？

它没有比 Bittorrent 更多的匿名性，但是匿名性（找出谁是评论/站点的所有者的可能性）将随着网络与站点或者的节点数的增长而增长。

ZeroNet 可以在匿名网络下工作：你能简单地使用 Tor 网络隐藏你的 IP 。


---


#### 如何在 Tor browser 中使用 ZeroNet?

在 Tor 模式下建议使用 Tor Browser 浏览 ZeroNet:

- 打开 Tor Browser
- 转到地址 `about:preferences#advanced`
- 点击 `设置...`
- 在 **不使用代理** 输入 `127.0.0.1`


---


#### 如何通过 Tor 使用 ZeroNet?

如果你想隐藏你的 IP ，安装最新版本的 ZeroNet 然后在 ZeroHello 点击 Tor > 为每个连接启用 Tor 。

在 Windows 下 Tor 是包含在 ZeroNet ，对于其他操作系统[跟随 Tor 安装说明](https://www.torproject.org/docs/installguide.html)，
编辑你的 torrc 配置，移除在 `# ControlPort 9051` 行的 `#` 然后重启 Tor 服务和 ZeroNet 。

> __提示：__ 你能使用 [Stats](http://127.0.0.1:43110/Stats) 页面来验证你的 IP 地址。

> __提示：__ 如果你连接错误，确信你已经安装了最新版本的 Tor 。(需要 0.2.7.5 或更高版本)


---


#### 如何在 Linux 下通过 Tor 使用 ZeroNet?

更新最新版本的 Tor (我们需要 0.2.7.5+)，跟随 [这些](https://www.torproject.org/docs/debian.html.en) 说明例如为 Debian ：

 - `echo 'deb http://deb.torproject.org/torproject.org jessie main'>> /etc/apt/sources.list.d/tor.list`
 - `gpg --keyserver keys.gnupg.net --recv 886DDD89`
 - `gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | apt-key add -`
 - `apt-get update`
 - `apt-get install tor`

编辑配置来启用控制协议：

 - `mcedit /etc/tor/torrc`
 - 移除 `ControlPort 9051` 和 `CookieAuthentication 1` 中的 `#` 符号（大约第 57 行）
 - `/etc/init.d/tor restart`
 - 通过 `usermod -a -G debian-tor [yourlinuxuser]` 添加你自己的权限来读取验证 cookie<br>（如果你不是在 Debian 下，通过 `ls -al /var/run/tor/control.authcookie` 来检查你的用户组）
 - 退出并重新登录你的用户来应用组的改变

> __提示:__ 使用 `echo 'PROTOCOLINFO' | nc 127.0.0.1 9051` 你可以验证你的 Tor 是否运行正常

> __提示:__ 通过运行 `zeronet.py --tor disable --proxy 127.0.0.1:9050 --disable_udp` ，不更改 torrc 也可以使用（或者使用更早版本的 Tor 客户端），但是你会丢失与其他 .onion 连接的可用性。



---

#### 我可以在多台机器上使用同一个用户吗?

可以, 你需要复制 `data/users.json`。


---


#### 如何注册一个 .bit 域名?

你可以使用 [Namecoin](https://namecoin.info/) 来注册 .bit 域名。
使用客户端的图形化界面或[命令行界面](http://www.christopherpoole.net/registering-a-bit-domain-with-namecoin.html)来管理你的域名。

在注册完成后你必须通过加一个 zeronet 段到里面来编辑你的记录，例如：

```
{
...
    "zeronet": {
        "": "1EU1tbG9oC1A8jz2ouVwGZyQ5asrNsE4Vr",
        "blog": "1BLogC9LN4oPDcruNz3qo1ysa133E9AGg8",
        "talk": "1TaLk3zM7ZRskJvrh3ZNCDVGXvkJusPKQ"
    },
...
}
```
"" 意味着顶级域名，任何其他的内容都是子域名。


> __提示:__ 注册 .bit 域名的其他可选项：[domaincoin.net](https://domaincoin.net/), [peername.com](https://peername.com/), [dotbit.me](https://dotbit.me/)

> __提示:__ 你可以在 [namecha.in](http://namecha.in/) 验证你的域名，例如：[zeroid.bit](http://namecha.in/name/d/zeroid)

> __提示:__ 你只能使用 [小写字母、数字与 - 在你的域名中](http://wiki.namecoin.info/?title=Domain_Name_Specification_2.0#Valid_Domains).

> __提示:__ 为使得 ZeroHello 链接到你的域名而不是你的站点地址，添加一个域名键值到你的 content.json。([例子](https://github.com/HelloZeroNet/ZeroBlog/blob/master/content.json#L6))


---


#### 我能使用生成的站点地址/私钥来接受比特币支付吗？

是的，这是一个标准的比特币地址。私钥是 WIF 格式的，所以你可以导入它到很多客户端。

> __提示:__ 不推荐保留大量钱在你的站点地址里，因为你必须在每次更改你的站点时输入你的私钥。


---


#### 当一些人托管恶意内容时会发生什么？

ZeroNet 是在沙盘中的，他们与你在互联网上访问的其他网站有者相同的权限。
你完全可以控制你想托管什么。如果你找到恶意内容，你可以在任何时候停止托管。


---


#### 有可能在远程机器上安装 ZeroNet 吗？
是的，你必须通过重命名 __plugins/disabled-UiPassword__ 目录为 __plugins/UiPassword__ 来启用 UiPassword 插件，
然后使用 `zeronet.py --ui_ip "*" --ui_password anypassword` 打开在远程机器上的。<br>
这将绑定 ZeroNet UI webserver 到所有接口，但是通过输入指定的密码来确保它只有你能访问。

> __提示:__ 你也可以使用 `--ui_restrict ip1 ip2` 来基于 IP 地址限制界面。

> __提示:__ 你可以通过创建 `zeronet.conf` 文件并加入 `[global]`, `ui_password = anypassword` 到里面来在配置文件中指定密码。


---


#### 有办法追踪 ZeroNet 的流量使用情况吗？

发送/接收的字节数显示在 ZeroNet 的侧边栏中。<br>（通过将右上角的“0”按钮拉到左边来打开它）

> __提示:__ 每个连接的统计页面：[http://127.0.0.1:43110/Stats](http://127.0.0.1:43110/Stats)


---


#### 如果两个人使用相同的私钥更改站点会发生什么？

每个 content.json 文件包含时间戳，客户端总是接受最新的那一个。


---


#### ZeroNet 是否使用了比特币的区块链?

不，ZeroNet 只使用比特币的加密学用于站点地址和内容签名/验证。
用户的身份信息也是基于比特币的 [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) 格式。

Namecoin 的区块链被用于域名注册。


---


#### ZeroNet 只支持 HTML, CSS 的网站吗？

ZeroNet 是为动态构建的，实时更新的网站，但是你可以用它托管任何种类的文件。
(VCS 版本库，你自己的瘦客户端，数据库等)


---


#### 我如何才能创建一个新的 ZeroNet 站点？

[跟着这些介绍。](/using_zeronet/create_new_site/)

---


#### 它是怎么工作的？

- 当你打开一个站点它将从 Bittorrent 网络询问访客的 IP 地址。
- 首先下载名为 __content.json__ 的文件，其中包含所有其他文件名、__hash__ 和站点拥有者的密码学签名。
- 使用站点__地址__和从文件中获得的站点拥有者的__签名__来__验证__下载的 content.json 文件。
- __下载其他文件__ (html, css, js,...) 并使用从 content.json 文件获得的 SHA512 hash 来验证它们。
- 每个访问过的站点也__通过你来做种__。
- 如果站点拥有者（有站点地址的私钥的人）__更改__了站点，并且他/她签名了新的 content.json 并__发布到节点上__。在节点验证 content.json 可信（使用签名）后，它们__下载改动的文件__并发布新的内容到其他节点。

更多信息:
 [描述 ZeroNet 的样站](/using_zeronet/sample_sites/),
 [关于 ZeroNet 如何工作的幻灯片](https://docs.google.com/presentation/d/1qBxkroB_iiX2zHEn0dt-N-qRZgyEzui46XS2hEa3AA4/pub)
