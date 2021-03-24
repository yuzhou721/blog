---
title: "Manjaro休眠修复"
date: 2021-03-22T23:43:32+08:00
draft: false
---
# 系统休眠修复
新装的monjaro无法休眠（我的swap不是挂的分区，是一个文件，分好以后按POWER键是已经看到有休眠功能了，但是不生效，和按关机效果一样），xfce版本有这个问题，不知道其他版本有没有。

## 查询设备uuid 就是你的swapfile所在目录的挂载分区uuid

    sudo lsblk -f
![uuid](_v_images/20190718091059482_102007118.png)
    
查询结果是afb15c24-49f8-425a-8975-24d28bbbdd48

## swapfile偏移量查询（如果是挂载的SWAP分区，就不需要查询这个）
    
    sudo filefrag -v /swapfile

![偏移量](_v_images/20190718091209382_1953074054.png)
 查询出来是 1306624
## 修改引导文件
    sudo vim /etc/default/grub

找到这行

    GRUB_CMDLINE_LINUX_DEFAULT=  (原来的不变后边的新增)resume=UUID=afb15c24-49f8-425a-8975-24d28bbbdd48 resume_offset=1306624"   

重新生成grub引导
 
     sudo update_grub

uuid指的是前面查询的swap文件或者分区 
如果是swapfile 要加上偏移量(也是filefrag查出来的)

## 修改内核缓存镜像HOOK钩子
    
    vim /etc/mkinitcpio.conf
    #找到这行（原来的不动，新增一项）
    HOOKS="base udev resume(新增的) autodetect modconf block filesystems keyboard fsck"

## 重新生成 initramfs 镜像:

    mkinitcpio -p linux419(linux后面加内核版本号 例如我的是419)

休眠就可以用了

## 相关文档
[archlinux-WIKI Power management/Suspend and hibernate](https://wiki.archlinux.org/index.php/Power_management/Suspend_and_hibernate_(简体中文))

[archlinux-WIKI mkinitcpio](https://wiki.archlinux.org/index.php/Mkinitcpio_(简体中文))

[manjaro-How to fix or enable hibernation on Manjaro 18.0.4?](https://forum.manjaro.org/t/how-to-fix-or-enable-hibernation-on-manjaro-18-0-4/91202)
