# 第三章：软件层

在本章中，我们将介绍以下配方：

+   探索镜像的内容

+   添加新的软件层

+   选择特定的包版本和提供者

+   添加支持的包

+   添加新包

+   添加数据、脚本或配置文件

+   管理用户和组

+   使用 sysvinit 初始化系统

+   使用 systemd 初始化系统

+   安装包安装脚本

+   减小 Linux 内核镜像大小

+   减小根文件系统镜像大小

+   发布软件

+   分析系统合规性

+   使用开源和专有代码

# 介绍

随着硬件特定更改的到来，下一步是定制目标根文件系统；即在 Linux 内核下运行的软件，也称为 Linux 用户空间。

通常的做法是从一个可用的核心镜像开始，然后根据嵌入式项目的需求进行优化和定制。通常选择的起始镜像是 `core-image-minimal` 或 `core-image-sato`，但任何一个都可以。

本章将展示如何添加软件层以包含这些更改，并解释一些常见的定制，如大小优化。还将展示如何向根文件系统添加新包，包括许可考虑事项。

# 探索镜像的内容

我们已经看到如何使用构建历史功能来获取包含在镜像中的包和文件列表。在这个配方中，我们将解释根文件系统是如何构建的，以便能够跟踪其组件。

## 准备工作

当包构建完成后，它们会根据架构在项目的工作目录（`tmp/work`）中进行分类。例如，在 `wandboard-quad` 构建中，我们会看到以下目录：

+   `all-poky-linux`：用于架构无关的包

+   `cortexa9hf-vfp-neon-poky-linux-gnueabi`：用于 cortexa9 硬浮动点包

+   `wandboard_quad-poky-linux-gnueabi`：用于机器特定包；在此情况下，是 `wandboard-quad`

+   `x86_64-linux`：用于构成主机 `sysroot` 的包

BitBake 将在其自己的目录中构建其依赖列表中的所有包。

## 如何做到...

要查找给定包的 `build` 目录，可以执行以下命令：

```
$ bitbake -e <package> | grep ^WORKDIR=

```

在 `build` 目录中，我们可以找到一些子目录（假设未使用 `rm_work`），这些子目录是构建系统在打包任务中使用的。这些子目录包括以下内容：

+   `deploy-rpms`：这是存储最终包的目录。我们可以在这里找到可以本地复制到目标并安装的单个包。这些包被复制到 `tmp/deploy` 目录，在 Yocto 构建根文件系统镜像时也会使用这些包。

+   `image`：这是默认的目标目录，`do_install`任务会将组件安装到该目录。它可以通过配方中的`D`配置变量进行修改。

+   `package`：这个包含了实际的包内容。

+   `package-split`：这里是根据最终的包名将内容分类到各个子目录中。配方可以根据`PACKAGES`变量指定将包内容拆分为多个最终包。除了默认包名称外，默认的包有：

    +   `dbg`：安装用于调试的组件

    +   `dev`：安装用于开发的组件，如头文件和库

    +   `staticdev`：安装用于静态编译的库和头文件

    +   `doc`：这是文档所在的目录

    +   `locale`：安装本地化组件

每个包中要安装的组件由`FILES`变量选择。例如，要添加到默认包，可以执行以下命令：

```
FILES_${PN} += "${bindir}/file.bin"

```

如果要添加到开发包，可以使用以下命令：

```
FILES_${PN}-dev += "${libdir}/lib.so"

```

## 它是如何工作的……

一旦 Yocto 构建系统完成其依赖包列表中的所有单独包的构建，它会运行`do_rootfs`任务，填充`sysroot`并构建根文件系统，然后创建最终的包镜像。你可以通过执行以下命令找到根文件系统的位置：

```
$ bitbake -e core-image-minimal | grep ^IMAGE_ROOTFS=

```

请注意，`IMAGE_ROOTFS`变量不可配置，且不应更改。

该目录中的内容稍后将根据`IMAGE_FSTYPES`配置变量中配置的镜像类型准备成一个镜像。如果某些内容已安装在此目录中，它将被安装到最终镜像中。

# 添加新的软件层

根文件系统的定制涉及向基础镜像中添加或修改内容。与这些内容相关的元数据将进入一个或多个软件层，具体取决于定制的需求量。

一个典型的嵌入式项目通常只有一个包含所有非硬件特定定制的软件层。但也可以为图形框架或系统范围的元素添加额外的层。

## 准备工作

