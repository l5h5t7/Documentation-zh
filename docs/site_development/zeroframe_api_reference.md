# ZeroFrame API 参考



# Wrapper

_These commands handled by wrapper frame and does not sent to UiServer using websocket_


#### wrapperConfirm _message, [button_caption]_
显示一条带有确认按钮的通知

参数              | 描述
                  ---  | ---
**message**            | 你想要显示的消息
**button_caption** （可选） | 确认按钮的标题 (默认：OK)

**Return**: True if clicked on button

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
**Return**: Browser's local store for the site

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
**permission**   | Name of permission (eg. Merger:ZeroMe)

---


#### wrapperSetLocalStorage _data_
Set browser's local store data stored for the site

参数              | 描述
                  ---  | ---
**data**               | Any data structure you want to store for the site

**Return**: None

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

**Return**: None

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

**Return**: Text entered to input

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

Set sites's viewport meta tag content (required for mobile sites)


参数           | 描述
               ---  | ---
**viewport**        | The viewport meta tag content

**Return**: None

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
Add a new certificate to current user.

参数            | 描述
                 --- | ---
**domain**           | 证书颁发者域名
**auth_type**        | 注册使用的验证方式
**auth_user_name**   | 注册使用的用户名
**cert**             | The cert itself: `auth_address#auth_type/auth_user_name` string signed by the cert site owner

**Return**: "ok", "Not changed" or {"error": error_message}

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

**Return**: None

**例子：**
```coffeescript
@cmd "certSelect", {"accepted_domains": ["zeroid.bit"]}
```


---


#### channelJoin _channel_

请求站点的事件通知

参数   | 描述
        --- | ---
**channel** | Channel to join

**Return**: None

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

**Return**: <list> 请求的结果

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

**Return**: <string> The content of the file


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
获得文件内容

参数        | 描述
             --- | ---
**inner_path**   | 你想要删除的文件

**Return**: "ok" on success else the error message


---


#### fileQuery _dir_inner_path, query_
简单的 json 文件请求命令

参数            | 描述
                 --- | ---
**dir_inner_path**   | Pattern of queried files
**query**            | Query command

**Return**: <list> Matched content

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
Return the rules for the file.

参数            | 描述
                 --- | ---
**inner_path**       | File inner path

**Return**: <list> Matched content

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
**inner_path**   | Inner path of the file you want to write
**content**      | Content you want to write to file (base64 encoded)

**Return**: "ok" on success else the error message

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

**Return**: <dict> 站点上的所有信息

**例子：**
```coffeescript
@cmd "siteInfo", {}, (site_info) =>
	@log "Site info:", site_info
```

**Example return value:**
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
Publish a content.json of the site

参数                 | 描述
                      --- | ---
