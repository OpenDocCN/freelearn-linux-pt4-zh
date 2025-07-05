# 第四章. Google Docs 气象站

在本章中，我们将查看一个简单的气象站，它也可以作为一个物联网设备使用。这次，我们的 BeagleBone Black 将收集环境数据，并将其发送到远程数据库，以便重新处理并展示到共享环境中。

本地和远程数据都可以在我们喜欢的浏览器中查看，因为这是一个本地系统，我们将使用 `wfrog` 工具，作为远程系统，我们将使用 Google Docs 表格。

# 功能基础

在这个项目中，我们的 BeagleBone Black 将通过两个传感器收集天气数据。但这一次，我们将使用现成的气象站软件，而不是编写专门的软件，在 BeagleBone Black 板上完成工作。远程端，我们将使用著名的 Google Docs 云系统来存储数据，并将其展示给用户。

通过这种方式，我们可以用较少的努力实现一个（*准*）专业的结果！

在这种情况下，我们的工作是连接传感器，将气象站软件调整到我们的硬件上，以便读取传感器数据，然后添加适当的代码，将数据发送到 Google Docs 表格。

# 硬件设置

这次，硬件设置并不复杂，因为我们只需要两个 I²C 芯片就可以获取气象站的基本环境数据，而所有的复杂性都在软件设置中，因为我们需要至少 3.13 版本的内核来管理传感器，并且需要完整的软件工具链来与 Google Docs 系统进行通信！

也许这对你来说不是问题，但我的 BeagleBone Black 运行的是 3.8 版本的内核，缺少一些驱动程序。这就是为什么我决定在一个外部 microSD 上安装基于 3.13 内核版本的新发行版，这样我就不必修改默认的板载 eMMC 设置。

在任何情况下，单纯设置硬件时，我可以使用当前运行的内核，在该内核中我可以通过以下命令启用名为 `I2C1` 的 I²C 总线：

```
root@beaglebone:~# echo BB-I2C1 > /sys/devices/bone_capemgr.9/slots

```

如果一切正常，你应该在板上看到以下内核活动：

```
part_number 'BB-I2C1', version 'N/A'
slot #7: generic override
bone: Using override eeprom data at slot 7
slot #7: 'Override Board Name,00A0,Override Manuf,BB-I2C1'
slot #7: Requesting part number/version based 'BB-I2C1-00A0.dtbo
slot #7: Requesting firmware 'BB-I2C1-00A0.dtbo' for board-name 'Override Board Name', version '00A0'
slot #7: dtbo 'BB-I2C1-00A0.dtbo' loaded; converting to live tree
slot #7: #2 overlays
omap_i2c 4802a000.i2c: bus 2 rev0.11 at 100 kHz
omap_i2c 4802a000.i2c: unable to select pin group
slot #7: Applied #2 overlays.

```

现在应该可以使用新的设备 `/dev/i2c-2`：

```
root@beaglebone:~# ls -l /dev/i2c-2
crw-rw---T 1 root i2c 89, 2 Apr 23 20:23 /dev/i2c-2

```

好的，现在我们可以开始将硬件添加到 BeagleBone Black，并测试与当前内核的连接。

### 注意

读者还可以查看本书作者所写的《*BeagleBone Essentials*》，由 *Packt Publishing* 出版，获取更多关于如何管理 BeagleBone Black 上的 I²C 总线的信息，这些总线是与传感器通信所必需的。

## 设置温湿度传感器

作为温湿度传感器，我决定使用下图所示的设备：

![设置温湿度传感器](img/B00255_04_01.jpg)

### 注意

