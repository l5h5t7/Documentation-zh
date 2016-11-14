# 如果你想合作开发 ZeroNet 的代码标准
 - 跟随 [PEP8](https://www.python.org/dev/peps/pep-0008/)
 - 简单比复杂好
 - 过早优化是一切罪恶的根源

### 命名
 - 类名：大驼峰式命名法
 - 函数名：小驼峰式命名法
 - 变量名：小写字母，下划线命名法

### 变量
 - file_path: 相对于工作目录的文件路径 (data/17ib6teRqdVgjB698T4cD1zDXKgPqpkrMg/css/all.css)
 - inner_path: 相对于站点目录的文件路径 (css/all.css)
 - file_name: all.css
 - file: Python 文件对象
 - privatekey: 站点的私钥 （不含 _）

### 源文件目录和命名
 - 每个文件一个类更好
 - 源文件名和目录来自类名：WorkerManager class = Worker/WorkerManager.py
