# s905d-phicomm-n1

Armbian-5.77 on N1, so far so good

N1 如何完美刷入 armbian 系统

https://www.right.com.cn/forum/thread-510423-1-1.html
(出处: 恩山无线论坛)

将 meson-gxl-s905d-phicomm-n1-xiangsm.dtb 文件放到 dtb / 文件夹 下面，修改一下 uEnv.ini 文件指向它：

dtb_name=/dtb/meson-gxl-s905d-phicomm-n1-xiangsm.dtb

ethaddr=fc:7c:02:ea:75:4d #固定 mac 地址用


https://gitlab.com/networkbase/s905d-phincomm-n1/-/raw/main/meson-gxl-s905d-phicomm-n1-xiangsm.dtb?inline=false

千万不要在安卓系统开机的情况下插入 U 盘，否则 U 盘中的文件权限会被安卓系统篡改！这不是一句废话！

以下是安装armbian 5.77到n1的具体步骤：
==========================================================

1. 从 https://yadi.sk/d/srrtn6kpnsKz2/Linux/ARMBIAN/5.77/S905 下载由@150balbes编译好的镜像，我选的是debian/desktop版，您请随意。

2. 解压镜像并写入U盘，以linux系统为例：
       $ xzcat --keep Armbian_5.77_Aml-s905_Debian_stretch_default_5.0.2_desktop_20190318.img.xz | sudo dd of=/dev/sdX bs=1M && sync

3. 将写好armbian的U盘插入关机状态的n1，通电启动，armbian就运行起来了。初次运行时会提示修改root密码和创建一个常规用户。

4. 此时，armbian用的是kdahas-vim开发板的dtb，所以不完全适配n1，一些设备不工作，这是正常的，我们只需修改/boot/uEnv.ini指向n1的dtb即可。
       - 修改前的uEnv.ini：      dtb_name=/dtb/meson-gxl-s905x-khadas-vim.dtb
       - 修改后的uEnv.ini：      dtb_name=/dtb/meson-gxl-s905d-phicomm-n1.dtb

5. 修改完成后重启系统，重启后所有设备(lan/wifi/bluetooth/etc.)全部能工作！除了系统负载有点高。
       $ sudo reboot

6. 下面解决系统负载问题，关键：修改随镜像文件自带的dtb文件中的一项与中断处理有关的设置：
       # 反编译原始n1 dtb文件为n1.dts
       $ dtc -I dtb -O dts -o n1.dts /boot/meson-gxl-s905d-phicomm-n1.dtb

       # 用vi 或 nano打开n1.dts，将第183行注释掉。修改前：phandle = <0x1e>;  修改后： #phandle = <0x1e>;
       $ vi n1.dts
       ...

       # 编译新的n1.dtb
       $ dtc -I dts -O dtb -o n1.dtb n1.dts

       # 复制n1.dtb到/boot/dtb中并修改相应uEnv.ini文件
       $ sudo cp -av n1.dtb /boot/dtb/meson-gxl-s905d-phicomm-n1-xiangsm.dtb
       $ sudo sed -i -e 's/-n1/-n1-xiangsm/' /boot/uEnv.ini   # 或用vi/nano可视化编辑

       # 好了，可以重启系统了，重启后，系统负载终于正常，并且各项硬件应该依然都能正常工作。
       $  sudo reboot

6a. 对于觉得怕修改和编译dtb麻烦的，附件里提供了已经修改过的dtb，md5:82a5d7。操作方法：
       下载附件 --> 解开dtb文件 --> 复制到armbian的/boot/dtb/ 目录 --> 相应修改/boot/uEnv.ini中的dtb文件路径设置
7. 刷写 U 盘中的 armbian 到 emmc;  nand–sata-install  ; halt ;
       关机拔掉 U 盘，所有刷机工作就结束
       如果你想重刷 armbian 系统，那么还是 U 盘启动 armbian，重新执行 nand-sata-install 即可完整覆盖掉 emmc 中的数据
       

==========================================================
[注1] 在n1运行android时，务！必！不！要！将armbian u盘插入n1，否则armbian u盘的ext4分区内的文件权限和所有者会被android系统篡改，引起各种异常。
[注2] 开始用CZ600 u盘，可能是速度太慢，导致bootloader超时，fallback到emmc启动了，然后就要从头来过，因为U盘里的文件系统已经被污染。后用cz43，正常。
<THE END>

联通外网:
     修改 / etc/network/interfaces，注释 hotplug 热插拔模式，换成 auto 即可：

root@aml:~# cat /etc/network/interfaces

source /etc/network/interfaces.d/*

# Wired adapter #1

auto eth0

#allow-hotplug eth0
  
在路由器上给 N1 分配静态 IP 的时候发现，armbian 的有线网卡每次重启后 MAC 地址都会变，解决方法也不复杂。

打开 / etc/network/interfaces 文件，在 eth0 网卡上写死一个 mac 地址即可：

root@aml:~# cat /etc/network/interfaces

source /etc/network/interfaces.d/* 
# Wired adapter #1  
allow-hotplug eth0  
no-auto-down eth0  
iface eth0 inet dhcp  
hwaddress 4e:03:88:54:39
       
       install docker by armbian-config 
       
       
       需要改一下 / etc/environment 中的 LC_ALL：

root@aml:~# cat /etc/environment
ARCH=arm64
LC_ALL=”en_US.utf-8″

执行 armbian-config，选择 personal，然后配置 Timezone 为 chongqing 即可。

执行 armbian-config，选择 personal，选择 Mirror 中清华大学的那个即可。
       
       让 N1 联无线网。

这样的话，N1 就可以放在任何有电源的地方，当一个小型 linux 服务器了。

执行 armbian-config，选择 network -> wlan0，选择 wifi 信号输入账号密码即可。
       
