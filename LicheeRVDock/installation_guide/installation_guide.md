## 使用 Licheerv 开发板安装 openEuler RISC-V 22.03 V2

注：
1. root 用户密码 `openEuler12#$`
2. openeuler用户密码 `openEuler12#$`

### 1. 准备硬件

1）LicheeRV 开发板：由 CNRV 获取得到开发板。

![LicheeRV](./img/LicheeRV.png)

2）32G micro-sd 卡及读卡器：Kingston TF/MicroSD 卡，容量 32 GB，速度U1，带读卡器。

![Kingston](./img/Kingston.png)

3）电源适配器及 type-c 线。

### 2. 准备系统镜像

D1 的系统镜像下载连接地址如下：https://mirror.iscas.ac.cn/openeuler-sig-riscv/openEuler-RISC-V/preview/openEuler-22.03-V2-riscv64/D1/

考虑到要安装验证Firefox浏览器，我们可以下载 openEuler-22.03-V2-xfce-d1-preview.img.tar.zst，连接如下：https://mirror.iscas.ac.cn/openeuler-sig-riscv/openEuler-RISC-V/preview/openEuler-22.03-V2-riscv64/D1/openEuler-22.03-V2-xfce-d1-preview.img.tar.zst

其他文件均无需下载。

```bash
wget https://mirror.iscas.ac.cn/openeuler-sig-riscv/openEuler-RISC-V/preview/openEuler-22.03-V2-riscv64/D1/openEuler-22.03-V2-xfce-d1-preview.img.tar.zst
```

### 3. 刷写镜像

以下步骤适用于Ubuntu20.04，如适用Windows操作系统，解压请下载[zstd](./zstd-v1.4.4-win32.zip)`zstd.exe -d ./openEuler-22.03-V2-xfce-d1-preview.img.tar.zst`，之后将.img.tar解压为.img文件，刷写请下载[win32diskimager](./win32diskimager-1.0.0-install.exe)

1. 解压镜像文件

```bash
sudo apt install zstd -y
tar -I zstdmt -xvf ./openEuler-22.03-V2-xfce-d1-preview.img.tar.zst
```

2. 镜像刷写

将64G micro-sd卡装入读卡器后，插入笔记本电脑。笔记本电脑通常带一个硬盘，所以sd卡对应设备是/dev/sdb

```bash
sudo dd if=./openEuler-22.03-V2-xfce-d1-preview.img of=/dev/sdb bs=1M iflag=fullblock oflag=direct conv=fsync status=progress
```

### 4. 安装串口调试软件

1）将Usb转uart串口通信模块连接到电脑usb口。

2）检查设备管理器中的COM端口，例如COM4。

![figure_2](./images/figure_2.png)

3）使用Xmodem安装固件。

安装teraterm，https://mobaxterm.mobatek.net/download.html

    选择菜单setup->Serial port setup
    Speed设置为115200
    Data设置为8bit
    Paritv设置为none
    Stoo bits设置为1bit
    Flowcontrol设置为none

![figure_4](./images/figure_4.png)

### 5. 启动D1

将64G micro-sd卡装入sd卡槽，连接tpye-c电源启动，用户名：`root`，密码：`openEuler12#$`， 用户名：`openeuler`，密码：`openEuler12#$`


**无线网络配置**

```
nmtui
```
