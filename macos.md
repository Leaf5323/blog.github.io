# 使用QEMU虚拟机加KVM虚拟化在Linux平台部署黑苹果

[@foxlet](https://github.com/foxlet)在其[macOS-Simple-KVM](https://github.com/foxlet/macOS-Simple-KVM)仓库里有十分清楚的解释，为了后期阅读的方便，我对其步骤进行了翻译与简单的整理并记录此文。

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

终端cd到克隆下来的文件夹，运行`jumpstart.sh`来下载所需的macOS镜像，代码示例如下

```bash
./jumpstart.sh --mojave
```

>注：你可以更改附加值来获取不同版本的macOS镜像，上面的例子`--mojave`获取10.14版本镜像，如更改为`--high-sierra`获取10.13版本镜像，更改为`--catalina`获取10.15版本镜像。  
获取的镜像应为`BaseSystem.img`，如果你已经拥有该镜像可以跳过这一步骤。另外，如果下载得到为`BaseSystem.dmg`，你需要使用`dmg2img`工具来进行格式转换。

### 第二步

首先，我们需要创建一个磁盘，这里仍然使用终端命令执行，代码如下

 ```bash
qemu-img create -f qcow2 MyDisk.qcow2 64G
 ```

然后，我们需要在克隆的文件`basic.sh`的末尾添加如下内容

```
-drive id=SystemDisk,if=none,file=MyDisk.qcow2 \
-device ide-hd,bus=sata.4,drive=SystemDisk \
```

>注：如果你使用无监视器的机器(headless system)如远程云端机器，你需要继续添加`-nographic`和`-vnc :0 -k en-us`来使用VNC远程连接。

接着，只需要运行`basic.sh`就可以启动并正常安装虚拟机了。

>***Q:发现安装时找不到硬盘？***  
>***A:记得先使用“磁盘工具”分区！***

### 选择项a 使用Virt-Manager对虚拟机进行管理

你可以把整个虚拟机导入到Virt-Manager里以便后期的硬件变动和性能优化，同时你需要进行如下两个步骤

1.在终端输入

```bash
sudo ./make.sh --add
```

2.执行上述命令之后，在新添加的虚拟机的“属性”里把`MyDisk.qcow2`添加为储存盘

### 选择项b 使用无监视器的机器(headless system)

如要使用无监视器的机器或者云端设备，你可以通过添加附加值给`headless.sh`来配置你的虚拟机硬件，示例如下

```bash
HEADLESS=1 MEM=1G CPUS=2 SYSTEM_DISK=MyDisk.qcow2 ./headless.sh
```

同时，该命令会默认开启端口为5900的VNC服务器

### 第三步

安装完成了！

但是只是完成了最基础的安装，所以当前的黑苹果会非常卡顿并且没有网络连接与声音设备

克隆得到的文件中有名为`docs`的文件夹，里面会详细介绍优化虚拟机性能的方法，如怎样添加运行内存，怎样桥接网络，怎样直通GPU，怎样修改屏幕分辨率，怎样开启声音设备等等

本文到此作结，翻译工作尽量本土化了，后期有时间的话再进一步翻译一下后期虚拟机安装完成之后的优化教程！🤣