该设备可以通过以下链接购买（或者通过网络搜索）：[`www.cosino.io/product/humidity-sensor`](http://www.cosino.io/product/humidity-sensor)。

该设备的数据手册可以在 [`dlnmh9ip6v2uc.cloudfront.net/datasheets/BreakoutBoards/HTU21D.pdf`](http://dlnmh9ip6v2uc.cloudfront.net/datasheets/BreakoutBoards/HTU21D.pdf) 找到。

该设备非常简单。I²C 连接如下：

| 引脚 | 温湿度传感器引脚 |
| --- | --- |
| P9.4 - Vcc | + |
| P8.17 - CLK | CL |
| P8.18 - SDA | DA |
| P9.2 - GND | - |

### 提示

如果您想进一步了解 I²C 总线的工作原理，可以从维基百科的文章 [`en.wikipedia.org/wiki/I%C2%B2C`](http://en.wikipedia.org/wiki/I%C2%B2C) 开始阅读。

现在，为了验证连接，我们可以使用 `i2cdetect` 命令，如下所示：

```
root@arm:~# i2cdetect -y -r 2
 0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: UU -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- -- 

```

### 提示

请注意，即使系统中的 I²C 总线被命名为 `I2C1`，它也必须使用 `2` 作为 ID 编号进行寻址！

在 `0x40` 地址处的字符串 `UU`（或 `40`）表示设备已连接！然而，由于某些硬件问题，您可能根本无法获取 `UU` 字符串。在这种情况下，我们可以使用 `i2cget` 命令，如下所示，强制设备进行 I²C 活动：

```
root@beaglebone:~# i2cget -y 2 0x40 0xe7 0x02

```

好的，设备已连接。但如果您收到以下输出，必须重新检查连接：

```
root@beaglebone:~# i2cget -y 2 0x40 0xe7
Error: Read failed

```

### 提示

请注意，您可能需要通过清除传感器上的焊接跳线来禁用板载上拉电阻。实际上，BeagleBone Black 的 I²C 控制器具有 I²C 总线规范所需的内部上拉电阻，在某些情况下，传感器板上的上拉电阻可能会干扰它。

## 设置气压传感器

作为气压传感器，我决定使用下图所示的设备：

![设置气压传感器](img/B00255_04_02.jpg)

### 注意

该设备可以通过以下链接购买（或通过上网搜索）：[`www.cosino.io/product/barometric_sensor`](http://www.cosino.io/product/barometric_sensor)。

该设备的数据手册可以在 [`www.epcos.com/inf/57/ds/T5400.pdf`](http://www.epcos.com/inf/57/ds/T5400.pdf) 找到，另附有有用的应用说明：[`www.epcos.com/inf/57/ds/T5400.pdf`](http://www.epcos.com/inf/57/ds/T5400.pdf)。

该设备有两个接口：I²C 和 SPI。然而，由于之前的设备是 I²C 接口，我决定使用相同的接口。因此，连接必须按照以下表格中的方式进行，其他引脚不连接：

| 引脚 | 气压传感器引脚 |
| --- | --- |
| P9.4 - Vcc | VCC |
| P8.17 - CLK | SCL/SCLK |
| P8.18 - SDA | SDA/MOSI |
| P9.2 - GND | GND |

### 提示

请注意，我们将把两个设备连接到同一 I²C 总线。暂时，您可以断开之前的传感器，然后连接这个传感器。但请记住，在最终配置中，所有传感器设备都将连接到同一总线。

现在，为了验证连接，我们可以使用 `i2cdetect` 命令，如下所示：

```
root@arm:~# i2cdetect -y -r 2
 0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- 77 

```

如前面的命令所示，`0x77` 地址处的字符串 `77`（或 `UU`）表示设备已连接！这时，设备应该能顺利被检测到。所以，如果你没有看到前面的输出，请考虑重新检查硬件连接。

## 最终图片

以下截图展示了我实现该项目并测试软件的原型。如你所见，这次的连接非常复杂。

![最终图片](img/B00255_04_03.jpg)

# 设置软件

现在，是时候全力以赴了！我们必须安装一个带有特定补丁的新内核，以添加所需的驱动程序。然后，我们需要设置我们的 Google 账户，以启用 Google Docs API 来管理云端的电子表格。最后，我们必须安装并正确配置我们选择的天气站软件，用于收集天气数据。

## 安装新内核

要安装新内核，我们必须使用主机 PC，通过以下命令下载源代码：

```
$ git clone git://github.com/RobertCNelson/bb-kernel.git

```

完成后，我们必须进入 `bb-kernel` 目录，然后检出内核版本 3.13：

```
$ git checkout am33x-v3.13

```

现在，我们应该通过从示例文件生成一个名为 `system.sh` 的配置文件来配置编译套件，操作如下：

```
$ cp system.sh.sample system.sh

```

在我的系统上，我使用以下设置修改了新创建的 `system.sh` 文件：

```
CC=/usr/bin/arm-linux-gnueabihf-
MMC=/dev/sdd
```

`MMC` 变量由安装工具（名为 `install_kernel.sh`）使用，并指向安装了 BeagleBone 系统的 microSD 所对应的设备。

### 提示

警告！你必须小心并确保 `MMC` 定义设置正确，否则主机可能会受到损坏。

现在，我们必须将补丁应用到书中的示例代码库中的 `chapter_04/0001-Add-support-for-I2C1-bus-and-the-connected-devices.patch` 文件，以启用名为 `I2C1` 的 I²C 总线以及前述传感器的驱动程序，并为气压传感器添加缺失的驱动程序。命令如下：

```
$ git am --whitespace=nowarn  0001-Add-support-for-I2C1-bus-and-the-connected-devices.patch

```

### 提示

请注意，`--whitespace=nowarn` 命令行选项是必要的，以防你的 `git` 系统配置为自动修复空格错误，而在这种情况下这是错误的。

如果一切正常，以下命令应该显示以下文本：

```
$ git log -1
commit 50949bd3a5c53d915dfdce8f790e3cfdd9ae702a
Author:     Rodolfo Giometti <giometti@hce-engineering.com>
AuthorDate: Wed Jun 24 21:58:50 2015 +0200
Commit:     Rodolfo Giometti <giometti@hce-engineering.com>
CommitDate: Wed Jun 24 22:06:06 2015 +0200

 Add support for I2C1 bus and the connected devices

 Signed-off-by: Rodolfo Giometti <giometti@hce-engineering.com>

```

在开始内核编译之前，让我简单介绍一下这个补丁。它仅添加了以下两个补丁：

```
$ ls patches/bbb-habp/
0001-iio-Add-t5403-barometric-pressure-sensor-driver.patch
0100-arm-am335x-bone-common.dtsi-enable-bus-I2C1-on-pins-.patch

```

第一个补丁是添加气压传感器的驱动程序，第二个补丁是启用标记为 `I2C1` 的 I²C 总线并定义已连接的设备。特别地，第二个补丁完成了以下代码片段中的步骤：

```
diff --git a/arch/arm/boot/dts/am335x-bone-common.dtsi b/arch/arm/boot/dts/am335x-bone-common.dtsi
index 5270d18..ba891ce 100644
--- a/arch/arm/boot/dts/am335x-bone-common.dtsi
+++ b/arch/arm/boot/dts/am335x-bone-common.dtsi
@@ -84,6 +84,13 @@
 >;
 };

+   i2c1_pins: pinmux_i2c1_pins {
+      pinctrl-single,pins = <
+         0x158 (PIN_INPUT_PULLUP | MUX_MODE2)    /* i2c1_sda.i2c1_sda */
+         0x15c (PIN_INPUT_PULLUP | MUX_MODE2)    /* i2c1_scl.i2c1_scl */
+      >;
+   };
+
 i2c2_pins: pinmux_i2c2_pins {
 pinctrl-single,pins = <
 0x178 0x73 /* (SLEWCTRL_SLOW | PIN_INPUT_PULLUP | MUX_MODE3) uart1_ctsn.i2c2_sda */
@@ -295,6 +302,24 @@
 };
 };

+&i2c1 {
+   pinctrl-names = "default";
+   pinctrl-0 = <&i2c1_pins>;
+
+   status = "okay";
+   clock-frequency = <400000>;
+
+   htu21: htu21@40 {
+      compatible = "htu21";
+      reg = <0x40>;
+   };
+
+   t5403: t5403@77 {
+      compatible = "t5403";
+      reg = <0x77>;
+   };
+};

```

首先，前面的代码通过选择合适的 `pinmux` 设置来定义 `i2c1_pins` 组，然后它启用 `I2C1` 总线，设置正确的总线频率，并为附加的传感器设备定义了合适的驱动程序。

然后，补丁添加了启用其应用所需的代码，如下所示：

```
$ git whatchanged -p -1 patch.sh 
...
diff --git a/patch.sh b/patch.sh
index 83787f7..ed3a886 100644
--- a/patch.sh
+++ b/patch.sh
@@ -191,6 +191,12 @@ saucy () {
 ${git} "${DIR}/patches/saucy/0003-saucy-disable-stack-protector.patch"
 }

+bbb_habp () {
+       echo "dir: bbb-habp"
+       ${git} "${DIR}/patches/bbb-habp/0001-iio-Add-t5403-barometric-pressure-s
+       ${git} "${DIR}/patches/bbb-habp/0100-arm-am335x-bone-common.dtsi-enable-
+}
+
 ###
 #arm
 deassert_hard_reset
@@ -211,4 +217,6 @@ boards

 saucy

+bbb_habp
+
 echo "patch.sh ran successful"

```

此外，作为最后一步，它将新添加的驱动程序启用到默认内核的配置中：

```
$ git whatchanged -p -1 patches/defconfig
commit b9b954d37ed2722f7e85e9192d697bb79544ca78
Author:     Rodolfo Giometti <giometti@linux.it>
AuthorDate: Wed Jun 24 21:58:50 2015 +0200
Commit:     Rodolfo Giometti <giometti@linux.it>
CommitDate: Wed Jun 24 22:31:32 2015 +0200

 Add support for I2C1 bus and the connected devices

 Signed-off-by: Rodolfo Giometti <giometti@hce-engineering.com>

diff --git a/patches/defconfig b/patches/defconfig
index 7be0172..464301d 100644
--- a/patches/defconfig
+++ b/patches/defconfig
@@ -4529,6 +4529,7 @@ CONFIG_IIO_SYSFS_TRIGGER=m
 CONFIG_IIO_ST_PRESS=m
 CONFIG_IIO_ST_PRESS_I2C=m
 CONFIG_IIO_ST_PRESS_SPI=m
+CONFIG_T5403=m

 #
 # Temperature sensors

```

好的，所有修改都已说明，现在我们可以使用以下命令开始编译内核：

```
$ ./build_kernel.sh

```

### 提示

这一步以及后续步骤需要较长时间，并且需要耐心，因此您应该拿一杯自己喜欢的茶或咖啡，耐心等待。

一段时间后，程序将显示标准的内核配置面板，我们应该验证所需的驱动程序是否已启用。您应该导航至**设备驱动程序** | **硬件监控支持**，并在**Measurement Specialties HTU21D 湿度/温度传感器**项中选择模块（**<M>**）；在**设备驱动程序** | **工业 I/O 支持** | **压力传感器**中，应该选择**EPCOS T5403 数字气压传感器驱动程序**作为模块。

然后，退出菜单，内核编译将开始。当编译结束时，新的内核映像已准备好，应该会出现以下信息：

```
-----------------------------
Script Complete
eewiki.net: [user@localhost:~$ export kernel_version=3.13.10-bone12]
-----------------------------

```

### 提示

请注意，当执行`build_kernel.sh`文件时，可能会出现以下错误信息：

```
$ ./build_kernel.sh
+ Detected build host [Ubuntu 14.04.3 LTS]
+ host: [x86_64]
+ git HEAD commit: [b00737d02a5b3567169a6c87311fec76a694fea6]
Debian/Ubuntu/Mint: missing dependencies, please install:
-----------------------------
sudo apt-get update
sudo apt-get install device-tree-compiler lzma lzop u-boot-tools libncurses5:i386 libstdc++6:i386
-----------------------------
* Failed dependency check

```

在这种情况下，您可以通过执行前面两个建议的`apt-get`（或`aptitude`）命令来解决问题。

现在，我们可以使用安装工具将其安装到 microSD 上：

```
$ ./tools/install_kernel.sh

```

在更新内核之前，工具会询问用户是否确定选择要安装内核的设备。例如，在我的系统中，我得到以下输出：

```
I see...
fdisk -l:
Disk /dev/sda: 500.1 GB, 500107862016 bytes
...
sdd       8:48   1   3.7G  0 disk
|-sdd1    8:49   1    12M  0 part  /media/giometti/BOOT
`-sdd2    8:50   1   3.7G  0 part  /media/giometti/rootfs
-----------------------------
Are you 100% sure, on selecting [/dev/sdd] (y/n)?

```

我的`MMC`变量设置为`/dev/sdd`；所以，如果我仔细查看相应的行，就可以验证这些是 BeagleBone Black 文件系统的正确名称。这样，我可以放心地输入`y`字符，选择“是”。

### 提示

请注意，microSD 应为*class 10*，并且至少为 4GB 大小。

在命令执行结束时，我们应该得到如下输出：

```
This script has finished...
For verification, always test this media with your end device...

```

现在，只需从主机上取下 microSD，并将其放入 BeagleBone Black 中。按住用户按钮开机，强制从 microSD 启动。如果一切正常，我们可以在正常登录后使用以下命令验证新的内核是否正在运行：

```
# uname -a
Linux arm 3.13.10-bone9 #1 SMP Fri Nov 7 23:25:59 CET 2014 armv7l GNU/ Linux

```

### 注意

读者还可以参考这本书《*BeagleBone Essentials*》，*Packt Publishing*出版，由本书的作者撰写，以获取更多关于如何在外部 microSD 上安装更新内核的信息。

好的，新内核已经准备好！现在我们可以验证所需的驱动程序是否也正确加载：

```
root@arm:~# lsmod | egrep '(t5403|htu21)'
t5403                   3072  0
htu21                   2385  0
industrialio           46516  3 t5403,ti_am335x_adc,kfifo_buf

```

新设备现在可以通过*sysfs*接口进行访问。要获取当前的气压值，可以使用以下命令：

```
root@arm:~# cat /sys/bus/iio/devices/iio\:device1/in_pressure_input
101.926000

```

数据以**千帕**（**kPa**）为单位给出。

可以使用以下命令访问温湿度传感器：

```
root@arm:~# cat /sys/class/hwmon/hwmon0/device/humidity1_input
42988
root@arm:~# cat /sys/class/hwmon/hwmon0/device/temp1_input
27882

```

湿度以相对湿度百分比（m%RH）给出，温度以千分之一摄氏度（m°C）给出，因此我们需要将这两个数值分别除以 1000，以得到相对湿度百分比（%RH）和摄氏度（°C）。

## 运行气象站软件

现在是设置我们的气象站的时候了。为了做到这一点，正如之前所说，我们决定使用一个现成的项目，而不是编写一个新的。这是因为现有很多非常优秀的气象站软件，我们可以使用它们来以更好的方式在本地显示收集到的数据。

这个软件就是 `wfrog` 项目。

### 注意

项目的主页在 [`code.google.com/p/wfrog/`](https://code.google.com/p/wfrog/)。

要安装它，我们可以通过以下命令从在线仓库获取源代码：

```
root@arm:~# svn checkout http://wfrog.googlecode.com/svn/trunk/ wfrog-read-only

```

### 提示

`svn` 命令位于 `subversion` 包中，可以通过以下命令安装：

```
root@arm:~# aptitude install subversion

```

请注意，程序的压缩档案可以在书本示例代码库中的 `chapter_04/wfrog/wfrog-read-only.tgz` 文件中找到。

下载后，我们应该进入新创建的目录 `wfrog-read-only`，并通过以下命令生成 `debian` 包：

```
root@arm:~# cd wfrog-read-only/
root@arm:~/wfrog-read-only# ./debian/rules binary

```

### 提示

请注意，您的系统可能缺少某些必要的软件包，以便能够生成新的 `debian` 包。通过以下命令，您应该可以安装所需的一切，以完成这项工作：

```
root@arm:~# aptitude install debhelper

```

如果一切顺利，`rules` 命令应该显示以下信息：

```
dpkg-deb: building package `wfrog' in `../wfrog_0.8.2-1_all.deb'.

```

然后，为了安装新的软件包，我们可以使用 `gdebi` 命令，它将为我们下载所有包的依赖项，如下所示：

```
root@arm:~/wfrog-read-only# gdebi ../wfrog_0.8.2-1_all.deb
Reading package lists... Done
Building dependency tree 
Reading state information... Done
Building data structures... Done
Building data structures... Done

Requires the installation of the following packages:
libxslt1.1  libyaml-0-2  python-cheetah  python-lxml  python-pygooglechart  python-serial  python-support  python-usb  python-yaml  python2.6  python2.6-minimal
Web-based customizable weather station software
 wfrog is a software for logging weather station data and statistics,
 viewing them graphically on the web and sending them to a remote FTP site.
 The layout and behaviour is fully customizable through an advanced configuration system.
 It is written in python with an extensible architecture allowing new station drivers to be written very easily.
 wfrog supports many weather stations and is compliant with the WESTEP protocol.
 Supported stations:
 * Ambient Weather WS1080
 * Davis VantagePro, VantagePro2
 * Elecsa AstroTouch 6975
 * Fine Offset Electronics WH1080, WH1081, WH1090, WH1091, WH2080, WH2081
 * Freetec PX1117
 * LaCrosse 2300 series
 * Oregon Scientific WMR100N, WMR200, WMRS200, WMR928X
 * PCE FWS20
 * Scientific Sales Pro Touch Screen Weather Station
 * Topcom National Geographic 265NE
 * Watson W8681
Do you want to install the software package? [y/N]:
...
Selecting previously unselected package wfrog.
(Reading database ... 40720 files and directories currently installed.)
Unpacking wfrog (from ../wfrog_0.8.2-1_all.deb) ...
Setting up wfrog (0.8.2-1) …

```

### 提示

可以通过以下命令安装 `gdebi` 命令：

```
root@arm:~# aptitude install gdebi

```

好的，现在软件已经安装完毕，但我们仍然需要完成一些步骤才能继续。第一个步骤是通过一个特殊的模拟器配置系统，以验证网页界面和数据采集系统是否正常工作。为此，我们应按如下方式执行 `wfrog` 命令：

```
root@arm:~# wfrog -S

```

### 提示

请注意，如果在软件第一次运行时遇到如下所示的错误，您可能需要以下补丁来修复问题：

```
Traceback (most recent call last):
 File "/usr/bin/wfrog", line 132, in <module>
 settings = wflogger.setup.SetupClient().setup_settings(SETTINGS_DEF, settings, settings_file)
 File "/usr/lib/wfrog/wflogger/setup.py", line 40, in setup_settings
 if source == None:
UnboundLocalError: local variable 'source' referenced before assignment

```

补丁如下：

```
root@arm:~/wfrog-read-only# svn diff
Index: wflogger/setup.py
===================================================================
--- wflogger/setup.py   (revision 973)
+++ wflogger/setup.py   (working copy)
@@ -35,6 +35,7 @@
 self.logger.debug('Current settings file: '+str(source_file))
 self.logger.debug('New settings file:'+target_file)
 defs = yaml.load( file(settings_def_file, 'r') )
+        source = None
 if source_file is not None:
 source = yaml.load( file(source_file, 'r') )
 if source == None:

```

然后，您只需要重新构建软件包。

上述补丁可以在书本示例代码库中的 `chapter_04/wfrog/0001-fix-setup.diff` 文件中找到。

现在，您应该仔细回答所有问题，并在系统要求您输入气象站模型的驱动程序时选择 `1) random-simulator` 选项，如下所示：

```
Please enter the driver for your station model:
 1) random-simulator - Station Simulator
 2) vantagepro2 - Davis VantagePro
 3) wh1080 - Fine Offset WH1080 and compatibles
 4) wh3080 - Fine Offset WH3080 and compatibles
 5) wmr200 - Oregon Scientific WMR200
 6) wmr928nx - Oregon Scientific WMR928NX
 7) wmrs200 - Oregon Scientific WMRS200
 8) ws2300 - LaCrosse WS2300
 9) ws28xx - LaCrosse WS28xx
