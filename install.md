# 如何科学高效地安装Arch

## ![icon](./img/arch.ico) 万能的Arch Wiki

就我个人来说，我是通过第三方的教程才了解到Arch Wiki这个厉害的百科。官方其实有非常详细地 ~~用英语~~ 手把手教你怎么安装系统、怎么安装驱动、怎么个性化UI等等，霎时间我才明白这个门槛想让我明白的道理：
>**当你冷静下来耐心去认真阅读Wiki，很多问题其实都可以自己解决，浮躁解决不了问题。**

这里贴出来[ArchWiki](https://wiki.archlinux.org/)，有时候即使不是Arch系的Linux，遇到麻烦可以在这个Wiki里面找找，说不定会给你解决的思路。

下面就是我整理的安装的步骤了，虽然来源是第三方，但基本上都是参考官方的Wiki。这里省去了一些比较罕见的安装条件，乍一看很繁琐，仔细看其实是有很强的逻辑性的，建议顺序阅读。

## ![icon](./img/arch.ico) 前期准备

关于下载iso镜像就不多赘述，刷写用的U盘最好是8G以上，不过鉴于Arch苗条的安装镜像，或许4G也可以，但是*本人没有测试过*。刷写工具推荐使用[Rufus](http://rufus.ie/)，界面简洁明了，还可以自动帮你配置好U盘的启动引导。

准备🆗之后就可以重启电脑，怎么选启动设备这里不过多赘述，如果必要的话可以去BIOS里把U盘设为启动首选项，选择U盘启动，等待Arch安装盘启动，待见到`root@archiso~#`差不多就算是前期准备完成了

>注：安装盘默认使用root用户登录，所以这里开始一段时间内的指令默认都是#下执行，如果不明白我在说什么直接按照代码框里面的内容输入就可以了

## ![icon](./img/arch.ico) 硬件配置

这里的硬件包括网络配置和硬盘配置。

>啊啊，忘了说了，没有网络连接的话是不能安装Arch的，不过看到这里一般都是由网络连接的吧？

### 有线网络连接

这个就很ez了，只需输入

```bash
dhcpcd
```

提示文字在没有出现错误的时候不用管，接着只用ping一下某个网站验证网络连接成功就好了，参考代码如下

```bash
ping baidu.com
```

但是这个命令默认不会自己中止的，确认连接没有问题之后使用组合键`Ctrl`+`C`就可以中止ping指令的运行了

>注：一般情况下，`Ctrl`+`C`组合键的功能都是强制终止当前命令，后面的过程可能会遇到网络问题下载卡住，到时候可以用这个组合键强制终止，然后重新输入终止的指令即可

### 无线网路连接

和上面的不太一样，我们使用如下命令

```bash
wifi-menu
```

然后跟随引导选择可用的WiFi，连接后同样执行一下ping指令确保网络连接正常，然后我们就可以进行下一步了

### 联网更新系统时间和日期

输入指令即可，无需多言

```bash
timedatectl set-ntp true
```

### 磁盘配置

>注：这里默认是使用全盘安装，所以下文会包含创建分区表以及分区的操作，如果你是单硬盘多分区的操作，我会在下文需要区分的操作进行说明

首先我们需要确认你的系统启动方式，输入如下指令

```bash
ls /sys/firmware/efi/efivars
```

如果命令没有错误地显示了目录，则系统以UEFI模式启动.如果目录不存在，系统则以BIOS模式（或CSM模式，就是UEFI模拟的BIOS兼容模式）启动。

>注：不同的启动模式对应不同的分区方式，详见下文磁盘编辑的表格

现在我们要罗列出你的磁盘信息，使用如下命令

```bash
fdisk -l
```

这时你可以从返回信息里看到磁盘及其分区情况，找到你想要安装Arch的磁盘，记住磁盘号，一般会被系统记录为如下格式

```
/dev/sdX 
注：上面的X一般为字母a,b,c...
```

找到磁盘后（这里假定为`/dev/sdX`，X根据实际情况更改）使用如下指令开始对磁盘进行分区，你会进入fdisk的编辑模式

```bash
fdisk /dev/sdX
```

这里会有两个分支，分别对应旧式的MBR分区表和现在常见的GPT分区表，对应操作（基本上就是输入）请看如下表格（一些解释其实就是回显的翻译，能看懂英文也可以直接看命令的回显，别嫌我太啰嗦就好😝）

|分区表|步骤1：创建分区表|步骤2：分区操作|步骤3：格式化分区|步骤4：挂载分区|
|:------:|:------:|:------:|:------:|:------:|
|BIOS模式对应MBR|`o`创建新MBR分区表|`n`新建分区，`Enter`使用默认分区序号Y（Y视实际情况而定，一般为数字1,2,3...），`Enter`使用默认开始扇区，`Enter`使用默认结束扇区，`w`保存分区退出编辑模式|`mkfs.ext4 /dev/sdXY`格式化系统分区|`mount /dev/sdXY /mnt`挂载系统分区|
|UEFI模式对应GPT|`g`创建新GPT分区表|`n`新建分区，`Enter`使用默认分区序号Y（Y视实际情况而定，一般为数字1,2,3...），`Enter`使用默认开始扇区,`+512M`设定分区大小为512MB（官方建议至少为260MB，可以自行调整大小但不推荐），`t`默认对刚才新建的分区转换文件系统，`1`更换分区文件系统为EFI System；`n`新建分区，`Enter`使用默认分区序号Z（Z视实际情况而定，一般为数字1,2,3...且大于Y），`Enter`使用默认开始扇区，`Enter`使用默认结束扇区，`w`保存分区退出编辑模式|`mkfs.fat -F32 /dev/sdXY`格式化EFI分区，`mkfs.ext4 /dev/sdXZ`格式化系统分区|`mount /dev/sdXZ /mnt`挂载系统分区，`mkdir /mnt/boot`用于挂载EFI分区，`mount /dev/sdXY /mnt/boot`挂载EFI分区|

>注：如果你需要针对分区进行安装，你可以跳过上述的步骤1或者步骤1和2，步骤3中只需要选中你的目标分区进行操作即可，步骤4操作同理

很显然，GPT分区表在这里操作起来比较繁琐，我已经尽力去简化了，比如官方说到的swap分区（上古时期RAM很少的时候，需要给磁盘划分swap分区来储存超出RAM容量的进程）由于当下电脑内存一般都够用就被我省略了。

好消息是，经过这一步，最复杂的部分可以说是过去了，往下的操作很少会到这个境界了😀

## ![icon](./img/arch.ico) 系统安装

### 配置包管理器源

一边写这篇文章一边在虚拟机里再实操安装Arch，而写到这里我发现，到现在，也就是2021.1.12为止，pacman已经会在连接到网络后自动对源的顺序进行排序了（官方说是一个名叫reflector的服务对70个近期同步过的镜像服务器进行连接速度测试，速度快的服务器会自动被排列mirrorlist前端），也就是说，曾经的手动更改mirrorlist已经成为历史了。现在系统会自动帮你找到最快的镜像源，所以这一步已经可以跳过了。

### 安装核心组件

执行如下指令，pacman会自动将系统需要的核心组件安装到系统分区

```bash
pacstrap /mnt base base-devel linux linux-firmware
```

>解释一下，官方指出最基础包只有base，linux和linux-firmware，因为后期使用AUR需要用到base-devel所以这里一并安装了。  
另外这里引用百度[archlinux吧](https://tieba.baidu.com/f?kw=archlinux&ie=utf-8)吧主[@天苯](https://tieba.baidu.com/home/main?un=%E5%A4%A9%E8%8B%AF&ie=utf-8&id=tb.1.bed08632.91duuyO9Oud9CDW60fhmGg&fr=pb&ie=utf-8)的经验以供参考：
>>要用AUR的话需要base-devel  
内核可自己选，一般是linux，用linux-lts或者别的自定义内核均可  
然后是文本编辑器，nano，vim，emacs什么的自选  
文件系统管理器，ext4什么的就e2fsprogs，有其他需要的话的像lvm支持，btrfs，xfs等等文件系统看相应的Wiki词条就是  
网络管理是相对麻烦些，netctl，dhcpcd，wpa_supplicant，networkmanager自选，然后看相应的词条就是  
纯命令行连wifi的话大概是netctl iw dialog wpa_supplicant wpa_actiond这些

等待下载之余，你可以继续听我叨叨。

再早些时候，这里其实不需要手动安装Linux内核的，这里还有个故事：很久以前我有一阵没折腾Arch，时隔很久又一次安装，结果GRUB死活检测不到新安装的Arch，百度不到去Google，结果还是折腾了一晚上才通过相似的问题找到某个论坛（已经忘记了是哪个了，好像是Reddit）的一个回复，说试试安装Linux内核，结果还真解决了问题。原来是Arch官方修改了策略，需要手动去安装内核了。我再一次犯了不看官方Wiki的错误，这里也再次提醒一下:

***如果出现了和本篇教程里不一样的情况或者出现了没有提及的问题，请参考[官方安装指导](https://wiki.archlinux.org/index.php/Installation_guide)***

自动化安装完成之后，我们就可以进行下一步了。

### 生成fstab信息

>fstab是一个静态储存文件系统信息的文件，储存于/etc/fstab。当系统启动的时候，系统会自动地从这个文件读取信息，并且会自动将此文件中指定的文件系统挂载到指定的目录。

我们需要做的，是执行下面的指令

```bash
genfstab -L /mnt >> /mnt/etc/fstab
```

### 进入新系统

这里我们需要用下面的指令来进入新安装的Arch来继续配置

```bash
arch-chroot /mnt
```

这时你的界面会有一点点变化，原来有亮红色root的`root@archiso~#`变成了纯白色的`[root@archiso /]#`，说明现在已经是通过新安装的Arch在执行操作了，接下来就可以对新系统进行配置了。

### 配置新系统

执行以下命令配置系统时区

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

执行以下命令生成相关文件

```bash
hwclock --systohc
```

为了方便后期的配置，这里我们先安装一些必要的软件包

```bash
pacman -S nano vim dialog wpa_supplicant ntfs-3g networkmanager
```

安装好之后我们就可以开始配置系统的本地化了

```bash
nano /etc/locale.gen
```

这里会打开nano文本编辑器，大致操作方式是键盘的方向键控制光标位置，剩下的操作和一般的文本编辑器差不多，包括`Backspace`回删和组合键`Ctrl`+`S`保存也是可以使用的。

这里我们用键盘移动光标找到如下内容，删去前面的#注释开头，等同于给系统添加上对应的语言

```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
```

>注：这里也可以自己选择性添加其他需要的语言，建议选用UTF-8编码

然后使用组合键`Ctrl`+`S`保存更改，再使用组合键`Ctrl`+`X`退出编辑器，接着进行本地化，执行如下指令

```bash
locale-gen
```

然后我们现在需要编辑系统的显示语言，这里有两个选项  

1. 就头铁用英文，这样不需要安装中文字体库，还可以锻炼英语😂  
这样的话只需要直接编辑`/etc/locale.conf`  
```bash
nano /etc/locale.conf
```  
输入  
```
LANG=en_US.UTF-8
```
然后保存退出即可

2. 选择母语，安装中文字体库，那么需要进行如下操作  
```bash
pacman -S wqy-microhei wqy-microhei-lite wqy-bitmapfont wqy-zenhei ttf-arphic-ukai ttf-arphic-uming adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts noto-fonts-cjk
```  
等待其安装好字体库之后同样编辑`/etc/locale.conf`  
```bash
nano /etc/locale.conf
```  
这里输入变成  
```
LANG=zh_CN.UTF-8
```  
然后保存退出即可

接着我们来配置主机名称，指令如下

```bash
nano /etc/hostname
```

然后输入你想要的名字，保存退出就好了

配置好主机名，接下来需要配置hosts文件，编辑`/etc/hosts`，指令如下

 ```bash
 nano /etc/hosts
 ```

 输入如下内容

 ```
 127.0.0.1  localhost
 ::1        localhost
 127.0.1.1  你的主机名.localdomain 你的主机名
 ```

接下来，设置#超级用户（root）密码，输入

```bash
passwd
```

然后我们来到了整个过程中第二麻烦的步骤：配置引导。计划是使用GRUB进行引导，考虑到后期折腾双系统，我们还需要安装一些软件包，指令如下

```bash
pacman -S os-prober
```

下面关于GRUB的配置一样有MBR和GPT两个分支，对应的指令我仍然用表格呈现

|分区表|步骤1：安装GRUB组件|步骤2：安装GRUB为磁盘主引导|步骤3：生成GRUB配置文件|
|:------:|:------:|:------:|:------:|
|MBR|`pacman -S grub`|`grub-install --target=i386-pc /dev/sdX`|`grub-mkconfig -o /boot/grub/grub.cfg`|
|GPT|`pacman -S grub efibootmgr`|`grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub`|`grub-mkconfig -o /boot/grub/grub.config`|

注：目前我还没有去研究怎样手动把GRUB安装在某个分区同时保证GRUB可用，换言之，目前还没办法做到用Windows的BCD引导来双系统，只能用GRUB去拉起Windows引导，Windows的快速启动也会受影响，因而双系统情况下可能导致Windows启动速度比正常情况下慢一些

## ![icon](./img/arch.ico) 安装后配置

接下来就没有难题了，可以大胆看到最后了！

### 添加用户

要知道刚才一直在用#，也就是所谓的超级用户（拥有最高权限），现在我们要创建一个日常使用的普通用户，代码如下

```bash
useradd -m -G wheel 用户名
```

>注：这里将用户添加到了wheel群组里面，这样接下来配置sudo分配权限会简单一些

给你创建的用户设置密码

```bash
passwd 用户名
```

### 安装配置sudo

我们需要安装sudo软件包以便日常我们使用普通用户也可以通过这个指令获取#权限

```bash
pacman -S sudo
```

安装完成后我们同样需要对其进行配置，正如上文所言，我们要允许wheel群组内用户获取#权限，但这里情况不太一样，我们先执行下面的指令

```bash
visudo
```

这时又进入了编辑模式，但是这次的编辑器是vi，vi编辑器默认会先进入**命令模式**，我们需要输入`i`来进入**输入模式**，然后我们可以使用方向键控制光标，我们需要找到如下表述

```
## Uncomment to allow members of group...（后面的省略，只需要看每行开头）
## %wheel ALL=(ALL)ALL...
```

删除`%wheel`前面的##注释符，这里`Backspace`不管用，需要用`Del`删除，你会发现原本白色的文字突然被识别然后变成不同颜色，这时我们需要按下`Escape`退出**输入模式**而进入**命令模式**，然后输入`:`（也就是`Shift`+`;`
），这时光标会自动出现在最底一行，这时我们输入`wq`然后回车就可以保存编辑并退出了

### CPU微码补丁

Intel和AMD的CPU的机器（X86架构平台还有第三家吗？那就都装吧）还需要安装一个微码软件包来提高系统稳定性，我用表格呈现需要输入的指令

|CPU提供商|对应指令|
|:------:|:------:|
|Intel|`pacman -S intel-ucode`|
|AMD|`pacman -S amd-ucode`|

### 安装显卡驱动

这里会有三个分支，这里还是用表格呈现需要输入的指令

|GPU提供商|对应指令|
|:------:|:------:|
|Intel|`pacman -S xf86-video-intel`|
|AMD|`pacman -S xf86-video-amdgpu xf86-video-ati`
|Nvidia（仅限开源驱动，闭源驱动这里不多赘述）|`pacman -S xf86-video-nouveau`

### 安装桌面环境

桌面的实现方式又超多种，这里介绍最好配置的X桌面系统，以及对应三种可用的桌面环境

首先要安装X桌面系统，代码如下

```bash
pacman -S xorg
```

可供选择的三款桌面我继续使用表格来表示需要的指令

|DE名称|桌面优势|桌面缺陷|对应指令|
|:------:|:------:|:------:|:------:|
|Xfce|轻量，占用资源少，占用磁盘空间小，可自定义空间大|动画简陋，界面老旧，依赖后期美化|`pacman -S xfce4 xfce4-goodies`|
|KDE|现代而美观，动画优美，可自定义空间大|占用很大磁盘空间，占用较多资源|`pacman -S plasma kde-applications`|
|DeepinDE|源自Deepin，设计美观并且比较轻量|可自定义空间比较小|`pacman -S deepin-extra`|

安装好桌面环境之后，我们还需要配置窗口管理器，这里我们选用sddm，兼容上述三个桌面环境并且后期比较好配置

```bash
pacman -S sddm
```

然后设置开机启动sddm

```bash
systemctl enable sddm
```

同时由于有了图形化界面，原来的命令行用网络连接服务可以禁用并启用图形界面使用的服务

```bash 
systemctl disable netctl
systemctl enable NetworkManager
```

最后的最后，为了避免出现桌面环境中网络相关小图标缺失，我们安装一个图标包

```bash
pacman -S network-manager-applet
```

## ![icon](./img/arch.ico) 完成安装

到这里，Arch的基础安装就算是结束了，现在我们可以退出安装盘然后重启进入新系统了！

```bash
exit
reboot
```

这篇文章到此终于可以画上一个句号，全文近一万字符，花了几天才完成，可能是我效率不够吧。这次算是好好熟悉了一下Markdown语法，我们下篇文章再见😌
