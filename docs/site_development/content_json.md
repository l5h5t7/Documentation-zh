# content.json的结构

每个ZeroNet站点都有一个 `content.json` 文件. ([content.json文件示例](https://github.com/HelloZeroNet/ZeroTalk/blob/master/content.json))

这个文件包含：站点中其他文件的清单，和一个你的私钥签名。以防止文件被篡改 (只有你本人及拥有你私钥的人才能修改)。

以下是该文件支持的字段:


# 自动生成的字段：


---

### address

你网站的地址

**例子**: 1TaLk3zM7ZRskJvrh3ZNCDVGXvkJusPKQ


---


### address_index

该网站的BIP32子密钥，你的BIP32种子的索引。克隆一个网站时自动生成。他使得你能通过BIP32种子来恢复网站的密钥。

**例子**: 30926910

---


### cloned_from

记录该网站从哪里克隆来的。

**Example**: 1BLogC9LN4oPDcruNz3qo1ysa133E9AGg8


---


### files

该网站中自动下载的文件的大小和sha512散列。由该命令自动生成： `zeronet.py siteSign siteaddress privatekey`.

**例子**:
```json
    "css/all.css": {
      "sha512": "869b09328f07bac538c313c4702baa5276544346418378199fa5cef644c139e8",
      "size": 148208
```


---


### files_optional

该网站中被访问才下载的文件的大小和sha512散列。由该命令自动生成： `zeronet.py siteSign siteaddress privatekey`.

**例子**:
```json
    "data/myvideo.mp4": {
      "sha512": "538c09328aa52765443464135cef644c144346418378199fa5cef61837819538",
      "size": 832103
```



---


### modified

该文件（content.json）生成的时间。

**例子**: 1425857522.076


---


### sign (deprecated)

该文件的ECDSA签名。 仅仅用于向前兼容, 很快就会去掉。

**Example**:
```json
  "sign": [
    43117356513690007125104018825100786623580298637039067305407092800990252156956,
    94139380599940414070721501960181245022427741524702752954181461080408625270000
  ],
```


---


### signers_sign

使用站点私钥签名的签名者 (允许多人使用)

**字符串格式**: [signers_required]:[signer address],[signer address]

**例子**: <small>HKNDz9IUHcBc/l2Jm2Bl70XQDL9HYHhJ2hUdg8AMyunACLgxyXBr7EW1/ME4hGkaFZSFmIxlInmxH+BrMVXbnLw=</small>


---


### signs

该文件的ECDSA签名。

**例子**:
```json
  "signs": {
    "1TaLk3zM7ZRskJvrh3ZNCDVGXvkJusPKQ": "G6/QXFKvACPQ7LhoZG4fgqmeOSK99vGM2arVWkm9pV/WPCfc2ulv6iuQnuzw4v5z82qWswcRq907VPdBsdb9VRo="
  },
```


----


### zeronet_version

生成该文件的客户端版本

**例子**: 0.4.1


---


# 设置：


### background-color

warpper的背景颜色

**Example**: #F5F5F5


---


### cloneable

设置为**true**时网站可以克隆。

为了使你的网站能被克隆，你还需要添加干净的初始数据文件 (没有任何帖子)。
你需要添加**-default**前缀到网站的数据文件及文件夹.
在网站的克隆过程中，这些带前缀的文件将替换掉原来的文件。



---


### description

网站的介绍，会显示在标题下方。

**例子**: Decentralized forum demo


---


### domain

网站的namecion域名，有就填上。客户端要有域名插件才管用（默认都有的）。

**例子**: Blog.ZeroNetwork.bit




---


### ignore

签名时要忽略的文件。

**例子**: `((js|css)/(?!all.(js|css))|data/users/.*)` (忽略所有的 js 和 css 文件除了t all.js和all.css并且不要添加 data/users/ 目录的任何文件)


---


### includes

包含另一个content.json

**例子**:

```json
{
  "data/users/content.json": {
    "signers": [ # Possible signers address for the file
      "1LSxsKfC9S9TVXGGNSM3vPHjyW82jgCX5f"
    ],
    "signers_required": 1 # Valid signs required to accept the file (Multisig possibility),
    "files_allowed": "data.json", # Preg pattern for the allowed files in the include file
    "includes_allowed": false, # Nested includes allowed or not
    "max_size": 10000, # Max sum filesize allowed in the include (in bytes)
  }
}
```


---


### merged_type

网站的Merger数据源。

**例子**: `ZeroMe`


---


### optional

可选文件的目录。

**例子**: `(data/mp4/.*|updater/.*)` (data/mp4目录及子目录的所有文件都将成为可选文件)


---


### signs_required

接受此文件需要的签名数量 (允许多人使用)


**例子**: 1


---


### title

网站标题, 在ZeroHello欢迎页的网站列表中可见。

**例子**: ZeroTalk


----

### user_contents

当前目录的用户规则。

字段                   | 描述
                  ---  | ---
**cert_signers**       | 允许的ID站点的地址
**permission_rules**   | 对某ID站用户的文件类型和总大小限制
**permissions**        | 用户权限（false=封禁）

**例子e**:
```json
  "user_contents": {
    "cert_signers": {
      "zeroid.bit": [ "1iD5ZQJMNXu43w1qLB8sfdHVKppVMduGz" ]
    },
    "permission_rules": {
      ".*": {
        "files_allowed": "data.json",
        "max_size": 10000
      },
      "bitid/.*@zeroid.bit": { "max_size": 40000 },
      "bitmsg/.*@zeroid.bit": { "max_size": 15000 }
    },
    "permissions": {
      "bad@zeroid.bit": false,
      "nofish@zeroid.bit": { "max_size": 100000 }
    }
  }
```

----


### viewport

窗口大小 (用于移动端适配)。

**例子**: width=device-width, initial-scale=1.0