> 1

```

配置完成后，您可以通过执行以下两个命令启动气象站系统：

```
root@beaglebone:~# /etc/init.d/wflogger start
root@beaglebone:~# /etc/init.d/wfrender start

```

然后，网页界面可以通过 BeagleBone Black 的 IP 地址（通常是 `192.168.7.2`）的 `7680` 端口访问，如下所示的截图所示：

![运行气象站软件](img/B00255_04_04.jpg)

好的，它已经工作了；但是系统现在使用的是模拟器，我们希望它使用我们刚刚安装的气象传感器的数据！因此，我们必须向新的气象站添加一个新驱动程序。为此，我们必须修改刚下载的源代码。

在`wfrog-read-only/wfdriver/station/`目录下，我们需要添加一个名为`bbb_habp.py`的新文件。以下显示了它的代码的第一部分，其中定义了一个与我们新站点相关的新类：

```
import time
import logging
from wfcommon import units

class BBBhabpStation(object):

    '''
    Station driver for BeagleBone Black Home Automation Blueprints.

    [Properties]

    period [numeric] (optional):
        Polling interval in seconds. Defaults to 60\.    
    '''

    period=60

    logger = logging.getLogger('station.bbb_habp')

    name = 'BeagleBone Home Automation Blueprints weather station'