**privatekey** （可选） | Private key used for signing (默认：current user's privatekey)
**inner_path** （可选） | Inner path of the content json you want to publish (默认：content.json)
**sign** （可选）       | If True then sign the content.json before publish (默认：True)

**Return**: "ok" on success else the error message

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
Sign a content.json of the site

参数                 | 描述
                      --- | ---
**privatekey** （可选） | Private key used for signing (默认：current user's privatekey)
**inner_path** （可选） | Inner path of the content json you want to sign (默认：content.json)

**Return**: "ok" on success else the error message

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

Force check and download changed content from other peers (only necessary if user is in passive mode and using old version of Zeronet)

参数     | 描述
          --- | ---
**address**   | Address of site want to update (only current site allowed without site ADMIN permission)

**Return:** None

**例子：**
```coffeescript
# Manual site update for passive connections
updateSite: =>
	$("#passive_error a").addClass("loading").removeClassLater("loading", 1000)
	@log "Updating site..."
	@cmd "siteUpdate", {"address": @site_info.address}
```


---


# Plugin: CryptMessage


#### userPublickey _[index]_

Get user's site specific publickey

参数            | 描述
                 --- | ---
**index** （可选） | Sub-publickey within site （默认：0）


**Return**: base64 encoded publickey

---

#### eciesEncrypt _text, [publickey], [return_aes_key]_

Encrypt a text using a publickey

参数                      | 描述
                           --- | ---
**text**                       | 想要加密的文本
**publickey** （可选）       | User's publickey index (int) or base64 encoded publickey (默认：0)
**return_aes_key** （可选）  | Get the AES key used in encryption (默认：False)


**Return**: Encrypted text in base64 format or [Encrypted text in base64 format, AES key in base64 format]

---

#### eciesDecrypt _params, [privatekey]_

Try to decrypt list of texts

参数                      | 描述
                           --- | ---
**params**                     | A text or list of encrypted texts
**privatekey** （可选）      | User's privatekey index (int) or base64 encoded privatekey (默认：0)


**Return**: Decrypted text or list of decrypted texts (null for failed decodings)

---

#### aesEncrypt _text, [key], [iv]_

Encrypt a text using the key and the iv

参数                      | 描述
                           --- | ---
**text**                       | 一条 AES 加密的文本
**key** （可选）             | Base64 编码的密码 (默认：generate new)
**iv** （可选）              | Base64 encoded iv (默认：generate new)


**Return**: [base64 encoded key, base64 encoded iv, base64 encoded encrypted text]


---

#### aesDecrypt _iv, encrypted_text, key_
#### aesDecrypt _encrypted_texts, keys_

Decrypt text using the IV and AES key

参数                      | 描述
                           --- | ---
**iv**                         | IV in Base64 format
**encrypted_text**             | Encrypted text in Base64 format
**encrypted_texts**            | List of [base64 encoded iv, base64 encoded encrypted text] pairs
**key**                        | Base64 encoded password for the text
**keys**                       | Keys for decoding (tries every one for every pairs)


**Return**: Decoded text or list of decoded texts


---


# Plugin: Newsfeed


#### feedFollow _feeds_

Set followed sql queries.

The SQL query should result in rows with cols:

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

**Return**: ok

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

Return of current followed feeds


**Return**: The currently followed feeds in the same format as in the feedFollow commands


---

#### feedQuery

Execute all followed sql query


**Return**: The result of the followed Sql queries


---

# Plugin: MergerSite


#### mergerSiteAdd _addresses_

开始下载新的合并站点

参数            | 描述
                 --- | ---
**addresses**         | Site address or list of site addresses


---

#### mergerSiteDelete _address_

停止做种并删除一个合并站点

参数            | 描述
                 --- | ---
**address**           | Site address


---

#### mergerSiteList _[query_site_info]_

返回合并站点。

参数            | 描述
                 --- | ---
**query_site_info**  | If True, then gives back detailed site info for merged sites

---


# 管理命令
_(requires ADMIN permission in data/sites.json)_



---


#### configSet _key, value_

Create or update an entry in ZeroNet config file. (zeronet.conf by default)


参数            | 描述
                 --- | ---
**key**              | Configuration entry name
**value**            | Configuration entry new value


**Return**: ok


---



#### certSet _domain_

设置当前站点使用的证书

参数            | 描述
                 --- | ---
**domain**           | 证书颁发者的域名

**Return**: None


---


#### channelJoinAllsite _channel_

请求每个站点的通知

参数           | 描述
               ---  | ---
**channel**         | 加入的通道 （参见 channelJoin）

**Return**: None




---


#### serverPortcheck

开始检查端口是否打开

**Return**: True （端口打开） 或 False （端口关闭）


---


#### serverShutdown

停止运行 ZeroNet 客户端

**Return**: None



---


#### serverUpdate

从 github 上重新下载 ZeroNet

**Return**: None


---


#### siteClone _address_
复制站点文件到一个新的站点

Every file and directory will be skipped if it has a `-default` subfixed version and the subfixed version will be copied instead of it.


Eg. If you have a `data` and a `data-default` directory: The `data` directory will not be copied and the `data-default` directory will be renamed to data.

参数           | 描述
               ---  | ---
**address**         | 想要克隆的站点地址

**Return**: None, automatically redirects to new site on completion


---


#### siteList

**Return**: <list> SiteInfo list of all downloaded sites


---


#### sitePause _address_
暂停站点服务

参数           | 描述
               ---  | ---
**address**         | 想要暂停的站点地址

**Return**: None


---


#### siteResume _address_
恢复站点服务

参数           | 描述
               ---  | ---
**address**         | 想要恢复的站点地址

**Return**: None


