# ZeroFrame API 参考



# Wrapper

_这些命令由封装框架处理，不会使用 websocket 发送到 UiServer_


#### wrapperConfirm _message, [button_caption]_
显示一条带有确认按钮的通知

参数              | 描述
                  ---  | ---
**message**            | 你想要显示的消息
**button_caption** （可选） | 确认按钮的标题 (默认：OK)

**返回值**: True 如果你点了按钮

**例子：**
```coffeescript
# Delete site
siteDelete: (address) ->
	site = @sites[address]
	title = site.content.title
	if title.length > 40
		title = title.substring(0, 15)+"..."+title.substring(title.length-10)
	@cmd "wrapperConfirm", ["Are you sure you sure? <b>#{title}</b>", "Delete"], (confirmed) =>
		@log "Deleting #{site.address}...", confirmed
		if confirmed
			$(".site-#{site.address}").addClass("deleted")
			@cmd "siteDelete", {"address": address}
```


---


#### wrapperGetLocalStorage
**返回值**: 浏览器对站点的本地存储

**例子：**
```coffeescript
@cmd "wrapperGetLocalStorage", [], (res) =>
	res ?= {}
	@log "Local storage value:", res
```


---

#### wrapperPermissionAdd _permission_

为站点请求新权限


参数        | 描述
             --- | ---
**permission**   | 权限名称 (eg. Merger:ZeroMe)

---


#### wrapperSetLocalStorage _data_
设置浏览器对站点的本地存储

参数              | 描述
                  ---  | ---
**data**               | 你想要存储在站点的任何结构

**返回值**: None

**例子：**
```coffeescript
Page.local_storage["topic.#{@topic_id}_#{@topic_user_id}.visited"] = Time.timestamp()
Page.cmd "wrapperSetLocalStorage", Page.local_storage
```


---


#### wrapperNotification _type, message, [timeout]_
显示一个通知

参数              | 描述
                  ---  | ---
**type**               | 可能的值：info, error, done
**message**            | 你想要显示的消息
**timeout** （可选） | 在这个间隔后隐藏显示 （毫秒）

**返回值**: None

**例子：**
```coffeescript
@cmd "wrapperNotification", ["done", "Your registration has been sent!", 10000]
```


---


#### wrapperPrompt _message, [type]_

确认用户的文本输入

参数           | 描述
               ---  | ---
**message**         | 你想要显示的消息
**type** （可选） | 输入的类型 (默认：text)

**返回值**: 输入的文本

**例子：**
```coffeescript
# Prompt the private key
@cmd "wrapperPrompt", ["Enter your private key:", "password"], (privatekey) =>
	$(".publishbar .button").addClass("loading")
	# Send sign content.json and publish request to server
	@cmd "sitePublish", [privatekey], (res) =>
		$(".publishbar .button").removeClass("loading")
		@log "Publish result:", res
```


#### wrapperSetViewport _viewport_

设置站点的 viewport 元标记内容 （移动站点需要）


参数           | 描述
               ---  | ---
**viewport**        | viewport 元标记内容

**返回值**: None

**例子：**
```coffeescript
# Prompt the private key
@cmd "wrapperSetViewport", "width=device-width, initial-scale=1.0"
```

---


# UiServer

The UiServer is for ZeroNet what the LAMP setup is for normal websites.

The UiServer will do all the 'backend' work (eg: querying the DB, accessing files, etc). This are the API calls you will need to make your site dynamic.



#### certAdd _domain, auth_type, auth_user_name, cert_
为当前用户添加一个新证书

参数            | 描述
                 --- | ---
**domain**           | 证书颁发者域名
**auth_type**        | 注册使用的验证方式
**auth_user_name**   | 注册使用的用户名
**cert**             | The cert itself: `auth_address#auth_type/auth_user_name` string signed by the cert site owner

**返回值**: "ok", "Not changed" or {"error": error_message}

**例子：**
```coffeescript
@cmd "certAdd", ["zeroid.bit", auth_type, user_name, cert_sign], (res) =>
	$(".ui").removeClass("flipped")
	if res.error
		@cmd "wrapperNotification", ["error", "#{res.error}"]
```


---


#### certSelect _accepted_domains_
显示证书选择器