```

然后，代码定义了读取环境数据的函数：`get_press()`函数读取压力，`get_temp()`函数读取温度，`get_hum()`函数读取湿度，如下所示：

```
   def get_press(self):
    f = open("/sys/bus/iio/devices/iio:device1/in_pressure_input", "r")
    v = f.read()
    f.close()

        return float(v) * 10.0

   def get_temp(self):
    f = open("/sys/class/hwmon/hwmon0/device/temp1_input", "r")
    v = f.read()
    f.close()

        return int(v) / 1000.0

   def get_hum(self):
    f = open("/sys/class/hwmon/hwmon0/device/humidity1_input", "r")
    v = f.read()
    f.close()

        return int(v) / 1000.0
```

完成此操作后，代码定义了核心功能，该功能通过调用`generate_event()`函数并传入适当的参数来生成所有天气事件。刚生成的事件存储在`e`变量中，我们只需填写其字段并通过`send_event()`函数将数据发送到气象站，如下所示的代码片段：

```
    def run(self, generate_event, send_event, context={}):
        while True:
            try:
                e = generate_event('press')
                e.value = self.get_press()
                send_event(e)
                self.logger.debug("press=%fhPa" % e.value)

            except Exception, e:
                self.logger.error(e)

            try:
                e = generate_event('temp')
                e.sensor = 0
                e.value = self.get_temp()
                send_event(e)
                self.logger.debug("temp=%fC" % e.value)

            except Exception, e:
                self.logger.error(e)

            try:
                e = generate_event('hum')
                e.sensor = 0
                e.value = self.get_hum()
                send_event(e)
                self.logger.debug("hum=%f%%RH" % e.value)

            except Exception, e:
                self.logger.error(e)

            try:
                e = generate_event('temp')
                e.sensor = 1
                e.value = self.get_temp()
                send_event(e)
                self.logger.debug("temp=%fC" % e.value)

            except Exception, e:
                self.logger.error(e)

            try:
                e = generate_event('hum')
                e.sensor = 1
                e.value = self.get_hum()
                send_event(e)
                self.logger.debug("hum=%f%%RH" % e.value)

            except Exception, e:
                self.logger.error(e)
