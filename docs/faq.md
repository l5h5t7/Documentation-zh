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

它没有比 Bittorrent 更多的匿名性，but privacy (the possibility to find out who is the owner of the comment/site) will increase as the network and the sites gains more peers.

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

If you want to hide your IP install the latest version of ZeroNet then click Tor > Enable Tor for every connection on ZeroHello.

On Windows Tor is bundled with ZeroNet for other OS [follow Tor install instructions](https://www.torproject.org/docs/installguide.html),
edit your torrc configuration file by removing `#` from line `# ControlPort 9051` then restart your Tor service and ZeroNet.

> __提示：__ 你能使用 [Stats](http://127.0.0.1:43110/Stats) 页面来验证你的 IP 地址。

> __提示：__ 如果你连接错误，确信你已经安装了最新版本的 Tor 。(0.2.7.5+ required)


---


#### 如何在 Linux 下通过 Tor 使用 ZeroNet?

更新最新版本的 Tor (我们需要 0.2.7.5+), follow [these](https://www.torproject.org/docs/debian.html.en) instructions eg. for Debian:

 - `echo 'deb http://deb.torproject.org/torproject.org jessie main'>> /etc/apt/sources.list.d/tor.list`
 - `gpg --keyserver keys.gnupg.net --recv 886DDD89`
 - `gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | apt-key add -`
 - `apt-get update`
 - `apt-get install tor`

编辑配置来启用控制协议：

 - `mcedit /etc/tor/torrc`
 - Remove the `#` character from lines `ControlPort 9051` and `CookieAuthentication 1` (line ~57)
 - `/etc/init.d/tor restart`
 - Add permission yourself to read the auth cookie by `usermod -a -G debian-tor [yourlinuxuser]`<br>(if you are not on Debian check the file's user group by `ls -al /var/run/tor/control.authcookie`)
 - Logout/Login with your user to apply group changes

> __提示:__ You can verify if your Tor running correctly using `echo 'PROTOCOLINFO' | nc 127.0.0.1 9051`

> __提示:__ It's also possible to use without modifying torrc (or using older version of Tor clients) by running it `zeronet.py --tor disable --proxy 127.0.0.1:9050 --disable_udp`, but then you will loose ability to talk with other .onion addresses.



---

#### 我可以在多台机器上使用同一个用户吗?

可以, 你需要复制 `data/users.json`。


---


#### 如何注册一个 .bit 域名?

You can register .bit domains using [Namecoin](https://namecoin.info/).
Manage your domains using the client's GUI or by the [command line interface](http://www.christopherpoole.net/registering-a-bit-domain-with-namecoin.html).

After the registration is done you have to edit your domain's record by adding a zeronet section to it, eg.:

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


> __提示:__ Other possibilities to register .bit domains: [domaincoin.net](https://domaincoin.net/), [peername.com](https://peername.com/), [dotbit.me](https://dotbit.me/)

> __提示:__ You can verify your domain on [namecha.in](http://namecha.in/), for example: [zeroid.bit](http://namecha.in/name/d/zeroid)

> __提示:__ You should use only [lower-cased letters, numbers and - in your domains](http://wiki.namecoin.info/?title=Domain_Name_Specification_2.0#Valid_Domains).

> __提示:__ To make ZeroHello to link your domain instead of your site's address, add a domain key to your content.json. ([Example](https://github.com/HelloZeroNet/ZeroBlog/blob/master/content.json#L6))


---


#### Can I use the generated site address/private key to accept Bitcoin payments?

Yes, it's a standard Bitcoin address. The private key is WIF formatted, so you can import it in most clients.

> __提示:__ It's not recommended to keep a high amount of money on your site's address, because you have to enter your private key every time you modify your site.


---


#### What happens when someone hosts malicious content?

The ZeroNet sites are sandboxed, they have the same privileges as any other website you visit over the Internet.
You are in full control of what you are hosting. If you find suspicious content you can stop hosting the site at any time.


---


#### 有可能在远程机器上安装 ZeroNet 吗？
Yes, you have to enable the UiPassword plugin by renaming the __plugins/disabled-UiPassword__ directory to __plugins/UiPassword__,
then start ZeroNet on the remote machine using <br>`zeronet.py --ui_ip "*" --ui_password anypassword`.
This will bind the ZeroNet UI webserver to all interfaces, but to keep it secure you can only access it by entering the given password.

> __提示:__ You can also restrict the interface based on ip address by using `--ui_restrict ip1 ip2`.

> __提示:__ You can specify the password in config file by creating a `zeronet.conf` file and add `[global]`, `ui_password = anypassword` lines to it.


---


#### Is there anyway to track the bandwidth ZeroNet is using?

The sent/received bytes are displayed at ZeroNet's sidebar.<br>(open it by dragging the topright `0` button to left)

> __提示:__ Per connection statistics page: [http://127.0.0.1:43110/Stats](http://127.0.0.1:43110/Stats)


---


#### What happens if two people use the same keys to modify a site?

Every content.json file is timestamped, the clients always accepts the newest one.


---


#### ZeroNet 是否使用了比特币的区块链?

不，ZeroNet only uses the cryptography of Bitcoin for site addresses and content signing/verification.
The users identification is also based on Bitcoin's [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) format.

Namecoin's blockchain is being used for domain registrations.


---


#### ZeroNet 只支持 HTML, CSS 的网站吗？

ZeroNet 是为动态构建的，实时更新的网站，但是你可以用它托管任何种类的文件。
(VCS 版本库，你自己的瘦客户端，数据库等)


---


#### 我如何才能创建一个新的 ZeroNet 站点？

[跟着这些介绍。](/using_zeronet/create_new_site/)

---


#### 它是怎么工作的？

- 当你打开一个站点它将从 Bittorrent 网络询问访客的 IP 地址
- 首先下载名为 __content.json__ 的文件，which holds all other filenames,
  __hashes__ and the site owner's cryptographic signature
- __Verifies__ the downloaded content.json file using the site's __address__ and the site owner's __signature__ from the file
- __Downloads other file__ (html, css, js...) and verifies them using the SHA512 hash for content.json file
- Each visited site becomes __also served by you__.
- If the site owner (who has the private key for the site address) __modifies__ the site, then he/she signs
  the new content.json and __publishes it to the peers__. After the peers have verified the content.json
  integrity (using the signature), they __download the modified files__ and publish the new content to other peers.

更多信息:
 [描述 ZeroNet 的样站](/using_zeronet/sample_sites/),
 [Slides about how does ZeroNet work](https://docs.google.com/presentation/d/1_2qK1IuOKJ51pgBvllZ9Yu7Au2l551t3XBgyTSvilew/pub)
