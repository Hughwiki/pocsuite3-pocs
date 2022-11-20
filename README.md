# pocsuite3-pocs

---
# 1、 安装

Pocsuite3 基于 Python3 开发，可以运行在支持 Python 3.7+ 的任何平台上，例如 Linux、Windows、MacOS、BSD 等。

2021 年 11 月，Pocsuite3 通过了 Debian 官方的代码及合规检查，正式加入 Debian、Ubuntu、Kali 等 Linux 发行版的软件仓库，可以通过 apt 命令一键获取。此外，Pocsuite3 也已经推送到 Python PyPi、MacOS 的 Homebrew 仓库、Archlinux 的 Aur 仓库、Dockerhub。

## 使用 Python3 pip 安装
```
pip3 install pocsuite3
```

## 使用国内镜像加速
```
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple pocsuite3
```


## 在 MacOS 上安装
```
brew update
brew info pocsuite3
brew install pocsuite3
```


## Debian, Ubuntu, Kali
由于 Linux APT 软件仓库机制的原因，通过 apt 安装的软件包可能略低于上游版本。
```
sudo apt update
sudo apt install pocsuite3

```

## Docker
```
docker run -it pocsuite3/pocsuite3
```


## Arch Linux
```
yay pocsuite3
```


## 源码安装#
```
wget https://github.com/knownsec/pocsuite3/archive/master.zip
unzip master.zip
cd pocsuite3-master
pip3 install -r requirements.txt
python3 setup.py install
```
---


