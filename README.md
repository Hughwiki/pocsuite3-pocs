# pocsuite3-pocs

---
# 安装

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


# 架构解析
为了使用的更加顺畅，有必要了解下框架的架构。整体而言，本框架主要包含四个部分，分别是目标收集、PoC 脚本加载、多线程检测、结果汇总。
<br/>
如下图所示：
![image](https://user-images.githubusercontent.com/118670924/202912824-d3542f3d-c6d9-427a-8c85-2437057424d6.png)

* *-u / --url* 指定单个 URL 或者 CIDR，支持 IPv4 / IPv6。使用 -p 参数可以提供额外的端口，配合 CIDR 可以很方便的探测一个目标网段
```
pocsuite -r poc.py -u https://example.com
pocsuite -r poc.py -u fd12:3456:789a:1::/120
pocsuite -r poc.py -u 172.16.218.1/24
pocsuite -r poc.py -u "https://[fd12:3456:789a:1::f0]:8443/test"
```

* *-f / --file* 指定一个文件，将多个 URL / CIDR 存到文件中，每行一个
```
# this is url.txt
172.16.218.1/24
https://example.com
# localhost
```
`pocsuite -r poc.py -f url.txt`