参数            | 描述
                 --- | ---
**accepted_domains** | List of domains that accepted by site as authorization provider

**返回值**: None

**例子：**
```coffeescript
@cmd "certSelect", {"accepted_domains": ["zeroid.bit"]}
```


---


#### channelJoin _channel_

请求站点的事件通知

参数   | 描述
        --- | ---
**channel** | 加入的通道

**返回值**: None

**Channels**:

 - **siteChanged** (joined by default)<br>Events: peers_added, file_started, file_done, file_failed

**Example**:
```coffeescript
# Wrapper websocket connection ready
onOpenWebsocket: (e) =>
	@cmd "channelJoinAllsite", {"channel": "siteChanged"}

# Route incoming requests and messages
route: (cmd, data) ->
	if cmd == "setSiteInfo"
		@log "Site changed", data
	else
		@log "Unknown command", cmd, data
```

**Example event data**
```json
{
	"tasks":0,
	"size_limit":10,
	"address":"1RivERqttrjFqwp9YH1FviduBosQPtdBN",
	"next_size_limit":10,
	"event":[ "file_done", "index.html" ],
	[...] # Same as siteInfo return dict
}

```


---


#### dbQuery _query_
在 sql 缓存中执行一个查询

参数            | 描述
                 --- | ---
**query**            | Sql 请求命令

**返回值**: <list> 请求的结果

**例子：**
```coffeescript
@log "Updating user info...", @my_address
Page.cmd "dbQuery", ["SELECT user.*, json.json_id AS data_json_id FROM user LEFT JOIN json USING(path) WHERE path='#{@my_address}/data.json'"], (res) =>
	if res.error or res.length == 0 # Db not ready yet or No user found
		$(".head-user.visitor").css("display", "")
		$(".user_name-my").text("Visitor")
		if cb then cb()
		return

	@my_row = res[0]
	@my_id = @my_row["user_id"]
	@my_name = @my_row["user_name"]
	@my_max_size = @my_row["max_size"]
```


---

#### fileGet _inner_path, [required]_
获得文件内容

参数        | 描述
             --- | ---
**inner_path**   | 你想要获得的文件
**required** （可选） | 如果不存在的话，尝试并等待文件。(默认：True)

**返回值**: <string> 文件的内容


**例子：**
```coffeescript
# Upvote a topic on ZeroTalk
submitTopicVote: (e) =>
	if not Users.my_name # Not registered
		Page.cmd "wrapperNotification", ["info", "Please, request access before posting."]
		return false

	elem = $(e.currentTarget)
	elem.toggleClass("active").addClass("loading")
	inner_path = "data/users/#{Users.my_address}/data.json"

	Page.cmd "fileGet", [inner_path], (data) =>
		data = JSON.parse(data)
		data.topic_votes ?= {} # Create if not exits
		topic_address = elem.parents(".topic").data("topic_address")

		if elem.hasClass("active") # Add upvote to topic
			data.topic_votes[topic_address] = 1
		else # Remove upvote from topic
			delete data.topic_votes[topic_address]

		# Write file and publish to other peers
		Page.writePublish inner_path, Page.jsonEncode(data), (res) =>
			elem.removeClass("loading")
			if res == true
				@log "File written"
			else # Failed
				elem.toggleClass("active") # Change back

	return false
```


---


#### fileDelete _inner_path_
删除一个文件

参数        | 描述
             --- | ---
**inner_path**   | 你想要删除的文件

**返回值**: 成功时返回 "ok" ，否则返回错误信息


---


#### fileQuery _dir_inner_path, query_
简单的 json 文件请求命令

参数            | 描述
                 --- | ---
**dir_inner_path**   | 请求文件的模式
**query**            | 请求命令

**返回值**: <list> 匹配的内容

**Query examples:**

 - `["data/users/*/data.json", "topics"]`: Returns all topics node from all user files
 - `["data/users/*/data.json", "comments.1@2"]`: Returns `user_data["comments"]["1@2"]` value from all user files
 - `["data/users/*/data.json", ""]`: Returns all data from users files

