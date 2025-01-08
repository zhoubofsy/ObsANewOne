# 创建调试环境

内核调试会使整个系统暂停，需要有一个物理链路连通才可以，通常采用串口调试。

  ## Parallels Desktop 

![[Pasted Graphic 2.png]]

创建一个虚拟机，用于安装需要调试的kenrel内核，增加一个“串行口”，设置源为`/tmp/sp2` 这是一个SOCKET文件，模式设置成“服务器”模式。在虚拟机（ARM64）中该串口被映射成`/dev/ttyAMA0`的设备。

  # 依赖包安装

  ## Ubuntu22

```shell

# apt update -y

# apt upgrade -y

# apt install -y build-essential libncurses-dev bison flex libssl-dev libelf-dev

```

  # 获取kernel源代码

  ## Ubuntu22

```shell

# apt install -y linux-source

# tar -xvf /usr/src/linux-source-<version>.tar.bz2 -C /usr/src/

```

  # 配置编译安装 kernel

## config配置

最小化配置

```shell

# cd /usr/src/linux-source-<version>

# make localmodconfig

```

打开`.config`文件，将签名相关配置关闭

```config

CONFIG_MODULE_SIG=n

CONFIG_MODULE_SIG_FORCE=n

CONFIG_MODULE_SIG_SHA512=n

```

执行`make menuconfig`设置自定义选项
## kernel编译安装

执行 `make -j$(nproc)` 开始编译内核，期间可能遇到关于`CONFIG_INFO_BTF`的问题，需要修改`.config`将其关闭`CONFIG_INFO_BTF=n`。

成功完成编译后执行`make modules_install`安装编译好的内核模块；执行`make install` 安装内核，期间需要关注硬盘空间情况，硬盘空间不足会导致安装失败尤其需要关注目录`/boot/` 和 `/usr/src/linux-source-<version>`或内核源代码目录。

# 调整Grub启动参数

成功安装后，`/boot/grub/grub.cfg`内会增加新内核启动项。

修改`/etc/default/grub`文件 

```config

GRUB_DEFAULT=0

#GRUB_TIMEOUT_STYLE=hidden

GRUB_TIMEOUT_STYLE=menu

GRUB_TIMEOUT=5

GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`

GRUB_CMDLINE_LINUX_DEFAULT="kgdboc=ttyAMA0,115200 nokalsr debug"

GRUB_CMDLINE_LINUX=""

```

修改后，使用`update-grub` 更新 `/boot/grub/grub.cfg`文件，然后重启系统。

系统重启后，使用命令`cat /sys/module/kgdboc/parameters/kgdboc`查看串口配置是否成功。

```shell

# cat /sys/module/kgdboc/parameters/kgdboc

ttyAMA0,115200

```

也可以不通过grub配置文件配置串口，可以手动临时修改

```shell

# echo “ttyAMA0,115200” > /sys/module/kgdboc/parameters/kgdboc

```

  
# kdb调试

## Parallels Desktop

在MacOS（M1）上，使用 socat 将SOCKET文件`/tmp/sp2` 映射到虚拟串口上，然后在使用  minicom 软件连接虚拟串口，进行调试。

### 安装socat 和 minimum

```shell

$ brew install scout

$ brew install minicom

```

### SOCKET文件映射虚拟串口

```shell

$ socat -v UNIX-CONNECT:/tmp/sp2 pty,link=/tmp/ttyS0

$ ls -alh /tmp/

……

srwxr-xr-x   1 zhoub  wheel     0B  1 29 10:43 sp2

lrwxr-xr-x   1 zhoub  wheel    12B  1 29 11:18 ttyS0 -> /dev/ttys015

……

```

此时已将 `/tmp/sp2`文件映射到串口设备 `/dev/ttys015` 上，此处生成的 `/tmp/ttyS0` 是其软链，后续可直接使用这个软链

### 连接虚拟串口等待调试

首先使用`minicom -s`配置串口

![[Pasted Graphic 3.png]]
  
然后使用 `minicom`通过串口登录

![[Pasted Graphic 4.png]]

## UTM

Todo…

## 调试

无论是那种虚拟机，都需要我们先进行串口登录，然后使用`echo g > /proc/sysrq-trigger` 触发系统中断，此时除串口终端外的所有其他终端全部Freeze，只能通过串口终端交互调试

  ![[Pasted Graphic 5.png]]

# gdb调试

## Parallels Desktop

![[Pasted Graphic 6.png]]  

创建一个虚拟机，安装上GDB，用于调试kenrel内核，增加一个“串行口”，设置源为`/tmp/sp2` 这是一个SOCKET文件，与上文中的SOCKET文件是同一个，模式设置成“客户端”模式。在虚拟机（ARM64）中该串口被映射成`/dev/ttyAMA0`的设备。

## UTM

Todo...

## 调试

整个GDB的调试在虚拟机中进行，kernel运行的环境需要使用`echo g > /proc/sysrq-trigger`中断内核运行，然后调试VM 找到 vmlinux，开始gdb调试

![[Pasted Graphic 7.png]]  

使用 `target remote /dev/ttyAMA0` 连接待调试内核

 ![[Pasted Graphic 8.png]]  

# 参考&鸣谢

[**linux内核调试（七）使用kdb/kgdb调试内核**]([https://zhuanlan.zhihu.com/p/546416941](https://zhuanlan.zhihu.com/p/546416941))
[**Ubuntu显示grub启动菜单以及修改默认启动项**]([https://zhuanlan.zhihu.com/p/552895466](https://zhuanlan.zhihu.com/p/552895466))