# 2、 架构解析
为了使用的更加顺畅，有必要了解下框架的架构。整体而言，本框架主要包含四个部分，分别是目标收集、PoC 脚本加载、多线程检测、结果汇总。
<br/>
如下图所示：
![image](https://user-images.githubusercontent.com/118670924/202912824-d3542f3d-c6d9-427a-8c85-2437057424d6.png)

---


# 3、 命令参数解析

## 单个目标检测

* **-u / --url** : 指定单个 URL 或者 CIDR，支持 IPv4 / IPv6。使用 -p 参数可以提供额外的端口，配合 CIDR 可以很方便的探测一个目标网段
```
pocsuite -r poc.py -u https://example.com
pocsuite -r poc.py -u fd12:3456:789a:1::/120
pocsuite -r poc.py -u 172.16.218.1/24
pocsuite -r poc.py -u "https://[fd12:3456:789a:1::f0]:8443/test"
```

## 批量目标检测

* **-f / --file** : 指定一个文件，将多个 URL / CIDR 存到文件中，每行一个
```
# this is url.txt
172.16.218.1/24
https://example.com
# localhost
```
```pocsuite -r poc.py -f url.txt```

## 指定端口、协议

* **-p / --ports** : 为 URL 或 CIDR 添加额外端口，格式：[协议:]端口, 协议是可选的，多个端口间以 , 分隔。

例如：pocsuite -r poc.py -u 172.16.218.1/31 -p 8080,https:8443 会加载以下目标。

```
172.16.218.0
172.16.218.0:8080
https://172.16.218.0:8443
172.16.218.1
172.16.218.1:8080
https://172.16.218.1:8443

```

## 修改默认端口协议

* **-s**
使用-s 参数可以不加载 target 本身的端口，只使用 -p 提供的端口。

例如：pocsuite -r poc.py -u 172.16.218.1/31 -p 8080,https:8443 -s 会加载以下目标。

```
172.16.218.0:8080
https://172.16.218.0:8443
172.16.218.1:8080
https://172.16.218.1:8443
```

## 搜索引擎查询

* **--dork-fofa** / **--fofa-user** / **--fofa-token**
通过 Fofa API 批量获取测试目标。

首次使用会提示输入 Fofa user email 和 Fofa API Key，验证可用后会保存到 $HOME/.pocsuiterc 文件中，除非 token 过期，下次使用不会重复询问，也可使用 --fofa-user 和 --fofa-token 参数提供。单页检索数量为 100。
```
pocsuite -r poc.py --dork-fofa 'thinkphp'

...
[16:33:23] [INFO] {"error":false,"email":"***","username":"***","fcoin":48,"isvip":true,"vip_level":2,"is_verified":false,"avatar":"https://nosec.org/missing.jpg","message":"","fofacli_ver":"4.0.3","fofa_server":true}
[16:33:23] [INFO] [PLUGIN] try fetch targets from Fofa with dork: thinkphp
[16:33:25] [INFO] [PLUGIN] got 88 target(s) from Fofa
[16:33:25] [INFO] pocsusite got a total of 88 tasks
[16:33:25] [INFO] starting 88 threads
```

## POC 脚本加载 指定一个或多个 poc/poc路径
* -r ：指定一个或多个 PoC 路径（或目录），如果提供的是目录，框架将遍历目录然后加载所有符合条件的 PoC。多个路径或目录之间用空格分隔。

```
# 加载单个 PoC 文件
pocsuite -r ecshop_rce.py

# 加载多个 PoC 文件
pocsuite -r pocsuite3/pocs/ecshop_rce.py pocsuite3/pocs/thinkphp_rce.py pocsuite3/pocs/wd_nas_login_bypass_rce.py

# 从目录加载
pocsuite -r pocsuite3/pocs

# 加载 nuclei template
pocsuite -r ./nuclei-templates/cves/2020/CVE-2020-14883.yaml

```

## POC 筛选
* -k : 指定关键词（支持正则）对 PoC 进行筛选，如组件名称、CVE 编号等。如果我们确认了目标组件，就可以用 -k 选项找到所以对应的 PoC 对目标进行批量测试。如果只提供了 -k 选项，-r 默认为 Pocsuite3 自带的 pocsuite3/pocs 目录。

```
pocsuite -r ./pocsuite3/pocs -k thinkphp
...
[17:11:05] [INFO] loading PoC script './pocsuite3/pocs/thinkphp_rce.py'
[17:11:06] [INFO] loading PoC script './pocsuite3/pocs/thinkphp_rce2.py'
...

```

## 组件或框架历史漏洞查询
* --vul-keyword / --ssv-id / --seebug-token
通过 Seebug API 读取指定组件或者类型的漏洞的 PoC。

首次使用会提示输入 Seebug API key，验证可用后会保存到 $HOME/.pocsuiterc 文件中，除非 token 过期，下次使用不会重复询问，也可使用 --seebug-token 参数提供。

```
# 通过关键词加载
pocsuite --vul-keyword redis

# 通过漏洞编号加载
pocsuite --ssv-id 89715


```
---


# 4、 运行控制

## --threads ：线程控制
线程池大小控制，默认为 Min(150, 目标总数)。

## --pcap ： 保存通信流量为.pacb文件

在运行 PoC 时使用 --pcap 参数，可以将通信流量保存为 pcap 文件。

![image](https://user-images.githubusercontent.com/118670924/202914082-9c9f3a14-1073-455e-836c-dbe27e32a9e3.png)

通过 wireshark 打开该文件进行流量分析。

![image](https://user-images.githubusercontent.com/118670924/202914127-cd2d2bc0-5560-45ee-845a-75beca0db1e5.png)



## 三种模式： 验证、攻击、shell

### --verify

验证模式，执行 PoC 脚本的 _verify() 方法， 进行漏洞验证。

### --attack

攻击模式，执行 PoC 脚本的 _attack() 方法，具体表现取决于方法的实现。

### --shell / --lhost / --lport / --tls

shell 模式，执行 PoC 脚本的 _shell() 方法，控制台会进入 shell 交互模式执行命令及获取输出。

Pocsuite3 在 shell 模式会默认监听本机的 6666 端（可通过 --lhost、--lport 修改），编写对应的攻击代码，让目标执行反向连接运行 Pocsuite3 系统 IP 的 6666 端口即可得到一个 shell。

#### Tips
`在 PoC 脚本中，attack 模式和 shell 模式的实现是可选的， 如果不指定运行模式，默认是 verify。`

---

# 5、 pocsuite 控制台参数说明

使用命令 poc-console 进去交互式控制台后，可通过 help 命令查看帮助，list 或 show all 列出所有 PoC 模块，use 加载某指定模块。相关参数可通过 set / setg 命令设置。

如下：
![image](https://user-images.githubusercontent.com/118670924/202914409-761e0be4-42e5-4fe5-a64e-6fb811d7a92a.png)
![image](https://user-images.githubusercontent.com/118670924/202914468-2077d1f2-2581-4123-84d1-57152a29ffbd.png)


在控制台中也可以执行系统命令。

---