```

最后一行用于安排下一个周期：

```
            # pause until next update time
            next_update = self.period - (time.time() % self.period)
            time.sleep(next_update)                
```

### 注意

前面的代码在书中示例代码库的`chapter_04/wfrog/bbb_habp.py`文件中报告。

现在，为了完成任务，我们需要按照以下方式修补`wfrog-read-only/wfdriver/station/__init__.py`文件：

```
root@arm:~/wfrog-read-only# svn diff wfdriver/station/__init__.py
Index: wfdriver/station/__init__.py
===================================================================
--- wfdriver/station/__init__.py   (revision 973)
+++ wfdriver/station/__init__.py   (working copy)
@@ -19,6 +19,7 @@
 import yaml

 import simulator
+import bbb_habp
 import wmrs200
 import wmr928nx
 import wmr200
@@ -66,6 +67,10 @@
 yaml_tag = u'!ws28xx'
 auto.stations.append(ws28xx)

+class YamlWS28xxStation(bbb_habp.BBBhabpStation, yaml.YAMLObject):
+    yaml_tag = u'!bbb_habp'
+auto.stations.append(bbb_habp)
+
 class YamlRandomSimulator(simulator.RandomSimulator, yaml.YAMLObject):
 yaml_tag = u'!random-simulator'
 auto.stations.append(simulator)

