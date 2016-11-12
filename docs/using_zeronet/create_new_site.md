# Create new ZeroNet site


### 1. 创建站点结构

* 如果 ZeroNet 还在运行的话，关闭它
* 定位到 ZeroNet 安装并运行的目录：

```bash
$ zeronet.py siteCreate
...
- Site private key: 23DKQpzxhbVBrAtvLEc2uvk7DZweh4qL3fn3jpM3LgHDczMK2TtYUq
- Site address: 13DNDkMUExRf9Xa9ogwPKqp7zyHFEqbhC2
...
- Site created!
$ zeronet.py
...
```

- 该操作将创建你的站点里的初始文件 ```data/13DNDkMUExRf9Xa9ogwPKqp7zyHFEqbhC2```.

> __Note:__
> 使用 bundle 版本的 Windows 用户必须定位到 ZeroBundle/ZeroNet 然后运行 `"../Python/python.exe" zeronet.py siteCreate`

### 2. 构建/更改站点

* 更新站点位于 ```data/[your site address key]``` (eg: 13DNDkMUExRf9Xa9ogwPKqp7zyHFEqbhC2) 的文件。
* 当你的站点已经准备好运行:

```bash
$ zeronet.py siteSign 13DNDkMUExRf9Xa9ogwPKqp7zyHFEqbhC2
- Signing site: 13DNDkMUExRf9Xa9ogwPKqp7zyHFEqbhC2...
Private key (input hidden):
```

* 输入你在创建站点时获得的私钥。这将给所有的文件签名以让节点能验证是站点所有者进行的更改。

### 3. 发布站点更改

* 为了通知节点你的更改你需要运行：

```bash
$ zeronet.py sitePublish 13DNDkMUExRf9Xa9ogwPKqp7zyHFEqbhC2
...
Site:13DNDk..bhC2 Publishing to 3/10 peers...
Site:13DNDk..bhC2 Successfuly published to 3 peers
- Serving files....
```

* 完成了！你已经成功地签名并发布了你的更改。
* 你的站点将可从 ```http://localhost:43110/13DNDkMUExRf9Xa9ogwPKqp7zyHFEqbhC2``` 被访问


**下一节:** [ZeroNet 开发者文档](/site_development/getting_started/)
