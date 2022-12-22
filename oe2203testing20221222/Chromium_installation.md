# Chromium 安装指引

## 下载内容

- 下载 QEMU 目录下的 `openEuler-22.03-V1-riscv64-qemu-xfce.qcow2.tar.zst`、`fw_payload_oe_qemuvirt.elf` 和 `preview_start_vm_xfce.sh`
- 下载地址 `https://mirror.iscas.ac.cn/openeuler-sig-riscv/openEuler-RISC-V/preview/openEuler-22.03-V1-riscv64/QEMU/`

## 启动

- 运行脚本

     `bash preview_start_vm_xfce.sh`
     
- 等待图形界面加载完成

## 安装 Chromium

- 使用包管理器直接安装

     `dnf install chromium`
     
- Xfce 桌面下打开终端，输入`chromium` 启动 Chromium

```shell
  Chromium
```

- 点击 Application Finder 图标,搜索 Chromium，点击 Launch

     `