**例子：**
```coffeescript
@cmd "fileQuery", ["data/users/*/data.json", "topics"], (topics) =>
	topics.sort (a, b) -> # Sort by date
		return a.added - b.added
	for topic in topics
		@log topic.topic_id, topic.inner_path, topic.title
```


---


#### fileRules _inner_path_
返回文件的规则。

参数            | 描述
                 --- | ---
**inner_path**       | 文件内部路径

**返回值**: <list> 匹配的内容

**Example result:**

```json
{
	"current_size": 2485,
	"cert_signers": {"zeroid.bit": ["1iD5ZQJMNXu43w1qLB8sfdHVKppVMduGz"]},
	"files_allowed": "data.json",
	"signers": ["1J3rJ8ecnwH2EPYa6MrgZttBNc61ACFiCj"],
	"user_address": "1J3rJ8ecnwH2EPYa6MrgZttBNc61ACFiCj",
	"max_size": 100000
}
```

**例子：**
```coffeescript
@cmd "fileRules", "data/users/1J3rJ8ecnwH2EPYa6MrgZttBNc61ACFiCj/content.json", (rules) =>
	@log rules
```


---


#### fileWrite _inner_path, content_

写入文件内容


参数        | 描述
             --- | ---
**inner_path**   | 你想要写入的文件的内部路径
**content**      | 你想要写入的文件内容 （base64 编码）

**返回值**: 成功时返回 "ok" ，否则返回错误信息

**例子：**
```coffeescript
writeData: (cb=null) ->
	# Encode to json, encode utf8
	json_raw = unescape(encodeURIComponent(JSON.stringify({"hello": "ZeroNet"}, undefined, '\t')))
	# Convert to to base64 and send
	@cmd "fileWrite", ["data.json", btoa(json_raw)], (res) =>
		if res == "ok"
			if cb then cb(true)
		else
			@cmd "wrapperNotification", ["error", "File write error: #{res}"]
			if cb then cb(false)
```

_Note:_ to write files that not in content.json yet, you must have `"own": true` in `data/sites.json` at the site you want to write


---



#### serverInfo

**Return:** <dict> 服务器上的所有信息

**例子：**
```coffeescript
@cmd "serverInfo", {}, (server_info) =>
	@log "Server info:", server_info
```

**Example return value:**
```json
{
	"debug": true, # 运行在调试模式
	"fileserver_ip": "*", # 绑定的文件服务器
	"fileserver_port": 15441, # 文件服务器的端口
	"ip_external": true, # Active of passive mode
	"platform": "win32", # Operating system
	"ui_ip": "127.0.0.1", # UiServer binded to
	"ui_port": 43110, # UiServer port (Web)
	"version": "0.2.5" # 版本
}
```




---


#### siteInfo

**返回值**: <dict> 站点上的所有信息

**例子：**
```coffeescript
@cmd "siteInfo", {}, (site_info) =>
	@log "Site info:", site_info
```

**实例返回值：**
```json
{
	"tasks": 0, # Number of files currently under download
	"size_limit": 10, # Current site size limit in MB
	"address": "1EU1tbG9oC1A8jz2ouVwGZyQ5asrNsE4Vr", # Site address
	"next_size_limit": 10, # Size limit required by sum of site's files
	"auth_address": "2D6xXUmCVAXGrbVUGRRJdS4j1hif1EMfae", # Current user's bitcoin address
	"auth_key_sha512": "269a0f4c1e0c697b9d56ccffd9a9748098e51acc5d2807adc15a587779be13cf", # Deprecated, dont use
	"peers": 14, # Peers of site
	"auth_key": "pOBdl00EJ29Ad8OmVIc763k4", # Deprecated, dont use
	"settings":  {
		"peers": 13, # Saved peers num for sorting
		"serving": true, # Site enabled
		"modified": 1425344149.365, # Last modification time of all site's files
		"own": true, # Own site
		"permissions": ["ADMIN"], # Site's permission
		"size": 342165 # Site total size in bytes
	},
	"bad_files": 0, # Files that needs to be download
	"workers": 0, # Current concurent downloads
	"content": { # Root content.json
		"files": 12, # Number of file, detailed file info removed to reduce data transfer and parse time
		"描述": "This site",
		"title": "ZeroHello",
		"signs_required": 1,
		"modified": 1425344149.365,
		"ignore": "(js|css)/(?!all.(js|css))",
		"signers_sign": null,
		"address": "1EU1tbG9oC1A8jz2ouVwGZyQ5asrNsE4Vr",
		"zeronet_version": "0.2.5",
		"includes": 0
	},
	"cert_user_id": "zeronetuser@zeroid.bit", # Currently selected certificate for the site
	"started_task_num": 1, # Last number of files downloaded
	"content_updated": 1426008289.71 # Content.json update time
}
```


