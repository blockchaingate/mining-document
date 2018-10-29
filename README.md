# 发链中文挖矿教程

## 显卡的选择

现在发布的挖矿程序暂时只有Ubuntu的版本支持`GPU`挖矿（Windows和Mac OSX暂时只支持`CPU`挖矿），因为CPU和GPU挖矿效率相差巨大，所以请尽量选择Ubuntu作为矿机的操作系统。发链挖矿使用的是Equihash算法，所以N卡的挖矿效率要优于A卡，建议选择N卡来进行挖矿，以下教程主要讲述如何在Ubuntu下用N卡挖矿。

## 文章中提到的一些基础操作

### 如何打开终端

1. 在搜索框中输入Terminal来搜索终端

![search terminal]()

2. 运行终端，结果如下图

![run terminal]()

![terminal]()

## Ubuntu

### 配置挖矿环境（可以跳过这一步如果你已经有一台之前配置好的矿机）

1. 首先请先安装`Ubuntu 16.04`作为矿机的操作系统，具体教程请参见这里[这里](http://forum.ubuntu.org.cn/viewtopic.php?t=478527)。

2. 然后我们需要安装Nvidia的显卡驱动，显卡驱动安装完成后请重启你的电脑，

```zsh
fab@ubuntu:~$ sudo apt purge nvidia*
fab@ubuntu:~$ sudo add-apt-repository ppa:graphics-drivers/ppa
fab@ubuntu:~$ sudo apt update
fab@ubuntu:~$ sudo apt install nvidia-410
```

3. 重启完成后在终端中输入以下命令确认驱动安装成功，如果可以看到类似于下面的结果那么证明驱动已经安装成功了。

```zsh
fab@ubuntu:~$ nvidia-smi
Mon Oct 29 12:37:27 2018       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 410.73                 Driver Version: 410.73                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Quadro K2000        Off  | 00000000:05:00.0  On |                  N/A |
| 30%   31C    P8    N/A /  N/A |    479MiB /  1997MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      1303      G   /usr/lib/xorg/Xorg                           276MiB |
|    0      3263      G   ...-token=44F696E2E4AACB3F653963E6CC8D6A29    13MiB |
|    0     10725      G   ...uest-channel-token=11123812141224894328   105MiB |
|    0     29245      G   budgie-wm                                     79MiB |
+-----------------------------------------------------------------------------+
```

4.（请执行此步骤如果你的显卡支持CUDA）
在命令行里输入以下命令来安装`CUDA`

```zsh
fab@ubuntu:~$ cd ~
fab@ubuntu:~$ wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_10.0.130-1_amd64.deb
fab@ubuntu:~$ sudo dpkg -i cuda-repo-ubuntu1604_10.0.130-1_amd64.deb
fab@ubuntu:~$ sudo apt-key adv --fetch-keys http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub
fab@ubuntu:~$ sudo apt update
fab@ubuntu:~$ sudo apt install -y cuda
```

5. 安装必须的软件包

请依次在terminal中运行以下命令，检查每行的执行情况，如果有错误提示的话请解决完相应错误之后再执行后续命令，千万不要跳过。

```zsh
fab@ubuntu:~$ sudo add-apt-repository ppa:bitcoin/bitcoin
fab@ubuntu:~$ sudo apt update
fab@ubuntu:~$ sudo apt upgrade -y
fab@ubuntu:~$ sudo apt install -y build-essential libtool autotools-dev autoconf pkg-config libssl-dev
fab@ubuntu:~$ sudo apt install -y libboost-all-dev
fab@ubuntu:~$ sudo apt install -y libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler
fab@ubuntu:~$ sudo apt install -y libqrencode-dev
fab@ubuntu:~$ sudo apt install -y libminiupnpc-dev
fab@ubuntu:~$ sudo apt install -y libdb4.8-dev libdb4.8++-dev
fab@ubuntu:~$ sudo apt install -y libevent-dev
fab@ubuntu:~$ sudo apt install -y libsodium-dev
fab@ubuntu:~$ sudo apt install -y clinfo
fab@ubuntu:~$ groups
fab@ubuntu:~$ sudo usermod -a -G video $LOGNAME
```

### 检测挖矿环境

1. 首先先检测操作系统环境。打开`终端`，输入以下命令来检测Ubuntu系统版本, 确认`Distributor ID`为`Ubuntu`，`Release`为`16.04`. 如果结果和以下运行结果不一致请按照[这里](http://forum.ubuntu.org.cn/viewtopic.php?t=478527)的教程重新安装`Ubuntu 16.04`.

```zsh
fab@ubuntu:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu //系统分支
Description:    Ubuntu 16.04.5 LTS
Release:        16.04 //系统版本
Codename:       xenial
```

2. 然后检测显卡型号。在`终端`中输入以下命令来显示显卡信息，确认`vendor`为`NVIDIA Corporation`，如果结果不符合，请先确认显卡驱动已经被正确安装。关于如何在Ubuntu下安装显卡驱动，请参照上面配置挖矿环境第2步的教程。

```zsh 
fab@ubuntu:~$ lshw -c video
  *-display                 
       description: VGA compatible controller
       product: GK107GL [Quadro K2000] //你的显卡型号
       vendor: NVIDIA Corporation //显卡品牌
       physical id: 0
       bus info: pci@0000:05:00.0
       version: a1
       width: 64 bits
       clock: 33MHz
       capabilities: vga_controller bus_master cap_list rom
       configuration: driver=nvidia latency=0
       resources: irq:53 memory:ee000000-eeffffff memory:d0000000-dfffffff memory:e0000000-e1ffffff ioport:b000(size=128) memory:c0000-dffff
```

3. 检测您的显卡是否支持`CUDA`。在[这里](https://developer.nvidia.com/cuda-gpus)搜索你的显卡型号来确定是否你的显卡是否支持`CUDA`。进入网页后选择`CUDA-Enabled Geforce Products`，点击后会有一个列表来显示现在所有支持CUDA的Nvidia Geforce显卡，如果你的显卡在列表里，那么证明你的显卡支持CUDA。如果你使用的是Quadro或者是Tesla系列的显卡那么请点击对应的选项。如果你不知道你自己的显卡型号，可以去上一条的结果里的`product`项中获得，例如我的显卡就是`Quadro K2000`。

### 下载和配置挖矿程序

1. 登陆[fabcoin.pro](http://fabcoin.pro/runtime.html)来下载最新的挖矿程序。请根据你的显卡是否支持CUDA来确定应该下载哪个版本。如果支持CUDA的话可以下载支持CUDA版本的挖矿程序，如果不支持的话可以下载OpenCL版本的挖矿程序。

![download instruction](https://mmbiz.qpic.cn/mmbiz_png/fz0bOjnnvcahaUSZJKCqjzmvDY4oznq9ACDJF0uHacFVAKa69emicW4qAlzuhDFPXFgQic3qWjpPxOChTI1YxNdQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

2. 解压缩到挖矿程序到`$HOME/fab-coin`

3. 输入以下命令启动发币全节点钱包

```zsh
fab@ubuntu:~$ cd ~/fabcoin/bin
fab@ubuntu:~$ ./fabcoin-qt
```

4. 备份钱包并且加密：自动同步->最大化钱包界面->鼠标移动到左上角标题栏出现->设置->加密钱包（打开钱包的密码）->文件->备份钱包（请备份至U盘并妥善保管）

5. 输入以下命令以启动GPU挖矿

```zsh
fab@ubuntu:~$ cd ~/fabcoin/bin
fab@ubuntu:~$ ./fabcoind -daemon -gen -G -allgpu
```

### 检查挖矿状态

1. 在命令行中输入以下命令查看GPU使用状况来确定挖矿是否在进行，如果挖矿正常进行，GPU占用率应该在100%，并且在`processes`里应该可以看到`fabcoind`

```zsh
fab@ubuntu:~$ nvidia-smi
```

2. 在命令行中输入以下命令来查看挖矿收益

```zsh
fab@ubuntu:~$ cd ~/fabcoin/bin
fab@ubuntu:~$ ./fabcoin-cli getwalletinfo
```

### 结束挖矿

1. 在命令行中输入以下命令来结束挖矿

```zsh
fab@ubuntu:~$ killall fabcoind
```
