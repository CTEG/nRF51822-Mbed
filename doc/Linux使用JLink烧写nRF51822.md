linux下的Jlink会提示升级固件(如果JLink的固件版本比PC上的软件版本低的话),  然后自动升级, 然后JLink就不亮灯了, 即固件升级失败. 所以在linux下使用JLink时, 不能升级. 请确保JLinkExe的版本和JLink固件版本一致. 

由于NRF51822EK_PRO本身集成Jlink, 所以直接用USB线连上电脑即可.  在linux下, NRF51822EK_PRO使用不了JLinkgdbserver. 至于为什么, 我也不知道. 咱这里只用来烧写.

输入./JLinkExe 连上JLink, 自动切换的SWD模式.  (如果连不上, 看看JLink的readme文件)

JLink_Linux_V480e_i386$ ./JLinkExe 
SEGGER J-Link Commander V4.80e ('?' for help)
Compiled Jan 31 2014 18:13:30
DLL version V4.80e, compiled Jan 31 2014 18:13:27
Firmware: J-Link ARM-OB STM32 compiled Aug 22 2012 19:52:04
Hardware: V7.00
S/N: xxxxxxxx 
Feature(s): RDI,FlashDL,FlashBP,JFlash,GDBFull 
VTarget = 3.300V
Info: Could not measure total IR len. TDO is constant high.
Info: Could not measure total IR len. TDO is constant high.
No devices found on JTAG chain. Trying to find device on SWD.
Info: Found SWD-DP with ID 0x0BB11477
Info: Found Cortex-M0 r0p0, Little endian.
Info: FPUnit: 4 code (BP) slots and 0 literal slots
Cortex-M0 identified.
Target interface speed: 100 kHz
J-Link>

连接成功!

接下来讲讲烧写NRF51822的地址问题:

根据官方给的文档, 使用不同版本的S110, 设置的链接地址和烧写地址都有所不同.

我用的是S110_5.2.1的版本, 蓝牙协议栈占用80KB Code Rom, s110 softdriver 烧写到0x0地址, 一直到0x14000(80*1024), 所以我们烧写使用S110蓝牙协议栈的程序时, 下载地址是0x14000, 使用JLink命令 loadbin your_bin 0x14000来烧写.如果不使用蓝牙协议栈的应用程序, 直接下载到0x0即可. 这个没实验过, 理论上是这个地址的.

下面贴上我的烧写脚本:

$cat NRF51822_FlashDL.JLinkScript 
r
h
loadbin /tmp/led_gcc_s110_xxaa.bin 0x14000
q

$cat FlashDownload.sh 
#!/bin/bash
JLinkScriptFile=$PWD/NRF51822_FlashDL.JLinkScript
JLinkDir=/opt/software/JLink_Linux_V480e_i386

cd $JLinkDir
./JLinkExe -if SWD -device nRF51822_xxAA -speed 1000 -CommanderScript $JLinkScriptFile

用在其他项目, 稍微修改一下路径和文件名就可以用了.