在开始工作之前，最好检查是否有其他人提供了类似的层。此外，如果你试图集成一个开源项目，检查是否已有现成的层。可以在[`layers.openembedded.org/`](http://layers.openembedded.org/)找到可用层的索引。

## 如何操作……

然后，我们可以通过`yocto-layer`命令创建一个新的`meta-custom`层，正如我们在第二章的*创建自定义 BSP 层*配方中所学到的。从`sources`目录中执行以下命令：

```
$ yocto-layer create custom

```

别忘了将该层添加到项目的`conf/bblayers.conf`文件以及模板的`conf`目录中，以便在所有新项目中使用。

默认的`conf/layer.conf`配置文件如下所示：

```
# We have a conf and classes directory, add to BBPATH
BBPATH .= ":${LAYERDIR}"

# We have recipes-* directories, add to BBFILES
BBFILES += "${LAYERDIR}/recipes-*/*/*.bb \
        ${LAYERDIR}/recipes-*/*/*.bbappend"

BBFILE_COLLECTIONS += "custom"
BBFILE_PATTERN_custom = "^${LAYERDIR}/"
BBFILE_PRIORITY_custom = "6"
```

我们在第二章的*创建自定义 BSP 层*配方中讨论了所有相关变量，*BSP 层*。

## 它是如何工作的...

在向新软件层添加内容时，我们需要记住我们的层需要与 Yocto 项目中的其他层良好兼容。因此，在定制配方时，我们将始终使用附加文件，并且仅在完全确定无法通过附加文件添加所需定制时，才会重载现有配方。

为了帮助我们管理多个层中的内容，我们可以使用以下`bitbake-layers`命令行工具：

+   `$ bitbake-layers show-layers`：此命令将显示配置的层，按 BitBake 的视角展示。这有助于检测`conf/bblayer.conf`文件中的错误。

+   `$ bitbake-layers show-recipes`：此命令将显示所有可用的配方及提供它们的层。它可以用来验证 BitBake 是否识别了你新创建的配方。如果没有显示，请验证文件系统层级是否与层的`BBFILES`变量在`conf/layer.conf`中的定义一致。

+   `$ bitbake-layers show-overlayed`：此命令将显示所有被另一个具有相同名称但优先级更高层中的配方所覆盖的配方。它有助于检测配方冲突。

+   `$ bitbake-layers show-appends`：此命令将列出所有可用的附加文件及其应用到的配方文件。它可以用来验证 BitBake 是否识别了你的附加文件。此外，和配方一样，如果它们没有显示，你需要检查文件系统层级和你层中的`BBFILES`变量。

+   `$ bitbake-layers flatten <output_dir>`：此命令将创建一个目录，其中包含所有配置层的内容，且没有覆盖的配方，并应用所有附加文件。这是 BitBake 将看到的元数据。这个扁平化目录有助于发现与层的元数据冲突。

## 还有更多...

我们有时会添加特定于某个开发板或机器的定制内容。这些内容不总是与硬件相关，因此它们可能出现在 BSP 或软件层中。

在这样做时，我们会尽量保持我们的定制尽可能具体。一个典型的例子是针对特定机器或机器系列进行定制。如果你需要为`wandboard-quad`机器添加补丁，可以使用以下代码行：

```
SRC_URI_append_wandboard-quad = " file://mypatch.patch"
```

如果补丁适用于所有基于 i.MX6 的开发板，你可以使用以下命令：

```
SRC_URI_append_mx6 = " file://mypatch.patch"
```

要使用机器系列重载，机器配置文件需要包含一个`SOC_FAMILY`变量，例如在`meta-fsl-arm-extra`中用于`wandboard-quad`的配置。参考以下代码行：

```
conf/machine/wandboard-quad.conf:SOC_FAMILY = "mx6:mx6q:wandboard"
```

为了让它出现在`MACHINEOVERRIDES`变量中，需要包含`soc-family.inc`文件，正如在`meta-fsl-arm`中的配置。以下是`conf/machine/include/imx-base.inc`文件中的相关代码摘录：

```
include conf/machine/include/soc-family.inc
MACHINEOVERRIDES =. "${@['', '${SOC_FAMILY}:']['${SOC_FAMILY}' != '']}"
```

BitBake 将搜索预定义的路径，查找包工作目录内的文件，该目录由`FILESPATH`变量定义，作为以冒号分隔的列表。具体来说：

```
${PN}-${PV}/${DISTRO}
${PN}/${DISTRO}
files/${DISTRO}

${PN}-${PV}/${MACHINE}
${PN}/${MACHINE}
files/${MACHINE}

${PN}-${PV}/${SOC_FAMILY}
${PN}/${SOC_FAMILY}
files/${SOC_FAMILY}

${PN}-${PV}/${TARGET_ARCH}
${PN}/${TARGET_ARCH}
files/${TARGET_ARCH}

${PN}-${PV}/
${PN}/
files/
```

在`wandboard-quad`的具体情况下，这将转换为以下内容：

```
${PN}-${PV}/poky
${PN}/poky
files/poky
${PN}-${PV}/wandboard-quad
${PN}/wandboard-quad
files/wandboard-quad
${PN}-${PV}/wandboard
${PN}/wandboard
files/wandboard
${PN}-${PV}/mx6q
${PN}/mx6q
files/mx6q
${PN}-${PV}/mx6
${PN}/mx6
files/mx6
${PN}-${PV}/armv7a
${PN}/armv7a
files/armv7a
${PN}-${PV}/arm
${PN}/arm
files/arm
${PN}-${PV}/
${PN}/
files/
```

在这里，`PN`是包名，`PV`是包版本。

最好将补丁放在这些层中最具体的地方，比如`wandboard-quad`，然后是`wandboard`、`mx6q`、`mx6`、`armv7a`、`arm`，最后是通用的`PN-PV`、`PN`和`files`。

注意，搜索路径指的是 BitBake 配方的位置，因此在添加内容时，追加文件始终需要添加路径。如果需要，我们的追加文件可以通过以下方式在`FILESEXTRAPATHS`变量中追加或前置，来添加额外的文件夹到该搜索路径：

```
FILESEXTRAPATHS_prepend := "${THISDIR}/folder:"
```

### 注意

注意立即操作符(`:=`)，它会立即扩展`THISDIR`，以及前置操作符，它将你添加的路径放在其他路径之前，这样你的补丁和文件可以首先在搜索中被找到。

此外，我们也看到在配置文件中使用`+=`和`=+`风格的操作符，但应避免在配方文件中使用，应该优先使用追加和前置操作符，如之前的示例代码所解释，以避免顺序问题。

# 选择特定的包版本和提供者

我们的层可以为同一包的不同版本提供配方。例如，`meta-fsl-arm`层包含多个不同类型的 Linux 源：

+   `linux-imx`：这是来自于 Freescale BSP 内核镜像，来源于[`git.freescale.com/git/cgit.cgi/imx/linux-2.6-imx.git/`](http://git.freescale.com/git/cgit.cgi/imx/linux-2.6-imx.git/)

+   `linux-fslc`：这是主线 Linux 内核，来自于[`github.com/Freescale/linux-fslc`](https://github.com/Freescale/linux-fslc)

+   `linux-timesys`：这是一个支持 Vybrid 平台的内核，来自于[`github.com/Timesys/linux-timesys`](https://github.com/Timesys/linux-timesys)

如前所述，所有配方默认提供包名（例如，`linux-imx`或`linux-fslc`），但所有 Linux 配方还必须提供`virtual/kernel`虚拟包。构建系统将根据构建的要求，如目标机器，解析`virtual/kernel`到最合适的 Linux 配方名称。

而在这些配方中，`linux-imx`例如，包含了 2.6.35.3 和 3.10.17 的配方版本。

在这个配方中，我们将展示如何告诉 Yocto 构建系统构建哪个特定的包和版本。

## 如何操作...

要指定我们要构建的精确包，构建系统允许我们指定要使用的提供者和版本。

### 我们如何选择使用哪个提供者？

我们可以通过使用`PREFERRED_PROVIDER`变量来告诉 BitBake 使用哪个配方。为了在我们的 Wandboard 机器上为`virtual/kernel`虚拟包设置首选提供者，我们需要在其机器配置文件中添加以下内容：

```
PREFERRED_PROVIDER_virtual/kernel = "linux-imx"
```

### 我们如何选择使用哪个版本？

在特定的提供者中，我们还可以告诉 BitBake 使用哪个版本，方法是使用 `PREFERRED_VERSION` 变量。例如，要为所有基于 i.MX6 的机器设置特定的 `linux-imx` 版本，我们将在 `conf/local.conf` 文件中添加以下内容：

```
PREFERRED_VERSION_linux-imx_mx6 = "3.10.17"
```

`%` 通配符被接受来匹配任何字符，正如我们在这里看到的：

```
PREFERRED_VERSION_linux-imx_mx6 = "3.10%"
```

然而，通常情况下，这种类型的配置会在机器配置文件中完成，在这种情况下，我们就不会使用 `_mx6` 的附加操作符了。

### 我们如何选择不使用哪个版本？

我们可以使用将 `DEFAULT_PREFERENCE` 变量设置为 `-1` 的方式来指定除非通过 `PREFERRED_VERSION` 变量显式设置，否则不使用某个版本。这在软件包的开发版本中很常见。

```
DEFAULT_PREFERENCE = "-1"
```

# 添加支持的包

通常，我们希望向已经在包含的 Yocto 层中有现成配方的镜像中添加新软件包。

当目标镜像与提供的核心镜像差异较大时，建议定义一个新的镜像，而不是定制现有的镜像。

这个配方将展示如何通过向现有镜像中添加支持的包来定制它，但如果需要，也可以创建一个全新的镜像配方。

## 准备工作

为了发现我们需要的软件包是否包含在我们配置的层中，以及支持哪些特定版本，我们可以使用来自构建目录的 `bitbake-layers`，正如我们之前所见：

```
$ bitbake-layers show-recipes | grep -A 1 htop
htop:
 meta-oe              1.0.3

```

或者，我们也可以像下面这样使用 BitBake：

```
$ bitbake -s | grep htop
htop                                                :1.0.3-r0

```

或者，我们可以在 `sources` 目录中使用 `find` Linux 命令：

```
$ find . -type f -name "htop*.bb"
./meta-openembedded/meta-oe/recipes-support/htop/htop_1.0.3.bb

```

一旦我们知道要包含哪些软件包在最终的镜像中，接下来我们来看看如何将它们添加到镜像中。

## 如何做到这一点...

在开发过程中，我们将使用我们项目的 `conf/local.conf` 文件来添加自定义内容。要向所有镜像中添加软件包，我们可以使用以下代码行：

```
IMAGE_INSTALL_append = " htop"
```

### 注意

请注意，第一个引号后有一个空格，用于将新包与现有包分开，因为附加操作符不会自动添加空格。

我们还可以通过以下方式将添加限制到特定镜像：

```
IMAGE_INSTALL_append_pn-core-image-minimal = " htop"
```

另一种简单的定制方式是通过利用 **特性**。特性是软件包的逻辑分组。例如，我们可以创建一个新的特性叫做 `debug-utils`，它将添加一整套调试工具。我们可以在配置文件或类中按如下方式定义我们的特性：

```
FEATURE_PACKAGES_debug-utils = "strace perf"
```

然后我们可以通过在 `conf/local.conf` 文件中添加 `EXTRA_IMAGE_FEATURES` 变量来将此特性添加到我们的镜像中，如下所示：

```
EXTRA_IMAGE_FEATURES += "debug-utils"
```

如果你将其添加到镜像配方中，你将使用 `IMAGE_FEATURES` 变量。

通常，特性作为 `packagegroup` 配方被添加，而不是单独列出作为软件包。让我们展示如何在 `recipes-core/packagegroups/packagegroup-debug-utils.bb` 文件中定义一个 `packagegroup` 配方：

```
SUMMARY = "Debug applications packagegroup"
LICENSE = "GPLv2"
LIC_FILES_CHKSUM = "file://${COREBASE}/LICENSE;md5=3f40d7994397109285ec7b81fdeb3b58"

inherit packagegroup

RDEPENDS_${PN} = "\
    strace \
    perf \
"
```

然后你将它添加到 `FEATURE_PACKAGES` 变量中，如下所示：

```
FEATURE_PACKAGES_debug-utils = "packagegroup-debug-utils"
```

我们可以使用`packagegroups`来创建更复杂的示例。有关详细信息，请参考*Yocto 项目开发手册*，[`www.yoctoproject.org/docs/1.7.1/dev-manual/dev-manual.html`](http://www.yoctoproject.org/docs/1.7.1/dev-manual/dev-manual.html)。

## 它是如何工作的...

自定义镜像的最佳方法是使用现有镜像作为模板来创建我们自己的镜像。我们可以使用`core-image-minimal.bb`，它包含以下代码：

```
SUMMARY = "A small image just capable of allowing a device to boot."

IMAGE_INSTALL = "packagegroup-core-boot ${ROOTFS_PKGMANAGE_BOOTSTRAP} ${CORE_IMAGE_EXTRA_INSTALL}"

IMAGE_LINGUAS = " "

LICENSE = "MIT"

inherit core-image

IMAGE_ROOTFS_SIZE ?= "8192"
```

然后扩展为我们自己的版本，以允许通过添加以下`meta-custom/recipes-core/images/custom-image.bb`镜像文件来自定义`IMAGE_FEATURES`：

```
require recipes-core/images/core-image-minimal.bb
IMAGE_FEATURES += "ssh-server-dropbear package-management"
```

当然，我们也可以使用现有的镜像作为模板，从头定义一个新的镜像。

## 还有更多...

自定义镜像的最终方法是通过添加 shell 函数，这些函数在镜像创建后执行。你可以通过在镜像配方或`conf/local.conf`文件中添加以下内容来实现：

```
ROOTFS_POSTPROCESS_COMMAND += "function1;...;functionN"
```

你可以在命令中使用`IMAGE_ROOTFS`变量来指定根文件系统的路径。

类别将使用`IMAGE_POSTPROCESS_COMMAND`变量，而不是`ROOTFS_POSTPROCESS_COMMAND`。

一个用例示例可以在`image.bbclass`中的`debug-tweaks`功能中找到，当镜像被调整以允许无密码 root 登录时。这个方法也通常用于自定义目标镜像的 root 密码。

### 配置包

正如我们在第二章的*配置 Linux 内核*一节中看到的那样，*BSP 层*，一些包（比如 Linux 内核）提供了一个配置菜单，可以通过`menuconfig` BitBake 命令进行配置。

另一个值得提到的、带有配置界面的包是 BusyBox。我们将展示如何配置 BusyBox，例如添加`pgrep`，这是一个通过名称查找进程 ID 的工具。具体步骤如下：

1.  配置 BusyBox：

    ```
    $ bitbake -c menuconfig busybox

    ```

1.  在**进程工具**中选择`pgrep`。

1.  编译 BusyBox：

    ```
    $ bitbake -C compile busybox

    ```

1.  将 RPM 包复制到目标：

    ```
    $ bitbake -e busybox | grep ^WORKDIR=
    $ scp ${WORKDIR}/deploy-rpms/cortexa9hf_vfp_neon/busybox- 1.22.1-r32.cortexa9hf_vfp_neon.rpm root@<target_ip>:/tmp

    ```

1.  在目标上安装 RPM 包：

    ```
    # rpm --force -U /tmp/busybox-1.22.1- r32.cortexa9hf_vfp_neon.rpm

    ```

    请注意，由于配置更改后包的版本没有增加，我们强制更新。

# 添加新包

我们已经看到如何自定义我们的镜像，以便我们可以向其中添加支持的包。当我们找不到现有的配方，或者需要集成我们自己开发的新软件时，我们将需要创建一个新的 Yocto 配方。

## 准备就绪

在开始编写新配方之前，我们需要问自己一些问题：

+   源代码存储在哪里？

+   它是源代码控制的，还是以 tarball 形式发布的？

+   源代码许可证是什么？

+   它使用什么构建系统？

+   它需要配置吗？

+   我们能否按原样交叉编译，还是需要打补丁？

+   需要部署到根文件系统的文件有哪些，它们应该放在哪里？

+   是否需要进行任何系统更改，比如新增用户或`init`脚本？

+   是否有任何依赖项需要提前安装到`sysroot`中？

一旦我们知道了这些问题的答案，就可以开始编写我们的食谱了。

## 如何操作...

最好从如下的空白模板开始，而不是从一个类似的食谱开始并修改它，因为这样做的结果会更加干净，并且仅包含严格需要的指令。

添加最小食谱的一个良好起点是：

```
SUMMARY = "The package description for the package management system"

LICENSE = "The package's licenses typically from meta/files/common-licenses/"
LIC_FILES_CHKSUM = "License checksum used to track open license changes"
DEPENDS = "Package list of build time dependencies"

SRC_URI = "Local or remote file or repository to fetch"
SRC_URI[md5sum] = "md5 checksums for all remote fetched files (not for repositories)"
SRC_URI[sha256sum] = "sha256 checksum for all remote fetched files (not for repositories)"

S = "Location of the source in the working directory, by default ${WORKDIR}/${PN}-${PV}."

inherit <class needed for some functionality>

# Task overrides, like do_configure, do_compile and do_install, or nothing.

# Package splitting (if needed).

# Machine selection variables (if needed).
```

## 它是如何工作的...

我们将在接下来的部分中更详细地解释每个食谱部分。

### 包许可证

每个食谱都需要包含`LICENSE`变量。`LICENSE`变量允许你指定多个、替代的以及按包类型的许可证，如下例所示：

+   对于 MIT 或 GPLv2 替代许可证，我们将使用：

    ```
    LICENSE = "GPL-2.0 | MIT"
    ```

+   对于 ISC 和 MIT 许可证，我们将使用：

    ```
    LICENSE = "ISC & MIT"
    ```

+   对于拆分包，所有包都是 GPLv2，除了文档部分，它是由知识共享协议保护的，我们将使用：

    ```
    LICENSE_${PN} = "GPLv2"
    LICENSE_${PN}-dev = "GPLv2"
    LICENSE_${PN}-dbg = "GPLv2"
    LICENSE_${PN}-doc = "CC-BY-2.0"
    ```

开源包通常会在源代码中包含许可证，如`README`、`COPYING`或`LICENSE`文件，甚至在源代码头文件中。

对于开源许可证，我们还需要为所有许可证指定`LIC_FILES_CHECKSUM`，以便构建系统在许可证发生变化时通知我们。要添加它，我们定位包含许可证的文件或文件部分，并提供其相对于源目录的路径以及其 MD5 校验和。例如：

```
LIC_FILES_CHKSUM = "file://${COREBASE}/meta/files/common- licenses/GPL-2.0;md5=801f80980d171dd6425610833a22dbe6"
LIC_FILES_CHKSUM = "file://COPYING;md5=f7bdc0c63080175d1667091b864cb12c"
LIC_FILES_CHKSUM = "file://usr/include/head.h;endline=7;md5=861ebad4adc7236f8d1905338 abd7eb2"
LIC_FILES_CHKSUM = "file://src/file.c;beginline=5;endline=13;md5=6c7486b21a8524b1879f a159578da31e"
```

专有代码的许可证应设置为`CLOSED`，不需要为其提供`LIC_FILES_CHECKSUM`。

### 获取包内容

`SRC_URI`变量列出了要获取的文件。构建系统将根据文件前缀使用不同的获取器。这些获取器可以是：

+   与元数据一起包含的本地文件（`file://`）。如果本地文件是补丁，`SRC_URI`变量可以通过补丁特定的参数扩展，如下所示：

    +   `striplevel`：默认的补丁去除级别是 1，但可以通过此参数进行修改

    +   `patchdir`：此参数指定应用补丁的目录位置，默认值为源目录

    +   `apply`：此参数控制是否应用补丁，默认情况下会应用补丁

+   存储在远程服务器上的文件（通常为`http(s)://`、`ftp://`或`ssh://`）。

+   存储在远程仓库中的文件（通常为`git://`、`svn://`、`hg://`或`bzr://`）。这些文件也需要一个`SRCREV`变量来指定修订版本。

存储在远程服务器上的文件（不是本地文件或远程仓库）需要指定两个校验和。如果有多个文件，可以通过`name`参数来区分；例如：

```
SRCREV = "04024dea2674861fcf13582a77b58130c67fccd8"
SRC_URI = "git://repo.com/git/ \
           file://fix.patch;name=patch \
           http://example.org/archive.data;name=archive"
SRC_URI[archive.md5sum] = "aaf32bde135cf3815aa3221726bad71e"
SRC_URI[archive.sha256sum] = "65be91591546ef6fdfec93a71979b2b108eee25edbc20c53190caafc9a92d4e7"
```

源目录文件夹`S`指定了源文件的位置。仓库将在此处检出，或者 tarball 会解压到此位置。如果 tarball 解压到标准的`${PN}-${PV}`位置，则可以省略，因为这是默认值。对于仓库，必须始终指定；例如：

```
S = "${WORKDIR}/git"
```

### 指定任务覆盖

所有食谱都继承了`base.bbclass`类，该类定义了以下任务：

+   `do_fetch`：该方法通过`SRC_URI`变量选择获取器，获取源代码。

+   `do_unpack`：该方法将代码解包到由`S`变量指定的位置。

+   `do_configure`：该方法根据需要配置源代码。默认情况下不执行任何操作。

+   `do_compile`：该方法默认编译源代码并运行 GNU make 目标。

+   `do_install`：该方法将构建结果从`build`目录`B`复制到目标目录`D`。默认情况下不执行任何操作。

+   `do_package`：该方法将交付物拆分成多个包。默认情况下不执行任何操作。

通常，仅重写配置、编译和安装任务，并且大多数情况下是通过继承`autotools`类隐式完成的。

对于不使用构建系统的自定义配方，您需要在相应的`do_configure`、`do_compile`和`do_install`重写方法中提供所需的配置（如果有）、编译和安装指令。以下是这种类型配方的示例，位于`meta-custom/recipes-example/helloworld/helloworld_1.0.bb`：

```
DESCRIPTION = "Simple helloworld application"
SECTION = "examples"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4f302"

SRC_URI = "file://helloworld.c"

S = "${WORKDIR}"

do_compile() {
             ${CC} helloworld.c -o helloworld
}

do_install() {
             install -d ${D}${bindir}
             install -m 0755 helloworld ${D}${bindir}
}
```

`meta-custom/recipes-example/helloworld/helloworld-1.0/helloworld.c`源文件如下：

```
#include <stdio.h>

int main(void)
{
    return printf("Hello World");
}
```

我们将在下一章中看到使用最常见构建系统的示例配方。

### 配置包

Yocto 构建系统提供了`PACKAGECONFIG`变量，用于通过定义多个特性来帮助配置包。您的配方通过以下方式定义单个特性：

```
PACKAGECONFIG ??= "feature"
PACKAGECONFIG[feature] = "--with-feature,--without-feature,build- deps-feature,rt-deps-feature"
```

`PACKAGECONFIG`变量包含一个以空格分隔的特性名称列表，并且可以在`bbappend`文件中进行扩展或重写；请查看以下示例：

```
PACKAGECONFIG_append = " feature1 feature2"
```

要从发行版或本地配置文件扩展或重写它，您需要使用以下语法：

```
PACKAGECONFIG_pn-<package_name> = "feature1 feature2"
PACKAGECONFIG_append_pn-<package_name> = " feature1 feature2"
```

接下来，我们使用四个有序参数来描述每个特性：

+   启用特性时的额外配置参数（对于`EXTRA_OECONF`）

+   启用特性时的额外配置参数（对于`EXTRA_OECONF`）

+   启用特性时的额外构建依赖（对于`DEPENDS`）

+   启用特性时的额外运行时依赖（对于`RDEPENDS`）

这四个参数是可选的，但必须保持其顺序，并且需要保留分隔的逗号。

例如，`wpa-supplicant`配方定义了两个特性，`gnutls`和`openssl`，但默认仅启用`gnutls`，如下所示：

```
PACKAGECONFIG ??= "gnutls"
PACKAGECONFIG[gnutls] = ",,gnutls"
PACKAGECONFIG[openssl] = ",,openssl"
```

### 拆分成多个包

将配方内容分成不同的包以满足不同的需求是常见做法。典型的例子是将文档包含在`doc`包中，将头文件和/或库包含在`dev`包中。我们可以通过使用`FILES`变量来实现这一点，示例如下：

```
FILES_${PN} += "List of files to include in the main package"
FILES_${PN}-dbg += "Optional list of files to include in the debug package"
FILES_${PN}-dev += "Optional list of files to include in the development package"
FILES_${PN}-doc += "Optional list of files to include in the documentation package"
```

### 设置特定于机器的变量

每个配方都有一个`PACKAGE_ARCH`变量，用于将配方分类到一个软件包源中，正如我们在*探索镜像内容*的配方中看到的那样。大多数时候，它们由 Yocto 构建系统自动分类。例如，如果配方是内核、内核模块配方、镜像配方，或者是交叉编译或构建本地应用程序，Yocto 构建系统将相应地设置软件包架构。

BitBake 还会查看`SRC_URI`的机器覆盖并调整软件包架构，如果你的配方使用`allarch`类，它将把软件包架构设置为`all`。

因此，在处理仅适用于特定机器或机器系列的配方时，或者涉及特定机器或机器系列的更改时，我们需要检查软件包是否被分类到适当的软件包源中。如果没有，则需要通过在配方中明确指定以下代码行来指定软件包架构：

```
PACKAGE_ARCH = "${MACHINE_ARCH}"
```

此外，当一个配方仅用于特定的机器类型时，我们使用`COMPATIBLE_MACHINE`变量来指定。例如，为了让它仅兼容`mxs、mx5 和 mx6 SoC 系列`，我们会使用以下内容：

```
COMPATIBLE_MACHINE = "(mxs|mx5|mx6)"
```

# 添加数据、脚本或配置文件

所有配方都继承了基类，并执行默认的任务集合。继承基类后，配方知道如何执行诸如获取和编译之类的任务。

由于大多数配方旨在安装某种可执行文件，基类知道如何构建它。但有时我们所需要的仅仅是将数据、脚本或配置文件安装到文件系统中。

如果数据或配置与应用程序相关，最合逻辑的做法是将其与应用程序的配方本身打包在一起，如果我们认为分开安装更好，甚至可以将其拆分成单独的包。

但有时，数据或配置与应用程序无关，可能适用于整个系统，或者我们只想为其提供一个单独的配方。根据需要，我们甚至可能希望安装一些不需要编译的 Perl 或 Python 脚本。

## 如何操作…

在这些情况下，我们的配方应该继承`allarch`类，该类被那些不产生特定架构输出的配方继承。

这种类型的配方示例`meta-custom/recipes-example/example-data/example-data_1.0.bb`可以在这里查看：

```
DESCRIPTION = "Example of data or configuration recipe"
SECTION = "examples"

LICENSE = "GPLv2"
LIC_FILES_CHKSUM = "file://${COREBASE}/meta/files/common-licenses/GPL-2.0;md5=801f80980d171dd6425610833a22dbe6"

SRCREV = "${AUTOREV}"
SRC_URI = "git://github.com/yoctocookbook/examples.git \
           file://example.data"

S = "${WORKDIR}/git"

inherit allarch

do_compile() {
}

do_install() {
        install -d ${D}${sysconfdir}
        install -d ${D}${sbindir}
        install -m 0755 ${WORKDIR}/example.data ${D}/${sysconfdir}/
        install -m 0755 ${S}/python-scripts/* ${D}/${sbindir}
}
```

它假设虚构的`examples.git`仓库包含一个我们想要包括在根文件系统中的`python-scripts`文件夹。

一个可用的配方示例可以在与本书附带的源代码中找到。

# 管理用户和组

在很多情况下，我们还需要向文件系统中添加或修改用户和组。这个配方解释了如何做到这一点。

## 准备工作

用户信息存储在`/etc/passwd`文件中，这是一个文本文件，用作系统用户信息的数据库。`passwd`文件是人类可读的。

每一行对应系统中的一个用户，格式如下：

```
<username>:<password>:<uid>:<gid>:<comment>:<home directory>:<login shell>
```

让我们看看这种格式的每个参数：

+   `username`：一个唯一的字符串，用于标识用户在登录时的身份

+   `uid`：用户 ID，Linux 用来识别用户的数字

+   `gid`：组 ID，Linux 用来识别用户主组的数字

+   `comment`：以逗号分隔的值，用来描述账户，通常是用户的联系方式

+   `home directory`：用户主目录的路径

+   `login shell`：为交互式登录启动的 Shell

默认的`passwd`文件存储在`base-passwd`包中，格式如下：

```
root::0:0:root:/root:/bin/sh
daemon:*:1:1:daemon:/usr/sbin:/bin/sh
bin:*:2:2:bin:/bin:/bin/sh
sys:*:3:3:sys:/dev:/bin/sh
sync:*:4:65534:sync:/bin:/bin/sync
games:*:5:60:games:/usr/games:/bin/sh
man:*:6:12:man:/var/cache/man:/bin/sh
lp:*:7:7:lp:/var/spool/lpd:/bin/sh
mail:*:8:8:mail:/var/mail:/bin/sh
news:*:9:9:news:/var/spool/news:/bin/sh
uucp:*:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:*:13:13:proxy:/bin:/bin/sh
www-data:*:33:33:www-data:/var/www:/bin/sh
backup:*:34:34:backup:/var/backups:/bin/sh
list:*:38:38:Mailing List Manager:/var/list:/bin/sh
irc:*:39:39:ircd:/var/run/ircd:/bin/sh
gnats:*:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:*:65534:65534:nobody:/nonexistent:/bin/sh
```

所有账户的直接登录功能都被禁用，密码字段上会显示一个星号，除 root 用户外，root 没有密码。这是因为默认情况下，镜像是通过启用`debug-tweaks`功能构建的，该功能启用了 root 用户的无密码登录功能。若启用 root 密码，则会显示加密后的 root 密码。

### 注意

不要忘记从生产镜像中移除`debug-tweaks`功能。

在相同时间安装的`/etc/group`文件包含系统组的信息。

`core-image-minimal`镜像不包括影子密码保护，但其他镜像，如`core-image-full-cmdline`，是包括的。启用时，所有密码字段会包含一个* x *，加密后的密码存储在`/etc/shadow`文件中，该文件只有超级用户才能访问。

系统所需的任何用户，如果不在我们之前看到的列表中，也需要被创建。

## 如何操作……

食谱添加或修改系统用户或组的标准方法是使用`useradd`类，该类使用以下变量：

+   `USERADD_PACKAGES`：此变量指定食谱中需要添加用户或组的独立包。对于主包，您可以使用以下内容：

    ```
    USERADD_PACKAGES = "${PN}"
    ```

+   `USERADD_PARAM`：此变量对应传递给 Linux `useradd`命令的参数，用于向系统添加新用户。

+   `GROUPADD_PARAM`：此变量对应传递给 Linux `groupadd`命令的参数，用于向系统添加新组。

+   `GROUPMEMS_PARAM`：此变量对应传递给 Linux `groupmems`命令的参数，用于管理用户主组的成员。

以下是使用`useradd`类的食谱示例片段：

```
inherit useradd

PASSWORD ?= "miDBHFo2hJSAA"
USERADD_PACKAGES = "${PN}"
USERADD_PARAM_${PN} = "--system --create-home \
                       --groups tty \
                       --password ${PASSWORD} \
                       --user-group ${PN}"
```

密码可以通过在主机上使用`mkpasswd` Linux 命令行工具生成，该工具随着`whois` Ubuntu 包一起安装。

## 还有更多……

在使用`useradd`类生成用户和组时，`uid`和`gid`值会在包安装过程中动态分配。如果不希望这样，可以通过提供自己的`passwd`和组文件来分配全局静态的`uid`和`gid`值。

要做到这一点，你需要在`conf/local.conf`文件中定义`USERADDEXTENSION`变量，如下所示：

```
USERADDEXTENSION = "useradd-staticids"
```

构建系统将搜索 `BBPATH` 变量以获取 `files/passwd` 和 `files/group` 文件，以获取 `uid` 和 `gid` 值。这些文件具有先前定义的标准 `passwd` 布局，密码字段被忽略。

可以使用 `USERADD_UID_TABLES` 和 `USERADD_GID_TABLES` 变量重写默认文件名。

您还需要定义以下内容：

```
USERADD_ERROR_DYNAMIC = "1"
```

这样做是为了当提供的文件中找不到所需的 `uid` 和 `gid` 值时，构建系统产生错误。

### 注意

请注意，如果在已构建的项目中使用 `useradd` 类，您需要删除 `tmp` 目录并从 `sstate-cache` 目录重新构建，否则将出现构建错误。

还有一种方法可以添加与特定配方无关但与映像相关的用户和组信息 - 使用 `extrausers` 类。它通过映像配方中的 `EXTRA_USERS_PARAMS` 变量配置，并如下使用：

```
inherit extrausers

EXTRA_USERS_PARAMS = "\
  useradd -P password root; \
  "
```

这将 root 密码设置为 `password`。

# 使用 sysvinit 初始化管理器

初始化管理器是根文件系统的重要部分。这是内核执行的第一件事，并且有责任启动系统的其余部分。

该配方将介绍 `sysvinit` 初始化管理器。

## 准备工作

这是 Yocto 中的默认初始化管理器，并且自操作系统起源以来一直在 Linux 中使用。内核传递一个 `init` 命令行参数，通常为 `/sbin/init`，然后启动它。此 `init` 进程具有 PID 1 并且是所有进程的父进程。`init` 进程可以由 BusyBox 实现，也可以是与 `sysvinit` 包独立安装的独立程序。它们都以相同的方式工作，基于 **运行级别** 的概念，即定义要运行哪些进程的机器状态。

`init` 进程将读取一个 `inittab` 文件，并查找默认运行级别。默认的 `inittab` 文件安装在 `sysvinit-inittab` 包中，并且如下所示：

```
# /etc/inittab: init(8) configuration.
# $Id: inittab,v 1.91 2002/01/25 13:35:21 miquels Exp $

# The default runlevel.
id:5:initdefault:

# Boot-time system configuration/initialization script.
# This is run first except when booting in emergency (-b) mode.
si::sysinit:/etc/init.d/rcS

# What to do in single-user mode.
~~:S:wait:/sbin/sulogin

# /etc/init.d executes the S and K scripts upon change
# of runlevel.
#
# Runlevel 0 is halt.
# Runlevel 1 is single-user.
# Runlevels 2-5 are multi-user.
# Runlevel 6 is reboot.

l0:0:wait:/etc/init.d/rc 0
l1:1:wait:/etc/init.d/rc 1
l2:2:wait:/etc/init.d/rc 2
l3:3:wait:/etc/init.d/rc 3
l4:4:wait:/etc/init.d/rc 4
l5:5:wait:/etc/init.d/rc 5
l6:6:wait:/etc/init.d/rc 6
# Normally not reached, but fallthrough in case of emergency.
z6:6:respawn:/sbin/sulogin
```

然后，`init` 运行 `/etc/rcS.d` 目录中以 `S` 开头的所有脚本，然后运行 `/etc/rcN.d` 目录中以 `S` 开头的所有脚本，其中 `N` 是运行级别值。

因此，`init` 进程只执行初始化并忽略进程。如果发生故障并且进程被终止，没有人会关心。如果系统不响应，系统看门狗将重新启动系统，但通常需要具有某种类型进程监视器以响应系统健康的应用程序，但 `sysvinit` 不提供这些类型的机制。

然而，`sysvinit` 是一个被充分理解和可靠的初始化管理器，建议保留，除非需要一些额外的功能。

## 如何做到...

在使用 `sysvinit` 作为初始化管理器时，Yocto 提供 `update-rc.d` 类作为辅助工具，安装初始化脚本，以便在需要时启动和停止它们。

使用此类时，你需要指定`INITSCRIPT_NAME`变量，该变量指定要安装的脚本名称，`INITSCRIPT_PARAMS`指定传递给`update-rc.d`工具的选项。你可以选择性地使用`INITSCRIPT_PACKAGES`变量列出包含初始化脚本的包。默认情况下，这只包含主包，如果提供多个包，则需要为每个包指定`INITSCRIPT_NAME`和`INITSCRIPT_PARAMS`，并使用覆盖方式。以下是一个示例片段：

```
INITSCRIPT_PACKAGES = "${PN}-httpd ${PN}-ftpd"
INITSCRIPT_NAME_${PN}-httpd = "httpd.sh"
INITSCRIPT_NAME_${PN}-ftpd = "ftpd.sh"
INITSCRIPT_PARAMS_${PN}-httpd = "defaults"
INITSCRIPT_PARAMS_${PN}-ftpd = "start 99 5 2 . stop 20 0 1 6 ."
```

当初始化脚本未绑定到特定的食谱时，我们可以为其添加一个特定的食谱。例如，以下食谱将在`recipes-example/sysvinit-mount/sysvinit-mount_1.0.bb`文件中运行`mount.sh`脚本。

```
DESCRIPTION = "Initscripts for mounting filesystems"
LICENSE = "MIT"

LIC_FILES_CHKSUM = "file://${COMMON_LICENSE_DIR}/MIT;md5=0835ade698e0bcf8506ecda2f7b4 f302"

SRC_URI = "file://mount.sh"

INITSCRIPT_NAME = "mount.sh"
INITSCRIPT_PARAMS = "start 09 S ."

inherit update-rc.d

S = "${WORKDIR}"

do_install () {
    install -d ${D}${sysconfdir}/init.d/
    install -c -m 755 ${WORKDIR}/${INITSCRIPT_NAME} ${D}${sysconfdir}/init.d/${INITSCRIPT_NAME}
}
```

# 使用`systemd`初始化管理器

作为`sysvinit`的替代方案，你可以配置项目使用`systemd`作为初始化管理器，尽管`systemd`包含更多功能。

## 准备工作

`systemd`初始化管理器正在取代大多数 Linux 发行版中的`sysvinit`和其他初始化管理器。它基于单元的概念，单元是所有与系统启动和维护相关的元素的抽象，而目标是将单元分组，并且可以视为运行级别的等价物。`systemd`定义的一些单元包括：

+   服务

+   套接字

+   设备

+   挂载点

+   快照

+   定时器

+   路径

默认目标及其对应的运行级别在下表中定义：

| Sysvinit | 运行级别 | Systemd 目标 | 备注 |
| --- | --- | --- | --- |
| 0 | `runlevel0.target` | `poweroff.target` | 关闭系统。 |
| 1, s, single | `runlevel1.target` | `rescue.target` | 单用户模式。 |
| 2, 4 | `runlevel2.target`, `runlevel4.target` | `multi-user.target` | 用户定义的/特定站点的运行级别。默认情况下，与`3`相同。 |
| 3 | `runlevel3.target` | `multi-user.target` | 多用户、非图形模式。用户通常可以通过多个控制台或网络登录。 |
| 5 | `runlevel5.target` | `graphical.target` | 多用户、图形模式。通常包括运行级别 3 的所有服务以及图形登录。 |
| 6 | `runlevel6.target` | `reboot.target` | 重启系统。 |

`systemd`初始化管理器设计上与`sysvinit`兼容，包括使用`sysvinit init`脚本。

`systemd`的一些特点包括：

+   允许更快启动时间的并行化功能

+   通过套接字和 D-Bus 进行服务初始化，以便仅在需要时启动服务

+   允许进程失败恢复的进程监控

+   系统状态快照和恢复

+   挂载点管理

+   基于事务依赖的单元控制，其中单元之间建立了依赖关系

## 如何操作...

要配置你的系统使用`systemd`，你需要通过将以下内容添加到分发配置文件中，在`poky`分发的默认配置文件` sources/poky/meta-yocto/conf/distro/poky.conf`中，或者在你项目的`conf/local.conf`文件中，来将`systemd`分发功能添加到项目中：

```
DISTRO_FEATURES_append = " systemd"
```

### 注意

注意开始引号后的空格。

```
VIRTUAL-RUNTIME_init_manager = "systemd"
```

此配置示例允许你定义一个带有`systemd`的主镜像和一个带有`sysvinit`的救援镜像，前提是它不使用`VIRTUAL-RUNTIME_init_manager`变量。因此，救援镜像不能使用`packagegroup-core-boot`或`packagegroup-core-full-cmdline`食谱。例如，本章稍后将介绍的*减少根文件系统镜像大小*食谱中的镜像大小已被减小，可以作为救援镜像的基础。

要完全从系统中删除`sysvinit`，你需要执行以下操作：

```
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"
VIRTUAL-RUNTIME_initscripts = ""
```

功能回填是自动扩展机器和分发功能，以保持向后兼容性。`sysvinit`分发功能会自动回填，因此，要删除它，我们需要将其添加到`DISTRO_FEATURES_BACKFILL_CONSIDERED`变量中，如前所述。

### 提示

请注意，如果你正在使用现有项目，并且按照前文所述更改了`DISTRO_FEATURES`变量，你需要删除`tmp`目录并使用` sstate-cache`重新构建，否则构建将失败。

## 还有更多...

不仅根文件系统需要配置，Linux 内核也需要特别配置，以满足`systemd`所需的所有功能。`systemd`源代码 README 文件中有一份详细的内核配置变量列表。举例来说，要扩展我们将在本章稍后介绍的*减少 Linux 内核镜像大小*食谱中的最小内核配置，以支持 Wandboard 的`systemd`，我们需要在`arch/arm/configs/wandboard-quad_minimal_defconfig`文件中添加以下配置更改：

```
+CONFIG_FHANDLE=y
+CONFIG_CGROUPS=y
+CONFIG_SECCOMP=y
+CONFIG_NET=y
+CONFIG_UNIX=y
+CONFIG_INET=y
+CONFIG_AUTOFS4_FS=y
+CONFIG_TMPFS=y
+CONFIG_TMPFS_POSIX_ACL=y
+CONFIG_SCHEDSTATS=y
```

为 Wandboard 提供的默认内核配置将正常启动一个带有`systemd`的`core-image-minimal`镜像。

### 安装 systemd 单元文件

Yocto 提供了`systemd`类作为辅助工具来安装单元文件。默认情况下，单元文件会安装在目标目录的`${systemd_unitdir}/system`路径下。

使用此类时，需要指定`SYSTEMD_SERVICE_${PN}`变量，值为要安装的单元文件的名称。你也可以选择使用`SYSTEMD_PACKAGES`变量来列出包含单元文件的包。默认情况下，这只包含主包，如果提供多个包，则需要使用覆盖方式指定`SYSTEMD_SERVICE`变量。

服务默认配置为在启动时自动启动，但你可以通过`SYSTEMD_AUTO_ENABLE`变量来更改此设置。

以下是一个示例代码片段：

```
SYSTEMD_PACKAGES = "${PN}-syslog"
SYSTEMD_SERVICE_${PN}-syslog = "busybox-syslog.service"
SYSTEMD_AUTO_ENABLE = "disabled"
```

# 安装包安装脚本

支持的包格式（RPM、ipk 和 deb）支持在包安装过程中添加安装脚本，这些脚本可以在不同的时间运行。在本配方中，我们将看到如何安装这些脚本。

## 准备工作

有不同类型的安装脚本：

+   **预安装脚本** (`pkg_preinst`)：这些脚本在软件包解包之前调用。

+   **后安装脚本** (`pkg_postinst`)：这些脚本在软件包解包后调用，并且会配置依赖关系。

+   **卸载前脚本** (`pkg_prerm`)：这些脚本在已安装或至少部分安装的软件包时调用。

+   **卸载后脚本** (`pkg_postrm`)：这些脚本在软件包的文件被移除或替换之后调用。

## 如何实现……

以下是一个在配方中安装预安装脚本的示例片段：

```
     pkg_preinst_${PN} () {
         # Shell commands
     }
```

所有安装脚本的工作方式相同，不同之处在于，后安装脚本可以在主机上创建根文件系统镜像时、在目标设备上（对于无法在主机上执行的操作），或者在软件包直接安装到目标设备时运行。请查看以下代码：

```
 pkg_postinst_${PN} () {
     if [ x"$D" = "x" ]; then
          # Commands to execute on device
     else
          # Commands to execute on host
     fi
 }
```

如果后安装脚本成功，软件包将被标记为已安装。如果脚本失败，软件包将被标记为未解包，脚本将在镜像再次启动时执行。

## 它是如何工作的……

一旦配方定义了安装脚本，特定包类型的类将在遵循特定格式的打包规则的同时安装该脚本。

对于后安装脚本，当在主机上运行时，`D` 被设置为目标目录，因此比较测试将失败。但当在目标设备上运行时，`D` 将为空。

### 注意

如果可能，建议在主机上执行后安装脚本，因为我们需要考虑到某些根文件系统将是只读的，因此无法在目标设备上执行某些操作。

# 减少 Linux 内核镜像大小

在根文件系统定制之前或与之并行，嵌入式项目通常需要进行镜像大小优化，以减少启动时间和内存使用。

更小的镜像意味着更少的存储空间、传输时间和编程时间，从而在制造和现场更新中节省成本。

默认情况下，`wandboard-quad` 的压缩 Linux 内核镜像（**zImage**）大约为 5.2 MB。这个配方将展示我们如何减少该镜像的大小。

## 如何实现……

一个能够从 microSD 卡根文件系统启动的 Wandboard 的最小内核配置示例是以下的 `arch/arm/configs/wandboard-quad_minimal_defconfig` 文件：

```
CONFIG_KERNEL_XZ=y
CONFIG_NO_HZ=y
CONFIG_HIGH_RES_TIMERS=y
CONFIG_BLK_DEV_INITRD=y
CONFIG_CC_OPTIMIZE_FOR_SIZE=y
CONFIG_EMBEDDED=y
CONFIG_SLOB=y
CONFIG_ARCH_MXC=y
CONFIG_SOC_IMX6Q=y
CONFIG_SOC_IMX6SL=y
CONFIG_SMP=y
CONFIG_VMSPLIT_2G=y
CONFIG_AEABI=y
CONFIG_CPU_FREQ=y
CONFIG_ARM_IMX6_CPUFREQ=y
CONFIG_CPU_IDLE=y
CONFIG_VFP=y
CONFIG_NEON=y
CONFIG_DEVTMPFS=y
CONFIG_DEVTMPFS_MOUNT=y
CONFIG_PROC_DEVICETREE=y
CONFIG_SERIAL_IMX=y
CONFIG_SERIAL_IMX_CONSOLE=y
CONFIG_REGULATOR=y
CONFIG_REGULATOR_ANATOP=y
CONFIG_MMC=y
CONFIG_MMC_SDHCI=y
CONFIG_MMC_SDHCI_PLTFM=y
CONFIG_MMC_SDHCI_ESDHC_IMX=y
CONFIG_DMADEVICES=y
CONFIG_IMX_SDMA=y
CONFIG_EXT3_FS=y
```

该配置生成一个 886 KB 压缩的 Linux 内核镜像（zImage）。

## 它是如何工作的……

除了硬件设计考虑因素（如从 NOR 闪存运行 Linux 内核，并使用**就地执行** (**XIP**) 来避免将镜像加载到内存中），内核大小优化的第一步是审查内核配置并移除所有多余的功能。

要分析内核块的大小，我们可以使用：

```
$ size vmlinux */built-in.o
text    data     bss     dec     hex filename
8746205  356560  394484 9497249  90eaa1 vmlinux
117253    2418    1224  120895   1d83f block/built-in.o
243859   11158      20  255037   3e43d crypto/built-in.o
2541356  163465   34404 2739225  29cc19 drivers/built-in.o
1956       0       0    1956     7a4 firmware/built-in.o
1728762   18672   10544 1757978  1ad31a fs/built-in.o
20361   14701     100   35162    895a init/built-in.o
29628     760       8   30396    76bc ipc/built-in.o
576593   20644  285052  882289   d7671 kernel/built-in.o
106256   24847    2344  133447   20947 lib/built-in.o
291768   14901    3736  310405   4bc85 mm/built-in.o
1722683   39947   50928 1813558  1bac36 net/built-in.o
34638     848     316   35802    8bda security/built-in.o
276979   19748    1332  298059   48c4b sound/built-in.o
138       0       0     138      8a usr/built-in.o

```

在这里，`vmlinux`是 Linux 内核 ELF 镜像，可以在 Linux 的`build`目录中找到。

一些通常需要排除的内容包括：

+   移除 IPv6（`CONFIG_IPV6`）和其他多余的网络功能

+   如果不需要，请移除块设备（`CONFIG_BLOCK`）

+   如果未使用，请移除加密特性（`CONFIG_CRYPTO`）

+   审查支持的文件系统类型并移除不需要的文件系统，例如在没有闪存设备上使用的闪存文件系统

+   避免模块，如果可能，移除内核中的模块支持（`CONFIG_MODULES`）。

一个好的策略是从一个最小的内核开始，逐步添加必要的功能直到你得到一个工作系统。从`allnoconfig` GNU make 目标开始，并审查`CONFIG_EXPERT`和`CONFIG_EMBEDDED`下的配置项，因为这些配置项不包含在`allnoconfig`设置中。

以下是一些可能不太明显，但在不移除功能的情况下显著减少镜像大小的配置更改：

+   将默认压缩方法从**Lempel–Ziv–Oberhumer** (**LZO**)改为 XZ（`CONFIG_KERNEL_XZ`）。不过，解压速度可能稍微慢一些。

+   将分配器从 SLUB 更改为**简单块列表** (**SLOB**)（`CONFIG_SLOB`），适用于内存较小的小型嵌入式系统。

+   除非你有 4GB 或更多内存，否则请不要使用高内存（`CONFIG_HIGHMEM`）。

你可能还希望为生产和开发系统设置不同的配置，因此你可以从生产镜像中移除以下内容：

+   `printk`支持（`CONFIG_PRINTK`）

+   `tracing`支持（`CONFIG_FTRACE`）

在编译方面，使用`CONFIG_CC_OPTIMIZE_FOR_SIZE`进行大小优化。

一旦基本配置完成，我们需要分析内核功能，找出进一步的优化空间。你可以使用以下命令打印一个已排序的内核符号列表：

```
$ nm --size-sort --print-size -r vmlinux | head
 808bde04 00040000 B __log_buf
 8060f1c0 00004f15 r kernel_config_data
 80454190 000041f0 T hidinput_connect
 80642510 00003d40 r drm_dmt_modes
 8065cbbc 00003414 R v4l2_dv_timings_presets
 800fbe44 000032c0 T __blockdev_direct_IO
 80646290 00003100 r edid_cea_modes
 80835970 00003058 t imx6q_clocks_init
 8016458c 00002e74 t ext4_fill_super
 8056a814 00002aa4 T hci_event_packet

```

然后，你需要查看内核源码以找到优化点。

可以通过运行中的 Wandboard 内核日志获取未压缩内核在内存中实际使用的空间，如下所示：

```
$ dmesg | grep -A 3 "text"
 .text : 0x80008000 - 0x80a20538   (10338 kB)
 .init : 0x80a21000 - 0x80aae240   ( 565 kB)
 .data : 0x80ab0000 - 0x80b13644   ( 398 kB)
 .bss  : 0x80b13644 - 0x80b973fc   ( 528 kB)

```

从这里开始，`.text`部分包含代码和常量数据，`.data`部分包含变量的初始化数据，`.bss`部分包含所有未初始化的数据。`.init`部分仅包含在 Linux 初始化期间使用的全局变量，这些变量在之后会被释放，如下所示的 Linux 内核启动信息所示：

```
Freeing unused kernel memory: 564K (80a21000 - 80aae000)
```

当前正在进行的工作旨在减少 Linux 内核的大小，因此预计新版本的内核将更小，并且能够更好地定制以用于嵌入式系统。

# 减小根文件系统镜像大小

默认情况下，`wandboard-quad` 解包后的 `core-image-minimal` 大约为 45 MB，而 `core-image-sato` 大约为 150 MB。本文将探讨减小它们大小的方法。

## 如何操作...

下面显示了一个小映像示例 `core-image-small`，不包括 `packagegroup-core-boot` 配方，并且可用作根文件系统映像的基础，`recipes-core/images/core-image-small.bb`：

```
DESCRIPTION = "Minimal console image."

IMAGE_INSTALL= "\
        base-files \
        base-passwd \
        busybox \
        sysvinit \
        initscripts \
        ${ROOTFS_PKGMANAGE_BOOTSTRAP} \
        ${CORE_IMAGE_EXTRA_INSTALL} \
"

IMAGE_LINGUAS = " "

LICENSE = "MIT"

inherit core-image

IMAGE_ROOTFS_SIZE ?= "8192"
```

该配方生成约 6.4 MB 的映像。如果使用 `poky-tiny` 发行版，可以通过在 `conf/local.conf` 文件中添加以下内容进一步减小。

```
DISTRO = "poky-tiny"
```

`poky-tiny` 发行版进行了一系列大小优化，可能会限制您可以包含在映像中的软件包集。要成功构建此映像，您必须跳过 Yocto 构建系统执行的某个完整性检查之一，方法是添加以下内容：

```
INSANE_SKIP_glibc-locale = "installed-vs-shipped"
```

使用 `poky-tiny`，映像的大小进一步减小至约 4 MB。

可以进一步减小映像的大小；例如，我们可以将 `sysvinit` 替换为 `tiny-init`，但这留给读者作为练习。

大小缩小的映像还与生产映像一起用于救援系统和制造测试流程。它们还非常适合作为 `initramfs` 映像，即 Linux 内核从内存中挂载的映像，甚至可以打包成单个 Linux 内核映像二进制文件。

## 工作原理...

从适当的映像（如 `core-image-minimal`）开始，并分析依赖项，如 第一章 中的 *调试构建系统* 配方所示，决定哪些不需要。您还可以使用映像构建历史中列出的文件大小，如 *使用构建历史* 配方中所见，也在 第一章 中的 *构建系统* 中，对文件大小按照 `files-in-image.txt` 文件的第四列进行逆序排序。

```
$ sort -r -g  -k 4,4 files-in-image.txt -o sorted-files-in-image.txt
sorted-files-in-image.txt:
-rwxr-xr-x root       root          1238640 ./lib/libc-2.19.so
-rwxr-xr-x root       root           613804 ./sbin/ldconfig
-rwxr-xr-x root       root           539860 ./bin/busybox.nosuid
-rwxr-xr-x root       root           427556 ./lib/libm-2.19.so
-rwxr-xr-x root       root           130304 ./lib/ld-2.19.so
-rwxr-xr-x root       root            88548 ./lib/libpthread-2.19.so
-rwxr-xr-x root       root            71572 ./lib/libnsl-2.19.so
-rwxr-xr-x root       root            71488 ./lib/libresolv-2.19.so
-rwsr-xr-x root       root            51944 ./bin/busybox.suid
-rwxr-xr-x root       root            42668 ./lib/libnss_files- 2.19.so
-rwxr-xr-x root       root            30536 ./lib/libnss_compat- 2.19.so
-rwxr-xr-x root       root            30244 ./lib/libcrypt-2.19.so
-rwxr-xr-x root       root            28664 ./sbin/init.sysvinit
-rwxr-xr-x root       root            26624 ./lib/librt-2.19.so

```

从中我们可以观察到 `glic` 是文件系统大小的最大贡献者。在仅限控制台系统上可以节省空间的其他地方包括：

+   使用 IPK 软件包管理器，因为它是最轻量的，或者更好的是，完全从生产根文件系统中删除 `package-management` 功能。

+   在 `conf/local.conf` 文件中指定 BusyBox 的 `mdev` 设备管理器，而不是 `udev`，如下所示：

    ```
    VIRTUAL-RUNTIME_dev_manager = "mdev"
    ```

    请注意，这仅适用于包括 `packagegroup-core-boot` 的核心映像。

+   如果我们在块设备上运行根文件系统，请在没有日志的情况下使用 ext2 而不是 ext3 或 ext4。

+   通过在 `bbappend` 中提供自己的配置文件，仅配置 BusyBox 的基本应用程序：

+   审查`glibc`配置，可以通过`DISTRO_FEATURES_LIBC`分发配置变量进行更改。它的使用示例可以在`poky-tiny`分发中找到，该分发包含在`poky`源代码中。`poky-tiny`分发可以作为定制小型系统分发的模板。

+   考虑切换到比默认的`glibc`更轻量的`C`库。曾经，`uclibc`被用作替代方案，但该库似乎在过去几年未再维护，并且当前的 Wandboard 的`core-image-minimal`镜像无法使用它进行构建。

    ### 注意

    最近，**musl**（[`www.musl-libc.org/`](http://www.musl-libc.org/)）有了一些新的动向，它是一个新的 MIT 许可证的`C`库。要启用它，你需要将以下内容添加到你的`conf/local.conf`文件中：

    TCLIBC = "musl"

    你需要将`meta-musl`层（[`github.com/kraj/meta-musl`](https://github.com/kraj/meta-musl)）添加到你的`conf/bblayers.conf`文件中。

    它目前为 QEMU 目标构建`core-image-minimal`，但仍然需要一些工作才能在真实硬件（如 Wandboard）上使用它。

+   使用`-Os`编译你的应用程序，以优化大小。

# 发布软件

当发布基于 Yocto 项目的产品时，我们必须考虑到我们是建立在许多不同的开源项目之上的，每个项目都有不同的许可要求。

至少，你的嵌入式产品将包含一个引导加载程序（可能是 U-Boot）、Linux 内核以及一个包含一个或多个应用程序的根文件系统。U-Boot 和 Linux 内核都使用**通用公共许可证第 2 版**（**GPLv2**）授权。根文件系统可能包含具有不同许可证的各种程序。

所有开源许可证都允许你销售一个商业产品，该产品可以同时包含专有和开源许可证的混合，只要它们是独立的，并且产品符合所有开源许可证的要求。我们将在稍后的*与开源和专有代码共存*食谱中讨论开源与专有代码的共存。

在将你的产品发布到公众之前，理解所有许可的影响非常重要。Yocto 项目提供了工具，使得处理许可要求变得更加简单。

## 准备工作

我们首先需要明确我们在使用 Yocto 项目构建的产品中需要遵守的要求。对于最严格的开源许可证，这通常意味着：

+   源代码分发，包括修改

+   许可文本分发

+   分发构建和运行软件所用的工具

## 如何操作...

我们可以使用`archiver`类来提供需要分发的成果物，以遵守许可要求。我们可以将其配置为：

+   提供原始未修补源代码作为 tar 包

+   提供应用于原始源代码的补丁

+   提供用于构建源代码的配方

+   提供有时必须附带的许可证文本（根据某些许可证的要求）

为了使用前面提到的`archiver`类，我们需要在`conf/local.conf`文件中添加以下内容：

```
INHERIT += "archiver"
ARCHIVER_MODE[src] = "original"
ARCHIVER_MODE[diff] = "1"
ARCHIVER_MODE[recipe] = "1"
COPY_LIC_MANIFEST = "1"
COPY_LIC_DIRS = "1"
```

源代码将在`tmp/deploy/sources`目录下的一个许可证子目录层次中提供。

对于`wandboard-quad`，我们可以在`tmp/deploy/sources`下找到以下目录：

+   `allarch-poky-linux`

+   `arm-poky-linux-gnueabi`

在查看分发的 Linux 内核源代码时，作为 GPLv2 软件包，我们可以在`tmp/deploy/sources/arm-poky-linux-gnueabi/linux-wandboard-3.10.17-r0`下找到以下内容：

+   `defconfig`

+   `github.com.wandboard-org.linux.git.tar.gz`

+   `linux-wandboard-3.10.17-r0-recipe.tar.gz`

所以我们有了内核配置、源代码压缩包，以及用于构建它的配方，其中包括：

+   `linux-wandboard_3.10.17.bb`

+   `linux-dtb.inc`

+   `linux-wandboard.inc`

根文件系统软件包的许可证文本也将包含在根文件系统中的`/usr/share/common-licenses`目录下，并按照软件包的目录层次进行存放。

该配置将为所有构建软件包提供交付物，但我们真正想做的是仅为那些许可证要求我们提供交付物的软件包提供它们。

当然，我们不希望盲目地分发`source`目录中的所有内容，因为它还会包含我们的专有源代码，而我们很可能不希望分发这些代码。

我们可以配置`archiver`类，仅为 GPL 和 LGPL 软件包提供源代码，方法如下：

```
COPYLEFT_LICENSE_INCLUDE = "GPL* LGPL*"
COPYLEFT_LICENSE_EXCLUDE = "CLOSED Proprietary"
```

此外，对于嵌入式产品，我们通常只关心实际出现在产品中的软件，因此我们可以通过以下方式限制要归档的配方类型，仅限于目标镜像：

```
COPYLEFT_RECIPE_TYPES = "target"
```

我们应当获得法律建议，以决定哪些软件包的许可证要求我们进行源代码分发。

还有其他配置选项，例如提供补丁或配置过的源代码，而不是分开的原始源代码和补丁，或者提供源`rpm`包而不是源代码压缩包。有关更多详情，请参见`archiver`类。

## 还有更多内容…

我们还可以选择分发整个构建环境。通常做到这一点的最佳方法是将我们的 BSP 和软件层发布到公共 Git 仓库中。我们的软件层可以提供`bblayers.conf.sample`和`local.conf.sample`，用于设置即用型`build`目录。

## 另请参见

+   这里没有讨论的其他要求包括选择的分发机制。建议在发布产品之前获得法律建议，以确保所有许可证义务都已履行。

# 分析您的系统以确保合规性

Yocto 构建系统使得向我们的法律顾问提供审计信息变得更加容易。此配方将解释如何操作。

## 如何做...

在`tmp/deploy/licenses`下，我们可以找到一个包含软件包（及其相应许可证）的目录列表，以及一个包含软件包和许可证清单的`image`文件夹。

对于之前提供的示例镜像`core-image-small`，我们有以下内容：

```
tmp/deploy/licenses/core-image-small-wandboard-quad-<timestamp>/package.manifest
base-files
base-passwd
busybox
busybox-syslog
busybox-udhcpc
initscripts
initscripts-functions
libc6
run-postinsts
sysvinit
sysvinit-inittab
sysvinit-pidof
update-alternatives-opkg
update-rc.d
```

相应的`tmp/deploy/licenses/core-image-small-wandboard-quad-<timestamp>/license.manifest`文件摘录如下：

```
PACKAGE NAME: base-files
PACKAGE VERSION: 3.0.14
RECIPE NAME: base-files
LICENSE: GPLv2

PACKAGE NAME: base-passwd
PACKAGE VERSION: 3.5.29
RECIPE NAME: base-passwd
LICENSE: GPLv2+
```

这些文件可以用来分析构成我们根文件系统的所有不同包。我们还可以对它们进行审核，以确保在公开发布产品时遵守许可证。

## 还有更多内容

你可以通过使用`INCOMPATIBLE_LICENSE`配置变量来指示 Yocto 构建系统特意避免某些许可证。通常使用它的方法是通过在`conf/local.conf`文件中添加以下内容来避免使用 GPLv3 类型的许可证：

```
INCOMPATIBLE_LICENSE = "GPL-3.0 LGPL-3.0 AGPL-3.0"
```

这将构建`core-image-minimal`和`core-image-base`镜像，只要没有包含额外的镜像特性。

# 使用开源和专有代码

嵌入式产品通常是基于像 Yocto 这样的开源系统构建的，并且包含增加价值和专门化产品的专有软件。这个专有部分通常是知识产权，需要得到保护，了解它如何与开源软件共存非常重要。

本食谱将讨论一些常见的开源包示例，这些包通常出现在嵌入式产品中，并简要解释如何将专有软件与它们一起使用。

## 如何做到这一点...

开源许可证可以大致分为两类，取决于它们是否：

+   **宽松性**：这些许可证类似于**互联网软件协会**（**ISC**）、MIT 和 BSD 许可证。它们附带的要求较少，只要求我们保留版权声明。

+   **限制性**：这些许可证类似于 GPL，它们要求我们不仅要分发源代码和修改版本，无论是与二进制文件一起还是在后期分发，还需要分发构建、安装和运行源代码的工具。

然而，某些许可证可能会“污染”修改和衍生作品的条件，这些通常被称为*病毒性许可证*，而其他一些则不会。例如，如果你将应用程序链接到 GPL 许可的代码，那么你的应用程序也将受到 GPL 的约束。

GPL 的病毒性特性让一些人对使用 GPL 许可的软件保持警惕，但需要注意的是，专有软件可以与 GPL 软件共存，只要理解并遵守许可证条款。

例如，违反 GPLv2 许可证将意味着失去将来分发 GPLv2 代码的权利，即使进一步的分发符合 GPLv2 规定。在这种情况下，能够再次分发代码的唯一方法是请求版权持有者的许可。

## 它是如何工作的...

接下来，我们将提供有关一些常见用于嵌入式产品的开源包的许可要求的指南。这并不构成法律建议，如前所述，在公开发布之前，应该对你的产品进行适当的法律审查。

### U-Boot 引导程序

U-Boot 采用 GPLv2 许可证，但由它启动的任何程序不会继承其许可证。因此，你可以自由使用 U-Boot 启动专有操作系统。例如，你可以使用 U-Boot 启动一个专有操作系统。然而，你的最终产品必须遵守关于 U-Boot 的 GPLv2 条款，因此必须提供 U-Boot 的源代码和修改。

### Linux 内核

Linux 内核也采用 GPLv2 许可证。任何在 Linux 内核用户空间运行的应用程序都不会继承其许可证，因此你可以在 Linux 上自由运行你的专有软件。然而，Linux 内核模块是 Linux 内核的一部分，因此必须遵守 GPLv2 许可证。此外，你的最终产品必须发布 Linux 内核源代码和修改，包括在你的产品中运行的外部模块。

### Glibc

GNU `C`库采用**较宽松通用公共许可证**（**LGPL**），允许动态链接而无需继承许可证。因此，你的专有代码可以与`glibc`动态链接，但当然你仍然需要遵守关于`glibc`的 LGPL。不过，值得注意的是，静态链接你的应用程序会使其与 LGPL 绑定。

### BusyBox

BusyBox 也采用 GPLv2 许可证。该许可证允许不相关的软件与它一起运行，因此你的专有软件可以与 BusyBox 自由运行。如前所述，关于 BusyBox，你需要遵守 GPLv2 并分发其源代码和修改。

### Qt 框架

Qt 有三种不同的许可证，这对于开源项目来说是很常见的。你可以选择是否需要商业许可证（在这种情况下，你的专有应用程序将受到保护），LGPL 许可证（如前所述，通过允许你的应用程序与 Qt 框架动态链接，只要你遵守 LGPL 条款，也能保护你的专有软件），或者 GPLv3 许可证（它将被继承到你的应用程序中）。

### X Windows 系统

`X.Org`源代码采用宽松的 MIT 风格许可证。因此，只要声明使用情况并保留版权声明，你的专有软件可以自由使用它。

## 还有更多内容...

让我们看看如何将我们的专有许可证代码集成到 Yocto 构建系统中。在为我们的应用程序准备配方时，我们可以采取几种许可证方式：

+   将`LICENSE`标记为封闭。这是专有应用程序的常见做法。我们使用以下方式：

    ```
    LICENSE = "CLOSED"
    ```

+   将`LICENSE`标记为专有，并包括某种类型的许可协议。这通常在发布二进制文件时使用，附带某种终端用户协议，并在配方中引用。例如，`meta-fsl-arm`使用这种类型的许可证来遵守 Freescale 的终端用户许可协议。以下是一个示例：

    ```
    LICENSE = "Proprietary"
    LIC_FILES_CHKSUM = "file://EULA.txt;md5=93b784b1c11b3fffb1638498a8dde3f6"
    ```

+   提供多种许可选项，例如开源许可证和商业许可证。在这种情况下，`LICENSE` 变量用于指定开源许可证，而 `LICENSE_FLAGS` 变量用于商业许可证。一个典型的例子是 Poky 中的`gst-plugins-ugly`包：

    ```
    LICENSE = "GPLv2+ & LGPLv2.1+ & LGPLv2+"
    LICENSE_FLAGS = "commercial"
    LIC_FILES_CHKSUM = "file://COPYING;md5=a6f89e2100d9b6cdffcea4f398e37343 \ file://gst/synaesthesia/synaescope.h;beginline=1;endline=20 ;md5=99f301df7b80490c6ff8305fcc712838 \  file://tests/check/elements/xingmux.c;beginline=1;endline=2 1;md5=4c771b8af188724855cb99cadd390068 \  file://gst/mpegstream/gstmpegparse.h;beginline=1;endline=18 ;md5=ff65467b0c53cdfa98d0684c1bc240a9"
    ```

当在配方中设置了 `LICENSE_FLAGS` 变量时，除非许可证也出现在 `LICENSE_FLAGS_WHITELIST` 变量中，否则包将不会被构建，通常该变量在你的 `conf/local.conf` 文件中定义。对于前面的例子，我们需要添加：

```
LICENSE_FLAGS_WHITELIST = "commercial"
```

`LICENSE` 和 `LICENSE_FLAGS_WHITELIST` 变量可以精确匹配以进行非常狭窄的匹配，也可以像前面例子那样广泛匹配，匹配所有以 `commercial` 开头的许可证。对于狭窄匹配，包名必须附加到许可证名称上；例如，如果我们只希望将前面例子中的 `gst-plugins-ugly` 包加入白名单，而不包括其他包，我们可以使用如下方式：

```
LICENSE_FLAGS_WHITELIST = "commercial_gst-plugins-ugly"
```

## 另见

+   你应该参考具体的许可证，以完整理解它们所施加的要求。你可以在[`spdx.org/licenses/`](http://spdx.org/licenses/)找到完整的开源许可证列表及其文档。
