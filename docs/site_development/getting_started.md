# 概述

ZeroNet可以发布静态和动态的站点。

尽管ZeroNet不能个运行像PHP或Ruby这样的脚本语言, 但是你可以用ZeroNet的API(叫做ZeroFrame), JavaScript (或者CoffeeScript) 和一个内置的数据库创建动态网站。

## ZeroChat 教程

在教程中我们将使用少于 100 行代码来构建一个点对点的、去中心化的、无服务器与后端的聊天站点。

* [在 ZeroBlog 阅读](http://127.0.0.1:43110/Blog.ZeroNetwork.bit/?Post:99:ZeroChat+tutorial)
* [在 Medium.com 阅读](https://decentralize.today/decentralized-p2p-chat-in-100-lines-of-code-d6e496034cd4)

## ZeroNet Debug模式

以`--debug` 参数运行ZeroNet会让网站开发更容易

用这个命令开启Debug模式: `python zeronet.py --debug`

### Debug 模式的特点:

- CoffeeScript -> JavaScript自动转换（这篇文档和所有示例站点都使用 [CoffeeScript](http://coffeescript.org/)编写)
- Debug 信息会在控制台中显示
- 自动重新加载修改过的源文件(UiRequest, UiWebsocket, FileRequest) 免去重新启动 (Linux平台需要 [PyFilesystem](http://pyfilesystem.org/))
- `http://127.0.0.1:43110/Debug`  在Python控制台对错误位置进行追踪和互动 (基于这个吊炸天的Werkzeug debugger - 需要 [Werkzeug](http://werkzeug.pocoo.org/))
- `http://127.0.0.1:43110/Console` 打开Python控制台(需要 [Werkzeug](http://werkzeug.pocoo.org/))

### Extra features (仅对你自己的站点有效)

 - 合并 CSS 文件: 网站文件夹内的所有CSS文件将会被合并至 `all.css`。你可以选择只让网站包含这个文件。如果想保留其他CSS文件使网站开发更容易, 你可以在`content.json`文件中添加ignore字段。这样的话，它们就不会和你的网站一起被发布。 (例: 向网站的`content.json`文件中添加`"ignore": "(js|css)/(?!all.(js|css))"`将会忽略所有的 CSS和JS文件除了`all.js`和`all.css`)
 - 何合并 JS 文件: 网站文件夹内的所有JS文件将会被合并至 `all.js`。 如果有CoffeeScript编译器(Window平台已打包) ，他会将`.coffee`转换为`.js`。
 - Order in which files are merged into all.css/all.js: Files inside subdirectories of the css/js folder comes first; Files in the css/js folder will be merged according to file name ordering (01_a.css, 02_a.css, etc)

### ZeroNet网站开发教程

#### [Part #1](http://127.0.0.1:43110/Blog.ZeroNetwork.bit/?Post:43:ZeroNet+site+development+tutorial+1):

 - 建立网站
 - 第一个ZeroFrame API calls

#### [Part #2](http://127.0.0.1:43110/Blog.ZeroNetwork.bit/?Post:46:ZeroNet+site+development+tutorial+2):

 - 用户登录
 - 向ZeroNet发布新内容
 - SQL数据库插入和查询
 - 实时更新网站数据
