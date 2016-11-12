# 安装 ZeroNet

* 下载 ZeroBundle 包：[Microsoft Windows](https://github.com/HelloZeroNet/ZeroBundle/releases/download/0.1.1/ZeroBundle-v0.1.1.zip), [Apple OS X](https://github.com/HelloZeroNet/ZeroBundle/releases/download/0.1.1/ZeroBundle-mac-v0.1.1.zip), [Linux 64bit](https://github.com/HelloZeroNet/ZeroBundle/releases/download/0.1.1/ZeroBundle-linux64-v0.1.1.tar.gz), [Linux 32bit](https://github.com/HelloZeroNet/ZeroBundle/releases/download/0.1.1/ZeroBundle-linux32-v0.1.1.tar.gz)
* 解压到任意目录
* 运行 `ZeroNet.cmd` (win), `ZeroNet(.app)` (osx), `ZeroNet.sh` (linux)

它将下载最新版本的 ZeroNet 并自动启动。

#### 手工为 Debian Linux 安装

* `sudo apt-get update`
* `sudo apt-get install msgpack-python python-gevent`
* `wget https://github.com/HelloZeroNet/ZeroNet/archive/master.tar.gz`
* `tar xvpfz master.tar.gz`
* `cd ZeroNet-master`
* 通过 `python zeronet.py` 启动
* 在你的浏览器中打开 http://127.0.0.1:43110/

### [Vagrant](https://www.vagrantup.com/)

* `vagrant up`
* 通过 `vagrant ssh` 访问虚拟机
* `cd /vagrant`
* 执行 `python zeronet.py --ui_ip 0.0.0.0`
* 在你的浏览器中打开 http://127.0.0.1:43110/

### [Docker](https://www.docker.com/)
* `docker run -d -v <local_data_folder>:/root/data -p 15441:15441 -p 43110:43110 nofish/zeronet`
* 在你的浏览器中打开 http://127.0.0.1:43110/

### [Virtualenv](https://virtualenv.readthedocs.org/en/latest/)

* `virtualenv env`
* `source env/bin/activate`
* `pip install msgpack-python gevent`
* `python zeronet.py`
* 在你的浏览器中打开 http://127.0.0.1:43110/

 