---


#### sitePublish _[privatekey], [inner_path], [sign]_
发布站点的 content.json

参数                 | 描述
                      --- | ---
**privatekey** （可选） | 用于签名的私钥 （默认：当前的用户私钥）
**inner_path** （可选） | 你想要发布的内容 json 的内部路径 （默认：content.json）
**sign** （可选）       | 如果填 True 在发布前给 content.json 签名 （默认：True）

**返回值**: 成功时返回 "ok" ，否则返回错误信息

**例子：**
```coffeescript
# Prompt the private key
@cmd "wrapperPrompt", ["Enter your private key:", "password"], (privatekey) =>
	$(".publishbar .button").addClass("loading")
	# Send sign content.json and publish request to server
	@cmd "sitePublish", [privatekey], (res) =>
		$(".publishbar .button").removeClass("loading")
		@log "Publish result:", res
```


---


#### siteSign _[privatekey], [inner_path]_
签名站点的 content.json

参数                 | 描述
                      --- | ---
**privatekey** （可选） | 用于签名的私钥 (默认：current user's privatekey)
**inner_path** （可选） | 你想要发布的内容 json 的内部路径 （默认：content.json）

**返回值**: 成功时返回 "ok" ，否则返回错误信息

> __Note:__
> Use "stored" as privatekey if its definied in users.json (eg. cloned sites)

**例子：**
```coffeescript
if @site_info["privatekey"] # Private key stored in users.json
	@cmd "siteSign", ["stored", "content.json"], (res) =>
		@log "Sign result", res
```


---



#### siteUpdate _address_

从其他节点强制检查和下载更改的内容 （只有用户在被动模式和使用老版本的 Zeronet 才必要）

参数     | 描述
          --- | ---
**address**   | 想要更新的站点地址 （没有站点的 ADMIN 权限只有当前的站点被允许）

**返回值** 没有

**例子：**
```coffeescript
# Manual site update for passive connections
updateSite: =>
	$("#passive_error a").addClass("loading").removeClassLater("loading", 1000)
	@log "Updating site..."
	@cmd "siteUpdate", {"address": @site_info.address}
```


---


# 插件：CryptMessage


#### userPublickey _[index]_

获取用户在网站特定的公钥

参数            | 描述
                 --- | ---
**index** （可选） | 站点的子公钥 （默认：0）


**返回值**: base64 编码的公钥

---

#### eciesEncrypt _text, [publickey], [return_aes_key]_

用公钥加密一段文本

参数                      | 描述
                           --- | ---
**text**                       | 想要加密的文本
**publickey** （可选）       | 用户的私钥索引 (int) 或 base64 编码的私钥 （默认：0）
**return_aes_key** （可选）  | 获取用于加密的 AES 密钥 （默认：False）


**返回值**: base64 格式的加密后的文本或 [base64 格式的加密后的文本, base64 格式的 AES 密钥]

---

#### eciesDecrypt _params, [privatekey]_

尝试解密文本列表

参数                      | 描述
                           --- | ---
**params**                     | 文本或加密的文本列表
**privatekey** （可选）      | 用户的私钥索引 (int) 或 base64 编码的私钥 （默认：0）


**返回值**: 解密的文本或解密的文本列表 （解密失败时返回 null）

---

#### aesEncrypt _text, [key], [iv]_

使用指定的 key 和 iv 加密文本

参数                      | 描述
                           --- | ---
**text**                       | 一条 AES 加密的文本
**key** （可选）             | Base64 编码的密码 (默认：generate new)
**iv** （可选）              | Base64 编码的 iv (默认：generate new)


**返回值**: [base64 编码的密钥, base64 编码的 iv, base64 编码的加密文本]


---

#### aesDecrypt _iv, encrypted_text, key_
#### aesDecrypt _encrypted_texts, keys_

使用指定的 key 和 iv 解密文本

参数                      | 描述
                           --- | ---
**iv**                         | Base64 格式编码的 IV
**encrypted_text**             | base64 编码的加密文本
**encrypted_texts**            | [base64 编码的 iv, base64 编码的加密文本] 列表对
**key**                        | 用于文本加密的 Base64 编码的密码
**keys**                       | 用于解密的密钥 (tries every one for every pairs)


**返回值**: Decoded text or list of decoded texts


---


# Plugin: Newsfeed


#### feedFollow _feeds_

设置订阅的 sql 请求。

SQL 请求应该返回列与行的结果：

Field          | 描述
           --- | ---
**type**       | 类型：post, article, comment, mention
**date_added** | 事件时间
**title**      | 显示的事件的第一行
**body**       | 事件的第二行和第三行
**url**        | 链接到事件页面

参数      | 描述
           --- | ---
**feeds**      | 格式：{"query name": [SQL query, [param1, param2, ...], ...}, 参数s will be escaped, joined by `,` inserted in place of `:params` in the Sql query.

**返回值**: ok

**例子：**
```coffeescript
# Follow ZeroBlog posts
query = "
	SELECT
	 post_id AS event_uri,
	 'post' AS type,
	 date_published AS date_added,
	 title AS title,
	 body AS body,
	 '?Post:' || post_id AS url
	FROM post
"
params = [""]
Page.cmd feedFollow [{"Posts": [query, params]}]
```

---

#### feedListFollow

返回当前订阅的消息来源


**返回值**: 与 feedFollow 命令相同格式的订阅的消息来源


---

#### feedQuery

执行所有消息来源的 sql 请求


**返回值**: 消息来源的 Sql 请求的结果


---

# Plugin: MergerSite


#### mergerSiteAdd _addresses_

开始下载新的合并站点

参数            | 描述
                 --- | ---
**addresses**         | 站点地址或站点地址列表


---

#### mergerSiteDelete _address_

停止做种并删除一个合并站点

参数            | 描述
                 --- | ---
**address**           | 站点地址


---

#### mergerSiteList _[query_site_info]_

返回合并站点。

参数            | 描述
                 --- | ---
**query_site_info**  | 如果设为 True , 返回合并站点的详细站点信息

---


# 管理命令
_(requires ADMIN permission in data/sites.json)_



---


#### configSet _key, value_

在 ZeroNet 配置文件中创建或更新一个条目。 (zeronet.conf by default)


参数            | 描述
                 --- | ---
**key**              | 配置条目名
**value**            | 配置条目新的值


**返回值**: ok


---



#### certSet _domain_

设置当前站点使用的证书

参数            | 描述
                 --- | ---
**domain**           | 证书颁发者的域名

**返回值**: None


---


#### channelJoinAllsite _channel_

请求每个站点的通知

参数           | 描述
               ---  | ---
**channel**         | 加入的通道 （参见 channelJoin）

**返回值**: None




---


#### serverPortcheck

开始检查端口是否打开

**返回值**: True （端口打开） 或 False （端口关闭）


---


#### serverShutdown

停止运行 ZeroNet 客户端

**返回值**: None



---


#### serverUpdate

从 github 上重新下载 ZeroNet

**返回值**: None


---


#### siteClone _address_
复制站点文件到一个新的站点

如果文件和目录有带 `-default` 后缀的版本，那些文件会被跳过，带后缀的版本会替代它被复制。


例如，如果你有一个 `data` 和一个 `data-default` 目录：`data`  目录将不会被复制，`data-default` 将被重命名成 `data` 。

参数           | 描述
               ---  | ---
**address**         | 想要克隆的站点地址

**返回值**: 没有，自动重定向到完成的新站点中


---


#### siteList

**返回值**: <list> 所有已下载的站点信息列表


---


#### sitePause _address_
暂停站点服务

参数           | 描述
               ---  | ---
**address**         | 想要暂停的站点地址

**返回值**: 没有

---


#### siteResume _address_
恢复站点服务

参数           | 描述
               ---  | ---
**address**         | 想要恢复的站点地址

**返回值**: 没有


