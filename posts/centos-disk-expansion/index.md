# 记一次CentOS磁盘扩容操作


前期安装的CentOS，专门拿来挂机跑青龙、机器人等项目，当时估算的磁盘分配少了，监控程序一直处于黄色告警，虽然不影响正常使用，但有空还是处理下为好，正常的资源利用率更有利于系统稳定运行。

<!--more-->

## 引言 LVM概述

- 在 Linux 系统中，我们经常使用 LVM （逻辑卷管理）的方式去管理和使用磁盘， LVM 可以动态扩容，给我们的使用带来了很多的便捷性。

- LVM结构图如下：

  {{< image src="https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-10-57-16-20231220105716.png" width="50%" caption="LVM结构图" title="LVM结构图" >}}

  - 物理卷（Physical Volume，PV）
    指磁盘分区或从逻辑上与磁盘分区具有同样功能的设备（如RAID），是LVM的基本存储逻辑块，但和基本的物理存储介质（如分区、磁盘等）比较，却包含有与LVM相关的管理参数。
  - 卷组（Volume Group，VG）
    类似于非LVM系统中的物理磁盘，其由一个或多个物理卷PV组成。可以在卷组上创建一个或多个LV（逻辑卷）。
  - 逻辑卷（Logical Volume，LV）
    类似于非LVM系统中的磁盘分区，逻辑卷建立在卷组VG之上。在逻辑卷LV之上可以建立文件系统（比如/home或者/usr等）。

## CentOS实操

下边主要通过截图记录本次操作，以便回顾关键点。

### 一 ESXi修改磁盘大小

打开ESXi中需要增加的虚拟机配置界面，直接修改硬盘大小（下图红框所示），点击保存，然后重启虚拟机即可。

![ESXi修改原磁盘大小](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-10-31-55-20231220103154.png)

至此，ESXi操作完毕。

### 二 CentOS扩容

使用SSH终端连接CentOS操作后续步骤。

**核心思想：使用增加的磁盘空间创建一个新PV，跟原有的PV接接，整合到同一个LV上去，完成扩容。**

#### 观察系统是否正常识别需要扩容的空间

使用`lsblk`查看磁盘分配情况

![原磁盘状态1](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-00-12-20231220110012.png)

使用`fdisk -l`确认

```sh
fdisk -l |grep /dev/sda
```

![原磁盘状态2](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-02-05-20231220110205.png)

在ESXi增加了磁盘大小重启后，再次使用`lsblk`

![增加磁盘空间后的磁盘状态1](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-03-49-20231220110349.png)

可以看到，`sda`总大小为64G，较原有的20G多了44G。

![增加磁盘空间后的磁盘状态2](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-08-48-20231220110847.png)

#### 对新增加的空间进行分区格式化

使用`fdisk`命令启动格式化

```sh
fdisk /dev/sda
```

![格式化并新建PV](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-16-38-20231220111636.png)

完成上述命令后，可观察终端输出，一般可能出现一些WARNING，说系统繁忙，我们直接重启虚拟机，能正常进入即成功。

#### 新建PV

使用`pvs`查看PV

![查看PV1](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-25-27-20231220112527.png)

创建新PV`pvcreate`

```sh
pvcreate /dev/sda3
```

![新PV](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-27-39-20231220112739.png)

再次`pvs`确认

![查看PV2](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-29-36-20231220112936.png)

#### 将新PV加入到VG

使用`vgs`查看VG

![查看VG1](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-35-21-20231220113521.png)

使用`vgextend`进行增加

```sh
vgextend centos /dev/sda3
```

![加入VG](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-43-51-20231220114351.png)

再次`vgs`确认

![查看VG2](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-42-38-20231220114237.png)

#### 扩容LV

使用`lvs`查看LV

![查看LV1](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-45-55-20231220114555.png)

扩容43G空间到LV

```sh
lvextend -L +43G /dev/centos/root
```

![扩容LV1](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-49-32-20231220114932.png)

当初实际在ESXi加的是44G，但经过分区格式化后是不足44G的，在这里磁盘是有损耗的，一定要注意参数不要直接写44G，我们可以通过再加1G验证

```sh
lvextend -L +1G /dev/centos/root
```

![扩容LV2](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-51-47-20231220115147.png)

这次改用`lvdisplay`可以查看确认更详细的信息

![查看LV2](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-54-30-20231220115429.png)

可以看到`root`下空间增至60G，扩容成功。

#### 刷新文件系统信息

本次CentOS使用的是XFS文件系统，所以使用`xfs_growfs`刷新；若是EXT文件系统就要用`resize2fs`命令；可以通过`lsblk -f`或`cat /etc/fstab`查看文件系统。

```sh
xfs_growfs /dev/centos/root
```

![刷新文件系统](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-57-38-20231220115738.png)

`reboot`后重新进入，通过`df -h`查看系统空间情况，确认扩容成功。

![查看最终成果](https://cdn.jsdelivr.net/gh/qiuzhi/static/blog/2023-12-20-11-59-51-20231220115951.png)

### 四 总结

至此，本次扩容完成，整体较为顺利，多谢大佬[Frank](http://www.qxzx.xyz/)的远程指导操作，让我更深刻学习Linux基础，收益甚多！

## 参考

- [LV扩容(lvextend)](https://www.jianshu.com/p/4c7acf819046)


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/centos-disk-expansion/  

