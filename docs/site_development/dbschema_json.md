# dbschema.json 的结构

[dbschema.json 文件的样例](https://github.com/HelloZeroNet/ZeroTalk/blob/master/dbschema.json)

下面的代码将跟随这些事件而运行：

 - 如果收到了更新的 data/users/*/data.json 文件 （例如：一个用户发了一个帖）：
   - 在 `data["topics"]` 中的每行被加载到 `topic` 表
   - 在 `data["comment_votes"]` 中的每个键被加载到 `comment_vote` 表作为 `comment_hash` 列并且值被存储到 `vote` 的相同行
 - 如果收到了更新的 data/users/content.json 文件 （例如：创建了新用户）：
   - The `"user_id", "user_name", "max_size", "added"` key in value of `content["include"]` is loaded into the `user` table and the key is stored as `path`

```json

{
  "db_name": "ZeroTalk", # 数据库名 （仅用于调试）
  "db_file": "data/users/zerotalk.db", # 相对于站点目录的数据库文件路径
  "version": 2, # 1 = JSON 表有包括目录和文件名的 path 列
                # 2 = JSON 表有有单独的目录和 file_name 列
                # 3 = 与版本 2 相同，但是还有 site 列 （用于合并站点）
  "maps": { # JSON 到数据库的映射
    ".*/data.json": { # db_file 的相对路径的正则匹配
      "to_table": [ # 加载值到表中
        {
          "node": "topics", # 读取 data.json[topics] 的键值
          "table": "topic" # 订阅 topic 表中的数据
        },
        {
          "node": "comment_votes", # 读取 data.json[comment_votes] 中的键值
          "table": "comment_vote", # 订阅 comment_vote 表数据中的数据
          "key_col": "comment_hash",
            # data.json[comment_votes] 是一个样例字典，字典中的值被加载到 comment_vote 表的 comment_hash 列

          "val_col": "vote"
            # data.json[comment_votes] 字典中的值被加载到 comment_vote 表的 vote 列

        }
      ],
      "to_keyvalue": ["next_message_id", "next_topic_id"]
        # 加载 data.json[next_topic_id] 到键值表
        # (key: next_message_id, value: data.json[next_message_id] value)

    },
    "content.json": {
      "to_table": [
        {
          "node": "includes",
          "table": "user",
          "key_col": "path",
          "import_cols": ["user_id", "user_name", "max_size", "added"],
            # 只导入 user 表的这些列
          "replaces": {
            "path": {"content.json": "data.json"}
              # 替换 content.json 的 path 列中的值到 data.json （需要加入）
          }
        }
      ],
      "to_json_table": [ "cert_auth_type", "cert_user_id" ]  # 保存 cert_auth_type 和 cert_user_id directly 到 JSON 表 （用于更快更简便的数据查询）
    }
  },
  "tables": { # 表的定义
    "topic": { # 定义 topic 表
      "cols": [ # 表的列
        ["topic_id", "INTEGER"],
        ["title", "TEXT"],
        ["body", "TEXT"],
        ["type", "TEXT"],
        ["parent_topic_hash", "TEXT"],
        ["added", "DATETIME"],
        ["json_id", "INTEGER REFERENCES json (json_id)"]
      ],
      "indexes": ["CREATE UNIQUE INDEX topic_key ON topic(topic_id, json_id)"],
        # 自动创建的索引

      "schema_changed": 1426195822
        # 上次映射改变的时间，如果客户端版本不同则自动破坏旧的，创建新的表然后重载里面的数据信息

    },
    "comment_vote": {
      "cols": [
        ["comment_hash", "TEXT"],
        ["vote", "INTEGER"],
        ["json_id", "INTEGER REFERENCES json (json_id)"]
      ],
      "indexes": ["CREATE UNIQUE INDEX comment_vote_key ON comment_vote(comment_hash, json_id)", "CREATE INDEX comment_vote_hash ON comment_vote(comment_hash)"],
      "schema_changed": 1426195826
    },
    "user": {
      "cols": [
        ["user_id", "INTEGER"],
        ["user_name", "TEXT"],
        ["max_size", "INTEGER"],
        ["path", "TEXT"],
        ["added", "INTEGER"],
        ["json_id", "INTEGER REFERENCES json (json_id)"]
      ],
      "indexes": ["CREATE UNIQUE INDEX user_id ON user(user_id)", "CREATE UNIQUE INDEX user_path ON user(path)"],
      "schema_changed": 1426195825
    },
    "json": {  # Json table format only required if you have specified to_json_table pattern anywhere
        "cols": [
            ["json_id", "INTEGER PRIMARY KEY AUTOINCREMENT"],
            ["directory", "TEXT"],
            ["file_name", "TEXT"],
            ["cert_auth_type", "TEXT"],
            ["cert_user_id", "TEXT"]
        ],
        "indexes": ["CREATE UNIQUE INDEX path ON json(directory, site, file_name)"],
        "schema_changed": 4
    }
  }
}
```

## data.json 文件的例子
```json
{
  "next_topic_id": 2,
  "topics": [
    {
      "topic_id": 1,
      "title": "Newtopic",
      "body": "Topic!",
      "added": 1426628540,
      "parent_topic_hash": "5@2"
    }
  ],
  "next_message_id": 19,
  "comments": {
    "1@2": [
      {
        "comment_id": 1,
        "body": "New user test!",
        "added": 1423442049
      }
    ],
    "1@13": [
      {
        "comment_id": 2,
        "body": "hello",
        "added": 1424653288
      },
      {
        "comment_id": 13,
        "body": "test 123",
        "added": 1426463715
      }
    ]
  },
  "topic_votes": {
    "1@2": 1,
    "4@2": 1,
    "2@2": 1,
    "1@5": 1,
    "1@6": 1,
    "3@2": 1,
    "1@13": 1,
    "4@5": 1
  },
  "comment_votes": {
    "5@5": 1,
    "2@12": 1,
    "1@12": 1,
    "33@2": 1,
    "45@2": 1,
    "12@5": 1,
    "34@2": 1,
    "46@2": 1
  }
}
```

## content.json 文件的例子

```json
{
  "files": {},
  "ignore": ".*/.*",
  "includes": {
    "13v1FwKcq7dx2UPruFcRcqd8s7VBjvoWJW/content.json": {
      "added": 1426683897,
      "files_allowed": "data.json",
      "includes_allowed": false,
      "max_size": 10000,
      "signers": [
        "13v1FwKcq7dx2UPruFcRcqd8s7VBjvoWJW"
      ],
      "signers_required": 1,
      "user_id": 15,
      "user_name": "meginthelloka"
    },
    "15WGMVViswrF13sAKb7je6oX3UhXavBxxQ/content.json": {
      "added": 1426687209,
      "files_allowed": "data.json",
      "includes_allowed": false,
      "max_size": 10000,
      "signers": [
        "15WGMVViswrF13sAKb7je6oX3UhXavBxxQ"
      ],
      "signers_required": 1,
      "user_id": 18,
      "user_name": "habla"
    }
  }
}
```
