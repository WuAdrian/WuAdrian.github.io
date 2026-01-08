### 第一步：环境准备与磁盘分区

进入 Root 模式：
sudo -i

磁盘清理与重新分区：
输入 fdisk /dev/sda。
输入 g (创建一个全新的 GPT 分区表)。
输入 n (新建系统分区) -> 分区号 1 -> 起始位置回车 -> 大小输入 +32G。
输入 n (新建数据分区) -> 分区号 2 -> 起始/结束位置全部回车 (占用剩余空间)。
输入 w 保存退出。

### 第二步：镜像上传与系统写入

准备镜像存放区：

格式化 sda2 方便传文件
mkfs.ext4 /dev/sda2
mkdir -p /mnt/sda2
mount /dev/sda2 /mnt/sda2

开启权限以便从电脑端上传镜像
chmod 777 /mnt/sda2
把img文件传到/mnt/sda2

【重要】写入系统方法( 后续如果升级 就切换回emmc内置系统里面 然后执行下面的方法 就可以直接升级，已实测)
关联镜像文件
losetup -fP /mnt/sda2/你的镜像文件名.img

执行克隆 (loop0p2 是镜像内的系统主分区)
dd if=/dev/loop0p2 of=/dev/sda1 bs=4M status=progress conv=fsync

释放镜像关联
losetup -D


### 第三步：系统扩容与关键配置

扩容 SSD 系统分区：由于镜像只有 3.4G，我们需要让它填满 32G 的 sda1。
mkdir -p /mnt/sda1
mount /dev/sda1 /mnt/sda1
btrfs filesystem resize max /mnt/sda1

修改 SSD 系统挂载表 (fstab)：这一步极其重要，不改会导致系统无法正常进入。
nano /mnt/sda1/etc/fstab

将内容修改为（确保 root 指向 sda1）：
/dev/sda1         /        btrfs    defaults,noatime,errors=remount-ro    0    1
/dev/mmcblk0p1    /boot    ext4     defaults,noatime,errors=remount-ro    0    2
tmpfs     /tmp     tmpfs    defaults,nosuid                       0    0

(Ctrl+O 保存，Enter 确认，Ctrl+X 退出)

### 第四步：切换引导开关

修改 eMMC 中真正的引导文件：
nano /boot/extlinux/extlinux.conf
找到 append 开头的那一行，将其中的 root=/dev/mmcblk0p2 修改为：root=/dev/sda1

### 第五步：收尾与重启Bash

reboot

### 第六步：双系统切换

1. 切换系统
手动修改nano /boot/extlinux/extlinux.conf
改成root=/dev/sda1就是硬盘系统
改成root=/dev/mmcblk0p2  就是内置系统

或命令一键修改
sudo sed -i 's|root=/dev/mmcblk0p2|root=/dev/sda1|g' /boot/extlinux/extlinux.conf && sync && echo "已成功切换为 SATA 启动模式"
sudo sed -i 's|root=/dev/sda1|root=/dev/mmcblk0p2|g' /boot/extlinux/extlinux.conf && sync && echo "已成功切换为 eMMC 启动模式"
切换后输入reboot重启即可

2.使用硬盘系统的时候升级内置系统
卸载 eMMC 的系统分区（如果它被识别为 RemovableDisk）
umount /dev/mmcblk0p2
关联你下载好的新镜像，
losetup -fP 你的镜像名.img

将镜像的系统分区 (loopXp2) 写入 eMMC 的系统分区 (mmcblk0p2)
请注意：执行前确认 loop0p2 是 3.4G 左右的那个分区
dd if=/dev/loop0p2 of=/dev/mmcblk0p2 bs=4M status=progress conv=fsync

释放关联
losetup -D
### 【可选】扩容EMMC

不扩容的话内置系统盘显示3.XG

创建临时挂载点并挂载 eMMC 系统区
mkdir -p /mnt/emmc_sys
mount /dev/mmcblk0p2 /mnt/emmc_sys

强制扩容文件系统至硬件上限
btrfs filesystem resize max /mnt/emmc_sys

验证容量是否变回 6.8G+
df -h /mnt/emmc_sys

卸载挂载点
umount /mnt/emmc_sys



3.使用内置eMMC系统的时候升级外置硬盘系统
假设你通过web后台 创建了down文件夹，把img上传到这个文件  你的最新镜像名.img
关联镜像文件
确保 SSD 系统分区已卸载 (防止写入冲突)
 执行克隆 (loop0p2 是镜像中的系统分区)
dd =/dev/loop0p2 of=/dev/sda1 bs=4M status=progress conv=fsync
释放镜像关联
losetup -D