```

通过这种方式，我们告诉`wfrog`系统已经添加了一个新站点。

### 注意

补丁保存在书中示例代码库的`chapter_04/wfrog/0002-add-bbb_habp-station.diff`文件中。

在进行所有前面的修改后，我们必须按照刚才展示的方式重新生成软件包，并通过重新运行配置程序并选择新的驱动程序来重新配置它，具体如下：

```
root@arm:~# wfrog -S
This is the setup of wfrog 0.8.2-svn user settings that will be written in /etc/wfrog/settings.yaml

Please enter the driver for your station model:
 1) bbb_habp - BeagleBone Home Automation Blueprints weather station
 2) random-simulator - Station Simulator
 3) vantagepro2 - Davis VantagePro
 4) wh1080 - Fine Offset WH1080 and compatibles
 5) wh3080 - Fine Offset WH3080 and compatibles
 6) wmr200 - Oregon Scientific WMR200
 7) wmr928nx - Oregon Scientific WMR928NX
 8) wmrs200 - Oregon Scientific WMRS200
 9) ws2300 - LaCrosse WS2300
[random-simulator] > 1

```

### 提示

让我提醒你，为了重新生成软件包，你必须执行以下两个命令：

```
root@arm:~/wfrog-read-only# ./debian/rules binary
root@arm:~/wfrog-read-only# gdebi ../wfrog_0.8.2-1_all.deb

```

请注意，这一次，添加了一个名为`bbb_habp`的新条目，因此只需选择它并根据需要重新配置系统。

当所有修改完成后，我们必须停止正在运行的`wfrog`任务：

```
root@arm:~# /etc/init.d/wflogger stop
[ ok ] Stopping wfrog logger - Weather Station Software : wfrog.
root@arm:~# /etc/init.d/wfrender stop
[ ok ] Stopping wfrog renderer - Weather Station Software : wfrender.

```

然后，我们可以安全地使用以下命令清除`wfrog`保存天气数据的文件：

```
root@arm:~# rm /var/lib/wfrog/wfrog-current.xml /var/lib/wfrog/wfrog.csv

```

然后，我们可以按照以下方式重新启动`wfrog`任务：

```
[....] Starting wfrog logger - Weather Station Software : wfrogStarting /usr/lib/wfrog/bin/wfrog...
Detaching to start /usr/lib/wfrog/bin/wfrog...done.
. ok
root@arm:~/chapter_04# /etc/init.d/wfrender start
[ ok ] Starting wfrog renderer - Weather Station Software : wfrender.

```

现在，如果你控制`/var/lib/wfrog`目录中的文件，你应该会看到它们将会被传感器的新数据重新填充。

## 添加 Google Docs API

我们的气象站现在已经完全正常工作，但正如本章一开始所述，我们还希望更进一步——我们希望一个能够通过网络将数据保存到云系统的气象站。为了做到这一点，我们决定使用 Google Docs 表格。

这个想法是获取一个包含当前天气数据和保存的历史数据的工作表，所有数据都会定期更新。让我们看看如何做到这一点。

执行此任务的 API 由`gspread`工具实现，我们可以通过以下命令将其安装到我们的 BeagleBone Black 上：

```
root@arm:~# aptitude install python-pip python2.7-dev libffi-dev
root@arm:~# pip install --upgrade cffi cryptography PyOpenSSL oauth2client gspread

