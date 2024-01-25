---
title: 零刻 SER6 Pro Vest 7735HS
date: 2023-06-06 02:03:13
tags:
 - PVE
categories:
 - Just-buy
---


一直想弄一个自己用的 PVE 机器，由于特殊的需求，希望可以便携一些，比较心水小主机，之前看中的零刻 SER6 Pro 7735HS 准系统版本 618 可以 1999 - 120 ，可以接受，于是就下单了。

到手观的感确实很迷你，遗憾的是只有一个 RJ45 网口，新出的 7840 版本有两个但是贵了快 1K 块。

下面是安装 PVE 和设置的过程。

抄自：
  - [SER6 Pro Vest 7735HS Proxmox VE 核显直通](https://gist.github.com/wjy20030407/f0b3658d56e8dec043614a6bdebed929) 
  - [PVE下安装Windows10并直通核显...](https://www.simaek.com/archives/69/)

<!--more-->
注： 这个机器网口太深，网线头卡扣正好全在口里，所以如果水晶头比较硬，最好处理一下，用胶带贴个一圈什么的

## 安装 PVE

Proxmox VE: [7.4-1](https://www.proxmox.com/en/downloads/item/proxmox-ve-7-4-iso-installer)

### 修复安装引导 Xorg

**PVE 8.x 可以直接进行图形安装，不需要额外操作**  

PVE 使用图形界面安装，正常安装进行时会报错，`Ctrl + Alt + F2` 可以显示错误信息，需要在 `Ctrl + Alt + F1` 下执行下面的命令

```bash
Xorg -configure
cp /xorg.conf.new /etc/X11/xorg.conf
sed -i 's/amdgpu/fbdev/g' /etc/X11/xorg.conf

xinit
```

即可进入图形安装向导，然后按照步骤安装。

### 删除 local-lvm 合并到 local
Shell 中运行命令
```bash
lvremove pve/data
lvextend -l +100%FREE -r pve/root
```
然后在 web 界面中前往 `数据中心` > `存储`, 移除 `local-lvm` ，再编辑 `local`，内容里选中所有的项目

### 禁用 Enterprise 源
不禁用的话 `apt update` 会遇到错误
```bash
Err:4 https://enterprise.proxmox.com/debian/pve bullseye InRelease                  
  401  Unauthorized [IP: xxx.xxx.xxx.xxx 443]
```

在 web 界面中，选择节点(pve)，右侧选择 `更新` > `存储库`，禁用掉 Enterprise 的存储库，可以添加一个 pve-no-subscription 的存储库用于 PVE 自身的更新 

## 核显直通

### 1. 开启 iommu

编辑文件 `/etc/default/grub`
```
sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT="quiet"/GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt initcall_blacklist=sysfb_init"/g' /etc/default/grub

update-grub
```

### 2. 加载内核模块
```
cat >>/etc/modules <<EOF
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
EOF
```
### 3. 将显卡加入 PVE 的驱动黑名单
```
cat >>/etc/modprobe.d/pve-blacklist.conf <<EOF
blacklist amdgpu
blacklist snd_hda_intel
EOF
```

### 4. 将显卡绑定至 vfio-pci

其中显卡是 1002:1681，声卡是 1002:1640
查找显卡 ID
```
lspci
```
输出如下 
```bash
... 
01:00.0 Non-Volatile memory controller: MAXIO Technology (Hangzhou) Ltd. Device 1602 (rev 01)
02:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8125 2.5GbE Controller (rev 05)
03:00.0 Network controller: Intel Corporation Wi-Fi 6 AX200 (rev 1a)
74:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Device 1681 (rev 0a)
74:00.1 Audio device: Advanced Micro Devices, Inc. [AMD/ATI] Device 1640
74:00.2 Encryption controller: Advanced Micro Devices, Inc. [AMD] VanGogh PSP/CCP
74:00.3 USB controller: Advanced Micro Devices, Inc. [AMD] Device 161d
...
```

VGA 是显卡，Audio 是声卡，然后再运行下面的命令

```
root@pve:~# lspci -n -s 74:00.0
74:00.0 0300: 1002:1681 (rev 0a)

root@pve:~# lspci -n -s 74:00.1
74:00.1 0403: 1002:1640
```
编辑文件 `/etc/modprobe.d/vfio.conf` 添加下面的内容
```bash
options vfio-pci ids=1002:1681,1002:1640
```

### 5. 刷新 initramfs 并重启 Proxmox VE
```
update-initramfs -u -k all

reboot
```

### 6. 获取 vBIOS

```bash 
gcc vbios.c -o vbios
./vbios
mv vbios_1002_1681.bin /usr/share/kvm/
```

{% note default vbios.c %}
```bash vbios.c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

typedef uint32_t ULONG;
typedef uint8_t UCHAR;
typedef uint16_t USHORT;

typedef struct {
    ULONG Signature;
    ULONG TableLength; // Length
    UCHAR Revision;
    UCHAR Checksum;
    UCHAR OemId[6];
    UCHAR OemTableId[8]; // UINT64  OemTableId;
    ULONG OemRevision;
    ULONG CreatorId;
    ULONG CreatorRevision;
} AMD_ACPI_DESCRIPTION_HEADER;

typedef struct {
    AMD_ACPI_DESCRIPTION_HEADER SHeader;
    UCHAR TableUUID[16]; // 0x24
    ULONG VBIOSImageOffset; // 0x34. Offset to the first GOP_VBIOS_CONTENT block from the beginning of the stucture.
    ULONG Lib1ImageOffset; // 0x38. Offset to the first GOP_LIB1_CONTENT block from the beginning of the stucture.
    ULONG Reserved[4]; // 0x3C
} UEFI_ACPI_VFCT;

typedef struct {
    ULONG PCIBus; // 0x4C
    ULONG PCIDevice; // 0x50
    ULONG PCIFunction; // 0x54
    USHORT VendorID; // 0x58
    USHORT DeviceID; // 0x5A
    USHORT SSVID; // 0x5C
    USHORT SSID; // 0x5E
    ULONG Revision; // 0x60
    ULONG ImageLength; // 0x64
} VFCT_IMAGE_HEADER;

typedef struct {
    VFCT_IMAGE_HEADER VbiosHeader;
    UCHAR VbiosContent[1];
} GOP_VBIOS_CONTENT;

int main(int argc, char** argv)
{
    FILE* fp_vfct;
    FILE* fp_vbios;
    UEFI_ACPI_VFCT* pvfct;
    char vbios_name[0x400];

    if (!(fp_vfct = fopen("/sys/firmware/acpi/tables/VFCT", "r"))) {
        perror(argv[0]);
        return -1;
    }

    if (!(pvfct = malloc(sizeof(UEFI_ACPI_VFCT)))) {
        perror(argv[0]);
        return -1;
    }

    if (sizeof(UEFI_ACPI_VFCT) != fread(pvfct, 1, sizeof(UEFI_ACPI_VFCT), fp_vfct)) {
        fprintf(stderr, "%s: failed to read VFCT header!\n", argv[0]);
        return -1;
    }

    ULONG offset = pvfct->VBIOSImageOffset;
    ULONG tbl_size = pvfct->SHeader.TableLength;

    if (!(pvfct = realloc(pvfct, tbl_size))) {
        perror(argv[0]);
        return -1;
    }

    if (tbl_size - sizeof(UEFI_ACPI_VFCT) != fread(pvfct + 1, 1, tbl_size - sizeof(UEFI_ACPI_VFCT), fp_vfct)) {
        fprintf(stderr, "%s: failed to read VFCT body!\n", argv[0]);
        return -1;
    }

    fclose(fp_vfct);

    while (offset < tbl_size) {
        GOP_VBIOS_CONTENT* vbios = (GOP_VBIOS_CONTENT*)((char*)pvfct + offset);
        VFCT_IMAGE_HEADER* vhdr = &vbios->VbiosHeader;

        if (!vhdr->ImageLength)
            break;

        snprintf(vbios_name, sizeof(vbios_name), "vbios_%x_%x.bin", vhdr->VendorID, vhdr->DeviceID);

        if (!(fp_vbios = fopen(vbios_name, "wb"))) {
            perror(argv[0]);
            return -1;
        }

        if (vhdr->ImageLength != fwrite(&vbios->VbiosContent, 1, vhdr->ImageLength, fp_vbios)) {
            fprintf(stderr, "%s: failed to dump vbios %x:%x\n", argv[0], vhdr->VendorID, vhdr->DeviceID);
            return -1;
        }

        fclose(fp_vbios);

        printf("dump vbios %x:%x to %s\n", vhdr->VendorID, vhdr->DeviceID, vbios_name);

        offset += sizeof(VFCT_IMAGE_HEADER);
        offset += vhdr->ImageLength;
    }

    return 0;
}
```
{% endnote %}

### 7. 配置 Windows 10 虚拟机
创建虚拟机时
 - BIOS 选择 SeaBIOS
 - Machine 选择 q35
 - Disks Bus/Device 选择 SCSI 并勾选 丢弃(Discard) 和 SSD 仿真(SSD emulation) (需开启高级选项)
 - Network Model 选择 VirtIO
 - CPU 启用 NUMA，类别选择 `host`

系统安装流程中需要使用 Windows 用的 [KVM 驱动](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/)，下载 ISO 镜像文件挂载为虚拟光驱。

然后进行系统的安装，安装完成后在 Windows 虚拟机内启用远程桌面并关机。

在虚拟机配置页面添加 PCI Device 选择 0000:74:00.0（显卡）并勾选 Primary GPU 和 PCI-Express，再添加一个 0000:74:00.1 (声卡)。将 Display 的 Graphic card 修改为 none。  
以及添加对应的 USB 设备直通你的键盘和鼠标。

编辑 `/etc/pve/qemu-server/.conf`，在显卡后追加 `romfile=vbios_1002_1681.bin`:
```
sed -i 's/0000:74:00.0,pcie=1,x-vga=1/0000:74:00.0,pcie=1,x-vga=1,romfile=vbios_1002_1681.bin/g' /etc/pve/qemu-server/<VM ID>.conf
```

启动虚拟机，通过远程桌面连接后下载 [驱动程序(离线版本)](https://www.amd.com/zh-hant/support/apu/amd-ryzen-processors/amd-ryzen-7-processors-radeon-graphics/amd-ryzen-7-7735hs) 并安装（安装时选择 Driver Only）。

后续虚拟机每次关机之前须手动弹出显卡及声卡，否则会导致 Proxmox VE 强制重启！

