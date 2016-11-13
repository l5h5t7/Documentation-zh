# ZeroNet 网络协议

 - 每条消息时使用 [MessagePack](http://msgpack.org/) 编码的
 - 每条消息有 3 个参数：
    * `cmd`: 请求的命令
    * `req_id`: 请求的唯一 ID （单调递增的随机数），当回复命令时客户端必须包含它
    * `params`: 请求的参数
 - 样例请求：`{"cmd": "getFile", "req_id": 1, "params:" {"site": "1EU...", "inner_path": "content.json", "location": 0}}`
 - 样例响应：`{"cmd": "response", "to": 1, "body": "content.json content", "location": 1132, "site": 1132}`
 - 样例错误响应：`{"cmd": "response", "to": 1, "error": "Unknown site"}`

# 节点请求

#### getFile _site_, _inner_path_, _location_
从客户端上请求一个文件

参数            | 描述
                 --- | ---
**site**             | 站点地址 （例如：1EU1tbG9oC1A8jz2ouVwGZyQ5asrNsE4Vr）
**inner_path**       | 相对于站点目录的文件路径
**location**         | 从这个字节开始请求 (max 512 bytes got sent in a request, so you need multiple requests for larger files)

**Return**:

返回值           | 描述
                 --- | ---
**body**             | 请求的文件内容
**location**         | 发送的最后一个字节的位置
**size**             | 文件的总大小


---


#### ping
检查是否客户端还在线

**Return**:

返回值           | 描述
                 --- | ---
**body**             | Pong


---


#### pex _site_, _peers_, _need_
交换客户端的节点。
节点被打包成 6 个字节 （使用 inet_ntoa 的 4 字节 IP + 2 字节端口）

参数            | 描述
                 --- | ---
**site**             | 站点地址 （例如：1EU1tbG9oC1A8jz2ouVwGZyQ5asrNsE4Vr）
**peers**            | 请求者已有的节点列表 （已打包的）
**need**             | 请求者想要的节点数

**Return**:

返回值           | 描述
                 --- | ---
**peers**            | 列出他有的站点的节点 （已打包的）


---

#### update _site_, _inner_path_, _body_
更新一个站点文件。


参数            | 描述
                 --- | ---
**site**             | 站点地址 （例如：1EU1tbG9oC1A8jz2ouVwGZyQ5asrNsE4Vr）
**inner_path**       | 相对于站点目录的文件路径
**body**             | 要更新的文件的完整内容

**Return**:

返回值           | 描述
                 --- | ---
**ok**               | 感谢消息成功更新