```

### 注意

项目的主页地址是 [`github.com/burnash/gspread`](https://github.com/burnash/gspread)。

在安装所有前述包之后，我们需要准备一个 Google 账户。在这个项目中，我使用了自己的账户，所以我只是访问了我的 Google Docs 页面。然后，我创建了一个名为`bbb_weather`的新电子表格。

### 注意

请参阅 Google Docs 文档 [`support.google.com/docs`](https://support.google.com/docs)，了解有关 Google Docs 使用的更多信息。

一旦创建，我们必须以一种可以从远程计算机访问的方式发布电子表格。为此，我们必须按照[`gspread.readthedocs.org/en/latest/oauth2.html`](http://gspread.readthedocs.org/en/latest/oauth2.html)中的说明进行操作，那里解释了 OAuth2 授权系统。以下是直接来自该页面的所需步骤的小列表：

1.  转到 Google 开发者控制台 ([`console.developers.google.com/project`](https://console.developers.google.com/project)) 并创建一个新项目（或选择您已有的项目）。

1.  在 API 中的**API & auth**下，启用**Drive API**。

1.  转到**凭证**，然后点击**创建新的客户端 ID**。

1.  选择**服务账户**。点击**创建客户端 ID**将生成一对新的公钥和私钥。您将自动下载一个 JSON 文件，里面包含以下数据：

    ```
    {
        "private_key_id": "2cd ... ba4",
        "private_key": "-----BEGIN PRIVATE KEY-----\nNrDyLw ...
                           jINQh/9\n-----END PRIVATE KEY-----\n",
        "client_email": "473 ... hd@developer.gserviceaccount.com",
        "client_id": "473 ... hd.apps.googleusercontent.com",
        "type": "service_account"
    }
    ```

1.  转到 Google Sheets，并与您在`json_key['client_email']`中拥有的电子邮件共享电子表格。否则，当您尝试打开它时，您将遇到`SpreadsheetNotFound`异常。

    ### 注意

    在接下来的代码示例中，我的 Google 凭据存储在`Project-9a372e9e20e6.json`文件中，出于安全原因，该文件没有在书籍的示例代码库中提供。

现在，为了测试一切是否正确设置，您可以使用以下命令在新创建的电子表格中创建一个空白工作表：

```
root@arm:~# ./create_new.py

```

### 注意

您可以在书籍示例代码库中的`chapter_04/wfrog/create_new.py`文件中找到前面的文件。

如果没有错误，您应该能找到一个名为`BBB weather`的新工作表，如下图所示：

![添加 Google Docs API](img/B00255_04_05.jpg)

### 提示

请注意，在执行前述命令时，您可能会遇到以下错误：

```
ImportError: No module named httplib2

```

在这种情况下，您可以通过以下命令解决安装缺失的`python-httplib2`包的问题：

```
root@arm:~# aptitude install python-httplib2

```

或者，您可能会遇到以下错误：

```
oauth2client.client.CryptoUnavailableError: No crypto library available

```

在这种情况下，解决方法是使用以下命令安装缺失的`python-crypto`包：

```
root@arm:~# aptitude install python-crypto

```

`create_new.py`文件中的代码非常简单，如下所示：

```
import gspread
import json
from oauth2client.client import SignedJwtAssertionCredentials

# Load the credentials
json_key = json.load(open('Project-9a372e9e20e6.json'))
scope = ['https://spreadsheets.google.com/feeds']
credentials = SignedJwtAssertionCredentials(json_key['client_email'], json_key['private_key'], scope)

# Ask for authorization
gc = gspread.authorize(credentials)

# Open the "bbb_weather" spreadsheet
sh = gc.open("bbb_weather")

# Add a new worksheet named "BBB weather" with size of 7x4 cells
wks = sh.add_worksheet(title="BBB weather", rows="7", cols="4")

# Setup the "current status" part
wks.update_acell('A1', 'Current status')

wks.update_acell('A2', 'Time (D h)')
wks.update_acell('B2', 'Pressure (hPa)')
wks.update_acell('C2', 'Temperature (C)')
wks.update_acell('D2', 'Humidity (%)')

# Setup the "old statuses" part
wks.update_acell('A5', 'Old statuses')

wks.update_acell('A6', 'Time (D h)')
wks.update_acell('B6', 'Pressure (hPa)')
wks.update_acell('C6', 'Temperature (C)')
wks.update_acell('D6', 'Humidity (%)')

wks.update_acell('A7', 'LAST')
```

首先，您需要注意，必须向 `json.load()` 函数提供您自己的凭证（即刚刚获得的 `Project-xxxxxxxxxxxx.json` 文件）。然后，注意在打开 `bbb_weather` 电子表格后，我们通过使用 `gc.open()` 函数简单地添加了一个名为 `sh.add_worksheet()` 的新工作表。接着，我们使用 `wks.update_acell()` 方法设置单元格内容。

现在，读者可能会好奇为什么 A7 单元格包含 `LAST` 字符串。耐心点，这将在稍后解释！

好的，现在我们需要一种方法将收集到的数据发送到我们的新工作表。如前所述，`wfrog` 程序将其数据存储在两个文件中：`wfrog-current.xml` 存储当前值，`wfrog.csv` 存储历史数据。为此，我们可以使用存储在书籍示例代码库中的 `chapter_04/send_data.py` 文件中的代码。以下是一些相关的代码片段。该文件的第一部分与 `create_new.py` 命令相同，因此我们可以跳过开头部分，直到打开 `bbb_weather` 电子表格为止：

```
# Open the "bbb_weather" spreadsheet
sh = gc.open("bbb_weather")

# Select the worksheet named "BBB weather"
wks = sh.worksheet("BBB weather")
Then we can parse the XML file and extract the current status data to be send over the network:
#
# Send data to Google Docs
#

