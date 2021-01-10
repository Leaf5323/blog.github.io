# 使用QEMU虚拟机加KVM虚拟化在Linux平台部署黑苹果

[@foxlet](https://github.com/foxlet)在其[macOS-Simple-KVM](https://github.com/foxlet/macOS-Simple-KVM)仓库里有十分清楚的解释，为了后期阅读的方便，我对其步骤进行了翻译并记录此文。

## 虚拟化与黑苹果

早期的黑苹果，都是在实体机上直接安装以最大化利用硬件的性能。随着虚拟化技术的成熟，虚拟机造成的性能损越来越小，于是在虚拟机上折腾黑苹果开始成为新的趋势。这里我们使用的虚拟化接口是KVM，使用的虚拟机平台则是QEMU。QEMU是开源的虚拟平台，同时对Linux多个发行版的支持也较好。下面我们直接进入正题。

## QEMU等所需软件的安装

这里收录了Debain系、Arch系、Void Linux、openSUSE、Fedoras和Gentoo的安装指令，可以根据自己的发行版复制对应的指令，在终端粘贴后执行。

#### Debain系（如Ubuntu, Debian, Mint, PopOS）

```bash
sudo apt-get install qemu-system qemu-utils python3 python3-pip
```

#### Arch系（如Manjaro, Arch）

```bash
sudo pacman -S qemu python python-pip python-wheel
```

#### Void Linux

```bash
sudo xbps-install -Su qemu python3 python3-pip
```

#### openSUSE Tumbleweed

```bash
sudo zypper in qemu-tools qemu-kvm qemu-x86 qemu-audio-pa python3-pip
```

#### Fedora

```bash
sudo dnf install qemu qemu-img python3 python3-pip
```

#### Gentoo

```bash
sudo emerge -a qemu python:3.4 pip
```

在完成了上面的软件的安装和配置之后,记得克隆[这个仓库](https://github.com/foxlet/macOS-Simple-KVM)，国内git会慢一点，解决办法有：

* 换用手机数据流量下的热点

* 科学上网

* 上面两个都不方便的话，耐心等一下吧

## 开始配置

### 第一步

终端cd到克隆下来的文件夹，运行`jumpstart.sh`



