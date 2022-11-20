# pocsuite3-pocs

---
# 1. pocsuite3 安装


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