# Parse the XML file holding the current weather status
xmldoc = minidom.parse('/var/lib/wfrog/wfrog-current.xml')

# Extract the data
time_obj = xmldoc.getElementsByTagName('time')
time = time_obj[0].firstChild.nodeValue
press_obj = xmldoc.getElementsByTagName('pressure')
press = float(press_obj[0].firstChild.nodeValue)
temp_obj = xmldoc.getElementsByTagName('temp')
temp = float(temp_obj[0].firstChild.nodeValue)
hum_obj = xmldoc.getElementsByTagName('humidity')
hum = float(hum_obj[0].firstChild.nodeValue)
print "current: %s press=%f temp=%f hum=%f" % (time, press, temp, hum)
```

一旦提取，这些数据可以发送到相应的单元格：

```
# Update the current status
wks.update_acell('A3', time)
wks.update_acell('B3', press)
wks.update_acell('C3', temp)
wks.update_acell('D3', hum)
```

现在是保存历史数据的时候了。这时，`LAST` 字符串帮助我们解决了问题！首先，我们需要解析 `.csv` 文件：

```
# Parse the CSV file holding the old weather statuses
csvfile = open('/var/lib/wfrog/wfrog.csv', 'rb')
reader = csv.reader(csvfile, delimiter=',')

# Skip the headers
headers = reader.next()
```

然后，我们使用 `wks.find("LAST").row` 方法请求包含 `LAST` 字符串的行号。由于 `LAST` 字符串最初位于第 7 行，我们可以找到在 `wfrog.csv` 文件中跳过多少行，从而找到要存储的新数据，如下所示：

```
# Find the "LAST" string where to insert data to
last = wks.find("LAST").row - 7
print "last saved row was %d" % last

# Skip already read row
for i in range(0, last):
   dummy = reader.next()
```

现在，我们可以通过使用 `wks.insert_row(data, n)` 函数将数据提取并保存到工作表中，将它们存储在 *第 n 行*：

```
# Start saving not yet saved data
for row in reader:
   time = row[1]
   press = float(row[11])
   temp = float(row[2])
   hum = float(row[3])
   print "old: %s press=%f temp=%f hum=%f" % (time, press, temp, hum)

   # Add a new line with an old status
   wks.insert_row([time, press, temp, hum], 7 + last)
   last += 1
```

要测试代码，我们可以执行以下 `send_data.py` 命令：

```
root@arm:~# ./send_data.py
current: 2015-06-27 13:56:00 press=1026.354367 temp=29.083000 hum=44.537000
last saved row was 0
old: 2015-06-27 12:24:32 press=1026.700000 temp=29.200000 hum=49.600000
old: 2015-06-27 12:35:00 press=1026.700000 temp=29.500000 hum=50.100000
old: 2015-06-27 12:45:00 press=1026.600000 temp=29.500000 hum=48.800000
old: 2015-06-27 12:55:00 press=1026.700000 temp=29.400000 hum=48.400000
old: 2015-06-27 13:05:00 press=1026.700000 temp=29.300000 hum=47.500000
old: 2015-06-27 13:15:00 press=1026.600000 temp=29.200000 hum=48.100000
old: 2015-06-27 13:25:00 press=1026.500000 temp=29.100000 hum=45.900000
old: 2015-06-27 13:35:00 press=1026.500000 temp=28.700000 hum=47.100000

```

### 提示

请注意，在前面的文件中，您必须根据您的 JSON 文件名修改以下行！

```
json_key = json.load(open('Project-9a372e9e20e6.json'))
```

程序正确地检测到之前没有保存历史数据，并从头开始保存新数据。我的工作表现在看起来像下面的截图所示：

![添加 Google Docs API](img/B00255_04_06.jpg)

现在，`LAST` 字符串指向第 15 行，因此如果我们等待新数据并重新运行命令，我们会得到如下结果：

```
root@arm:~# ./send_data.py
current: 2015-06-27 13:51:00 press=1026.334273 temp=29.276000 hum=46.871000
last saved row was 8
old: 2015-06-27 13:46:00 press=1026.500000 temp=28.900000 hum=48.200000

```

如下截图所示，我们的程序已经将新数据保存在了正确的位置：

![添加 Google Docs API](img/B00255_04_07.jpg)

为了自动化这些步骤，我们可以使用 `cron` 守护进程，按照预定的延迟安排执行 `send_data.py` 程序。

# 最终测试

这次，最终测试比其他项目需要更多的时间，因为我们必须收集多项数据才能得到合适的图形。因此，我们按照前面的截图所示执行 `wfrog` 任务。然后，我们让它们运行两天或更长时间。对于我的测试，结果如下两个截图所示：

![最终测试](img/B00255_04_08.jpg)![最终测试](img/B00255_04_09.jpg)

然后，我们可以通过查看我们的 Google 账户来验证前面的天气数据是否已存储在 Google Docs 表格中。以下截图显示了我的测试结果：

![最终测试](img/B00255_04_10.jpg)

# 概要

在本章中，我们了解了如何通过将现成的天气站软件安装到我们的 BeagleBone Black 上，适配硬件传感器，并安装带有适当驱动程序的新内核版本。接着，我们查看了如何将数据存储到 Google Docs 表格中，以便日后处理。

在下一章，我们将继续与远程系统进行交互，以便管理洗衣监控系统。我们将使用 WhatsApp 系统来了解洗衣机何时完成工作。
