# 第七章：使用 Python 构建和管理 KVM 实例

在本章中，我们将讨论以下主题：

+   安装和使用 Python libvirt 库

+   使用 Python 定义 KVM 实例

+   使用 Python 启动、停止和删除 KVM 实例

+   使用 Python 检查 KVM 实例

+   使用 libvirt 和 Bottle 构建一个简单的 REST API 服务器

# 介绍

`libvirt` 库提供了一个虚拟化无关的接口，用于控制 KVM（及其他技术，如 XEN 和 LXC）实例的完整生命周期。通过 Python 绑定，我们可以定义、启动、销毁和删除虚拟机，以及 `virsh` 用户空间工具实现的任何其他功能。实际上，我们可以通过运行以下命令看到 `virsh` 命令使用了多个 libvirt 共享库：

```
root@kvm:~# ldd /usr/bin/virsh | grep libvirt
libvirt-lxc.so.0 => /usr/lib/x86_64-linux-gnu/libvirt-lxc.so.0 (0x00007fd050d88000)
libvirt-qemu.so.0 => /usr/lib/x86_64-linux-gnu/libvirt-qemu.so.0 (0x00007fd050b84000)
libvirt.so.0 => /usr/lib/x86_64-linux-gnu/libvirt.so.0 (0x00007fd050394000)
root@kvm:~#

```

Python libvirt 模块还提供了监控和报告 CPU、内存、存储和网络资源使用情况的方法，具体功能取决于所使用的虚拟化驱动程序类型。

在本章中，我们将使用 Python libvirt API 的一个小子集来定义、启动、检查和停止 KVM 实例。

要查看 Python libvirt 模块提供的函数、类和方法的完整列表，请执行：

`root@kvm:~# pydoc libvirt`

# 安装和使用 Python libvirt 库

在这个食谱中，我们将安装 Python libvirt 模块及其依赖项，创建一个新的虚拟环境，并安装用于交互计算的 `iPython` 命令行工具。

# 准备工作

在这个食谱中，我们需要以下内容：

+   一台安装并配置了 libvirt 和 QEMU 的 Ubuntu 主机

+   我们在第一章《*使用 QEMU 和 KVM 入门*》的*使用 debootstrap 安装自定义操作系统到镜像*食谱中构建的 `debian.img` 原始镜像文件

+   Python 2.7 解释器，通常由 `python2.7` 包提供

# 如何操作...

要安装 Python libvirt 模块、`iPython` 工具，并为我们的测试创建新的虚拟环境，请按照以下步骤操作：

1.  安装 Python 开发包 `pip` 和 `virtualenv`：

```
root@kvm:~# apt-get install python-pip python-dev pkg-config build-essential autoconf libvirt-dev
root@kvm:~# pip install virtualenv
Downloading/unpacking virtualenv
 Downloading virtualenv-15.1.0-py2.py3-none-any.whl (1.8MB): 1.8MB downloaded
Installing collected packages: virtualenv
Successfully installed virtualenv
Cleaning up...
root@kvm:~#

```

1.  创建一个新的 Python 虚拟环境并激活它：

```
root@kvm:~# mkdir kvm_python
root@kvm:~# virtualenv kvm_python/
New python executable in /root/kvm_python/bin/python
Installing setuptools, pip, wheel...done.
root@kvm:~# source kvm_python/bin/activate
(kvm_python) root@kvm:~# cd kvm_python/
(kvm_python) root@kvm:~/kvm_python# ls -la
total 28
drwxr-xr-x 6 root root 4096 May 9 17:28 .
drwx------ 8 root root 4096 May 9 17:28 ..
drwxr-xr-x 2 root root 4096 May 9 17:28 bin
drwxr-xr-x 2 root root 4096 May 9 17:28 include
drwxr-xr-x 3 root root 4096 May 9 17:28 lib
drwxr-xr-x 2 root root 4096 May 9 17:28 local
-rw-r--r-- 1 root root 60 May 9 17:28 pip-selfcheck.json
(kvm_python) root@kvm:~/kvm_python#

```

1.  安装 libvirt 模块：

```
(kvm_python) root@kvm:~/kvm_python# pip install libvirt-python
Collecting libvirt-python
 Using cached libvirt-python-3.3.0.tar.gz
Building wheels for collected packages: libvirt-python
 Running setup.py bdist_wheel for libvirt-python ... done
 Stored in directory: /root/.cache/pip/wheels/67/f0/5c/c939bf8fcce5387a36efca53eab34ba8e94a28f244fd1757c1
Successfully built libvirt-python
Installing collected packages: libvirt-python
Successfully installed libvirt-python-3.3.0
(kvm_python) root@kvm:~/kvm_python# pip freeze
appdirs==1.4.3
libvirt-python==3.3.0
packaging==16.8
pyparsing==2.2.0
six==1.10.0
(kvm_python) root@kvm:~/kvm_python# python --version
Python 2.7.6
(kvm_python) root@kvm:~/kvm_python#

```

1.  安装 `iPython` 并启动它：

```
(kvm_python) root@kvm:~/kvm_python# apt-get install ipython
...
(kvm_python) root@kvm:~/kvm_python# ipython
Python 2.7.6 (default, Oct 26 2016, 20:30:19)
Type "copyright", "credits" or "license" for more information.

IPython 1.2.1 -- An enhanced Interactive Python.
? -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help -> Python's own help system.
object? -> Details about 'object', use 'object??' for extra details.

In [1]:

```

# 它是如何工作的...

我们从第 1 步开始安装依赖包。由于我们将使用 Python 虚拟环境进行开发，因此也会安装 `virtualenv` 包。Python libvirt 模块将在虚拟环境中使用 `pip` 包管理器安装，因为我们不想让宿主机安装额外的包。

在第 2 步中，我们创建并激活一个新的 Python 虚拟环境，在第 3 步中安装 Python libvirt 模块。

最后，在第 4 步中，我们安装并启动 `iPython` 开发工具，整个章节中我们将使用该工具。

# 使用 Python 定义 KVM 实例

在本食谱中，我们将使用在前一个食谱中安装的 Python libvirt 模块来定义一个新的 KVM 实例。我们将使用虚拟环境和 `iPython` 开发工具进行以下示例。

# 准备就绪

对于这个食谱，我们将需要以下内容：

+   一台已安装并配置了 libvirt 和 QEMU 的 Ubuntu 主机

+   我们在 第一章 *使用 debootstrap 安装自定义操作系统* 中构建的 `debian.img` 原始镜像文件，*开始使用 QEMU 和 KVM*

+   Python 2.7、`iPython` 工具和我们在本章 *安装并使用 Python libvirt 库* 中创建的虚拟环境

# 如何执行...

要使用 Python libvirt 模块定义新的 KVM 实例，请遵循以下说明：

1.  在 iPython 解释器中，导入 `libvirt` 模块：

```
In [1]: import libvirt

In [2]:

```

1.  创建实例定义字符串：

```
In [2]: xmlconfig = """
<domain type='kvm' id='1'>
 <name>kvm_python</name>
 <memory unit='KiB'>1048576</memory>
 <currentMemory unit='KiB'>1048576</currentMemory>
 <vcpu placement='static'>1</vcpu>
 <resource>
   <partition>/machine</partition>
 </resource>
 <os>
   <type arch='x86_64' machine='pc-i440fx-trusty'>hvm</type>
   <boot dev='hd'/>
 </os>
 <features>
   <acpi/>
   <apic/>
   <pae/>
 </features>
 <clock offset='utc'/>
 <on_poweroff>destroy</on_poweroff>
 <on_reboot>restart</on_reboot>
 <on_crash>restart</on_crash>
 <devices>
   <emulator>/usr/bin/qemu-system-x86_64</emulator>
   <disk type='file' device='disk'>
     <driver name='qemu' type='raw'/>
     <source file='/tmp/debian.img'/>
     <backingStore/>
     <target dev='hda' bus='ide'/>
     <alias name='ide0-0-0'/>
     <address type='drive' controller='0' bus='0' target='0' unit='0'/>
   </disk>
   <controller type='usb' index='0'>
     <alias name='usb'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x2'/>
   </controller>
   <controller type='pci' index='0' model='pci-root'>
     <alias name='pci.0'/>
   </controller>
   <controller type='ide' index='0'>
     <alias name='ide'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
   </controller>
   <interface type='network'>
     <mac address='52:54:00:da:02:01'/>
     <source network='default' bridge='virbr0'/>
     <target dev='vnet0'/>
     <model type='rtl8139'/>
     <alias name='net0'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
   </interface>
   <serial type='pty'>
     <source path='/dev/pts/5'/>
     <target port='0'/>
     <alias name='serial0'/>
   </serial>
   <console type='pty' tty='/dev/pts/5'>
     <source path='/dev/pts/5'/>
     <target type='serial' port='0'/>
     <alias name='serial0'/>
   </console>
   <input type='mouse' bus='ps2'/>
   <input type='keyboard' bus='ps2'/>
   <graphics type='vnc' port='5900' autoport='yes' listen='0.0.0.0'>
     <listen type='address' address='0.0.0.0'/>
   </graphics>
   <video>
     <model type='cirrus' vram='16384' heads='1'/>
     <alias name='video0'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
   </video>
   <memballoon model='virtio'>
     <alias name='balloon0'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
   </memballoon>
 </devices>
</domain>
"""

In [3]:

```

1.  获取到虚拟化主机的连接：

```
In [3]: conn = libvirt.open('qemu:///system')

In [4]:

```

1.  定义新实例，但不启动它：

```
In [4]: instance = conn.defineXML(xmlconfig)

In [5]:

```

1.  列出主机上已定义的实例：

```
In [5]: instances = conn.listDefinedDomains()

In [6]: print 'Defined instances: {}'.format(instances)
Defined instances: ['kvm_python']

In [7]:

```

1.  使用 `virsh` 命令确保实例已经定义：

```
(kvm_python) root@kvm:~/kvm_python# virsh list --all
 Id  Name        State
----------------------------------------------------
 -   kvm_python  shut off

(kvm_python) root@kvm:~/kvm_python#

```

# 它是如何工作的...

在本食谱中，我们使用的是在 第一章 *开始使用 QEMU 和 KVM* 中创建的现有原始 Debian 镜像，用于定义 KVM 实例。

在步骤 1 中，我们导入了 libvirt 包，并定义了新的 KVM 实例。在步骤 2 中，我们将 XML 格式的字符串赋值给 `xmlconfig` 变量。注意，该定义包含了新实例的名称和镜像文件的位置。

在步骤 3 中，我们获得了连接对象，并将其赋值给 `conn` 变量。现在，我们可以使用可用方法来定义 KVM 客户机。

要列出 iPython 中某个对象的所有可用方法，请输入变量名并后跟 `*.*`，然后按 *Tab* 键两次：

在 [7] 中：conn.

显示所有 117 个可能性？（y 或 n）

`conn.allocPages`                                                                     `conn.getURI` `conn.nodeDeviceLookupByName`

`conn.baselineCPU`                                                               `conn.getVersion` `conn.nodeDeviceLookupSCSIHostByWWN`

`conn.c_pointer`                                                 `conn.interfaceDefineXML` `conn.numOfDefinedDomains`        `conn.interfaceLookupByMACString` `conn.numOfDefinedInterfaces`

...

在 [7] 中：conn.

要获取某个方法的帮助，可以在方法后附加问号字符：

`在 [7] 中：conn.defineXML?`

`类型：实例方法`

`字符串形式：<绑定方法 virConnect.defineXML，位于 <libvirt.virConnect 对象，地址 0x7fc5e57dc350>>`

`文件：/root/kvm_python/lib/python2.7/site-packages/libvirt.py`

`定义：conn.defineXML(self, xml)`

`文档字符串：`

`定义一个域，但不启动它。`

`此定义是持久的，直到显式取消定义为止`

`virDomainUndefine()。该域的先前定义将会被`

`如果已经存在，则会被覆盖。`

某些虚拟机管理程序可能会阻止此操作，如果当前有

在一个临时域上进行块复制操作，且其 ID 与

如果正在定义域，使用`virDomainBlockJobAbort()`来

首先停止块复制操作。

在资源释放后应使用`virDomainFree`来释放资源

不再需要该域对象。

在[7]：

在第 4 步中，我们在`libvirt.virConnect`连接对象上使用`defineXML()`方法，传入 XML 定义字符串，并将其赋值给实例变量。我们可以通过运行以下命令查看新对象的类型：

```
In [7]: type(instance)
Out[7]: libvirt.virDomain

In [8]:

```

在第 5 步中，我们通过使用`listDefinedDomains()`方法列出主机上的已定义实例，并在第 6 步中使用`virsh`命令确认结果。

# 还有更多内容...

让我们为前面的 Python 代码添加一些简单的错误检查，并将所有内容写入一个新文件。在后续的示例中，我们将继续向这个文件添加内容：

```
(kvm_python) root@kvm:~/kvm_python# cat kvm.py
import libvirt

xmlconfig = """
<domain type='kvm' id='1'>
 <name>kvm_python</name>
 <memory unit='KiB'>1048576</memory>
 <currentMemory unit='KiB'>1048576</currentMemory>
 <vcpu placement='static'>1</vcpu>
 <resource>
   <partition>/machine</partition>
 </resource>
 <os>
   <type arch='x86_64' machine='pc-i440fx-trusty'>hvm</type>
   <boot dev='hd'/>
 </os>
 <features>
   <acpi/>
   <apic/>
   <pae/>
 </features>
 <clock offset='utc'/>
 <on_poweroff>destroy</on_poweroff>
 <on_reboot>restart</on_reboot>
 <on_crash>restart</on_crash>
 <devices>
   <emulator>/usr/bin/qemu-system-x86_64</emulator>
   <disk type='file' device='disk'>
     <driver name='qemu' type='raw'/>
     <source file='/tmp/debian.img'/>
     <backingStore/>
     <target dev='hda' bus='ide'/>
     <alias name='ide0-0-0'/>
     <address type='drive' controller='0' bus='0' target='0' unit='0'/>
   </disk>
   <controller type='usb' index='0'>
     <alias name='usb'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x2'/>
   </controller>
   <controller type='pci' index='0' model='pci-root'>
     <alias name='pci.0'/>
   </controller>
   <controller type='ide' index='0'>
     <alias name='ide'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
   </controller>
   <interface type='network'>
     <mac address='52:54:00:da:02:01'/>
     <source network='default' bridge='virbr0'/>
     <target dev='vnet0'/>
     <model type='rtl8139'/>
     <alias name='net0'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
   </interface>
   <serial type='pty'>
     <source path='/dev/pts/5'/>
     <target port='0'/>
     <alias name='serial0'/>
   </serial>
   <console type='pty' tty='/dev/pts/5'>
     <source path='/dev/pts/5'/>
     <target type='serial' port='0'/>
     <alias name='serial0'/>
   </console>
   <input type='mouse' bus='ps2'/>
   <input type='keyboard' bus='ps2'/>
   <graphics type='vnc' port='5900' autoport='yes' listen='0.0.0.0'>
     <listen type='address' address='0.0.0.0'/>
   </graphics>
   <video>
     <model type='cirrus' vram='16384' heads='1'/>
     <alias name='video0'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
   </video>
   <memballoon model='virtio'>
     <alias name='balloon0'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
   </memballoon>
 </devices>
</domain>
"""

conn = libvirt.open('qemu:///system')
if conn == None:
  print 'Failed to connecto to the hypervizor'
  exit(1)

instance = conn.defineXML(xmlconfig)
if instance == None:
  print 'Failed to define the instance'
  exit(1)

instances = conn.listDefinedDomains()
print 'Defined instances: {}'.format(instances)

conn.close()

(kvm_python) root@kvm:~/kvm_python#

```

要执行脚本，请确保首先未定义`python_kmv`实例，然后运行：

```
(kvm_python) root@kvm:~/kvm_python# python kvm.py
Defined instances: ['kvm_python']
(kvm_python) root@kvm:~/kvm_python#

```

# 使用 Python 启动、停止和删除 KVM 实例

在这个示例中，我们将使用在上一示例中定义的实例对象上的`create()`方法启动它，并使用`destroy()`方法停止它。

要获取有关`create()`方法的更多信息，请运行：

```
In [1]: instance.create?
Type: instancemethod
String Form:<bound method virDomain.create of <libvirt.virDomain object at 0x7fc5d9b97d90>>
File: /root/kvm_python/lib/python2.7/site-packages/libvirt.py
Definition: instance.create(self)
Docstring:
Launch a defined domain. If the call succeeds the domain moves from the
defined to the running domains pools. The domain will be paused only
if restoring from managed state created from a paused domain. For more
control, see virDomainCreateWithFlags().

In [2]:

```

# 准备就绪

对于这个示例，我们将需要以下内容：

+   一台安装并配置了 libvirt 和 QEMU 的 Ubuntu 主机

+   我们在第一章中的*使用 debootstrap 安装自定义操作系统的镜像*示例中构建的`debian.img`原始镜像文件，*开始使用 QEMU 和 KVM*

+   Python 2.7、iPython 工具和我们在本章的*安装和使用 Python libvirt 库*示例中创建的虚拟环境

+   我们在本章的*使用 Python 定义 KVM 实例*示例中创建的实例对象

# 如何做...

要启动之前定义的 KVM 实例，获取其状态并最终停止它，可以使用以下 Python 代码：

1.  在实例对象上调用`create()`方法：

```
In [1]: instance.create()
Out[1]: 0

In [2]:

```

1.  通过调用实例对象上的`isActive()`方法，确保实例处于运行状态：

```
In [2]: instance.isActive()
Out[2]: 1

In [3]:

```

1.  检查从主机操作系统获取的 KVM 实例状态：

```
(kvm_python) root@kvm:~/kvm_python# virsh list --all
 Id   Name        State
----------------------------------------------------
 5    kvm_python  running

(kvm_python) root@kvm:~/kvm_python#

```

1.  使用`destroy()`方法停止实例：

```
In [3]: instance.destroy()
Out[3]: 0

In [4]:

```

1.  确保实例已被销毁：

```
In [4]: instance.isActive()
Out[4]: 0

In [5]:

```

1.  删除实例并列出所有已定义的来宾：

```
In [5]: instance.undefine()
Out[5]: 0

In [6]: conn.listDefinedDomains()
Out[6]: []

In [7]:

```

# 它是如何工作的...

在第 1 步中，我们调用`create()`方法启动定义的实例。如果成功，来宾系统将从关闭状态过渡到运行状态，正如我们在第 3 步命令的输出中看到的那样。在第 2 步中，我们使用`isActive()`方法来检查实例的状态。输出为 1 表示实例正在运行。

在第 4 步中，我们使用`destroy()`方法停止实例，并在第 5 步中进行确认。

最后在第 6 步中，我们使用`undefine()`方法删除实例，并通过`listDefinedDomains()`调用列出所有已定义的实例。

# 还有更多内容...

让我们将新代码添加到我们在*使用 Python 定义 KVM 实例*配方中开始的 Python 脚本中。更新后的脚本应该如下所示：

```
(kvm_python) root@kvm:~/kvm_python# cat kvm.py
import libvirt
import time

xmlconfig = """
<domain type='kvm' id='1'>
 <name>kvm_python</name>
 <memory unit='KiB'>1048576</memory>
 <currentMemory unit='KiB'>1048576</currentMemory>
 <vcpu placement='static'>1</vcpu>
 <resource>
 <partition>/machine</partition>
 </resource>
 <os>
 <type arch='x86_64' machine='pc-i440fx-trusty'>hvm</type>
 <boot dev='hd'/>
 </os>
 <features>
 <acpi/>
 <apic/>
 <pae/>
 </features>
 <clock offset='utc'/>
 <on_poweroff>destroy</on_poweroff>
 <on_reboot>restart</on_reboot>
 <on_crash>restart</on_crash>
 <devices>
 <emulator>/usr/bin/qemu-system-x86_64</emulator>
 <disk type='file' device='disk'>
 <driver name='qemu' type='raw'/>
 <source file='/tmp/debian.img'/>
<backingStore/>
 <target dev='hda' bus='ide'/>
 <alias name='ide0-0-0'/>
 <address type='drive' controller='0' bus='0' target='0' unit='0'/>
 </disk>
 <controller type='usb' index='0'>
 <alias name='usb'/>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x2'/>
 </controller>
 <controller type='pci' index='0' model='pci-root'>
 <alias name='pci.0'/>
 </controller>
 <controller type='ide' index='0'>
 <alias name='ide'/>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
 </controller>
 <interface type='network'>
 <mac address='52:54:00:da:02:01'/>
 <source network='default' bridge='virbr0'/>
 <target dev='vnet0'/>
 <model type='rtl8139'/>
 <alias name='net0'/>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
 </interface>
 <serial type='pty'>
 <source path='/dev/pts/5'/>
 <target port='0'/>
 <alias name='serial0'/>
 </serial>
 <console type='pty' tty='/dev/pts/5'>
 <source path='/dev/pts/5'/>
 <target type='serial' port='0'/>
 <alias name='serial0'/>
 </console>
 <input type='mouse' bus='ps2'/>
 <input type='keyboard' bus='ps2'/>
 <graphics type='vnc' port='5900' autoport='yes' listen='0.0.0.0'>
 <listen type='address' address='0.0.0.0'/>
 </graphics>
 <video>
 <model type='cirrus' vram='16384' heads='1'/>
 <alias name='video0'/>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
 </video>
 <memballoon model='virtio'>
 <alias name='balloon0'/>
<address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
 </memballoon>
 </devices>
</domain>
"""

conn = libvirt.open('qemu:///system')
if conn == None:
 print 'Failed to connecto to the hypervizor'
 exit(1)

instance = conn.defineXML(xmlconfig)
if instance == None:
 print 'Failed to define the instance'
 exit(1)

instances = conn.listDefinedDomains()
print 'Defined instances: {}'.format(instances)

time.sleep(5)

if instance.create() < 0:
 print 'Failed to start the {} instance'.format(instance.name())
exit(1)

if instance.isActive():
 print 'The instance {} is running'.format(instance.name())
else:
 print 'The instance {} is not running'.format(instance.name())

time.sleep(5)

if instance.destroy() < 0:
 print 'Failed to stop the {} instance'.format(instance.name())
 exit(1)
else:
 print 'The instance {} has been destroyed'.format(instance.name())

if instance.undefine() < 0:
 print 'Failed to remove the {} instance'.format(instance.name())
 exit(1)
else:
 print 'The instance {} has been undefined'.format(instance.name())

conn.close() 
(kvm_python) root@kvm:~/kvm_python#

```

启动脚本时，应该定义一个新的实例，启动它，停止它，最后删除它：

```
(kvm_python) root@kvm:~/kvm_python# python kvm.py
Defined instances: ['kvm1', 'kvm_python']
The instance kvm_python is running
The instance kvm_python has been destroyed
The instance kvm_python has been undefined
(kvm_python) root@kvm:~/kvm_python#

```

在前面的脚本中，我们使用了`instance.name()`方法来获取 KVM 客户机的名称并打印它。我们还通过调用`conn.close()`关闭与虚拟化管理程序的连接，从而进行清理。

# 使用 Python 检查 KVM 实例

在这个配方中，我们将收集实例信息，使用`libvirt.virDomain`类中的方法。

欲了解更多关于 libvirt Python API 的信息，请参考官方文档：[`libvirt.org/docs/libvirt-appdev-guide-python/en-US/html/index.html`](http://libvirt.org/docs/libvirt-appdev-guide-python/en-US/html/index.html)。

# 准备工作

对于这个配方，我们将需要以下内容：

+   一个安装并配置了 libvirt 和 QEMU 的 Ubuntu 主机

+   我们在第一章的*使用 debootstrap 安装自定义操作系统到镜像*配方中构建的`debian.img`原始镜像文件，*开始使用 QEMU 和 KVM*

+   Python 2.7、`iPython`工具和我们在本章的*安装并使用 Python libvirt 库*配方中创建的虚拟环境

+   我们在本章的*使用 Python 定义 KVM 实例*配方中创建的`instance`对象，代表 KVM 客户机

# 如何操作...

要收集关于运行实例的 CPU、内存和状态信息，可以使用以下 Python 方法：

1.  获取实例的名称：

```
In [1]: instance.name()
Out[1]: 'kvm_python'

In [2]:

```

1.  确保实例正在运行：

```
In [2]: instance.isActive()
Out[2]: 1

In [3]:

```

1.  收集 KVM 实例的资源统计信息：

```
In [3]: instance.info()
Out[3]: [1, 1048576L, 1048576L, 1, 10910000000L]

In [4]:

```

1.  获取分配给实例的最大物理内存：

```
In [4]: instance.maxMemory()
Out[4]: 1048576L

In [5]:

```

1.  提取实例的 CPU 统计信息：

```
In [5]: instance.getCPUStats(1)
Out[5]:
[{'cpu_time': 10911545901L,
 'system_time': 1760000000L,
 'user_time': 1560000000L}]

In [6]:

```

1.  检查虚拟机是否使用了硬件加速：

```
In [6]: instance.OSType()
Out[6]: 'hvm'

In [7]:

```

1.  收集实例状态：

```
In [82]: state, reason = instance.state()

In [83]: if state == libvirt.VIR_DOMAIN_NOSTATE:
 ....:           print('The state is nostate')
 ....: elif state == libvirt.VIR_DOMAIN_RUNNING:
 ....:           print('The state is running')
 ....: elif state == libvirt.VIR_DOMAIN_BLOCKED:
 ....:           print('The state is blocked')
 ....: elif state == libvirt.VIR_DOMAIN_PAUSED:
 ....:           print('The state is paused')
 ....: elif state == libvirt.VIR_DOMAIN_SHUTDOWN:
 ....:           print('The state is shutdown')
 ....: elif state == libvirt.VIR_DOMAIN_SHUTOFF:
 ....:           print('The state is shutoff')
 ....: elif state == libvirt.VIR_DOMAIN_CRASHED:
 ....:           print('The state is crashed')
 ....: elif state == libvirt.VIR_DOMAIN_PMSUSPENDED:
 ....:           print('The state is suspended')
 ....: else:
 ....:           print('The state is unknown')
 ....:
The state is running

In [84]:

```

# 它是如何工作的...

在本配方中，我们使用了`libvirt.virDomain`类中的一些新方法。让我们更详细地了解它们的作用，然后将它们添加到我们在*使用 Python 定义 KVM 实例*配方中开始的简单`kvm.py` Python 脚本中。

在第 1 步和第 2 步中，我们获取了 KVM 实例的名称，并确保它处于运行状态。

在第 3 步中，我们收集了以下实例信息，并以 Python 列表的形式返回：

+   **state**：实例的状态，如在[`libvirt.org/html/libvirt-libvirt-domain.html#virDomainState`](https://libvirt.org/html/libvirt-libvirt-domain.html#virDomainState)中的*virDomainState*枚举类型中定义

+   **maxMemory**：客户机使用的最大内存

+   **memory**：实例当前使用的内存量

+   **nbVirtCPU**：分配的虚拟 CPU 数量

+   **cpuTime**：实例使用的时间（以纳秒为单位）

在第 4 步中，我们收集了分配给实例的内存。注意它如何与第 3 步函数的输出匹配。

在第 5 步，我们收集关于来宾实例 CPU 的信息。我们可以看到 CPU、系统和用户时间。

第 6 步中，`OSType()` 方法的 `hvm` 输出表示来宾操作系统设计为在裸机上运行，要求完全虚拟化，如 KVM。

在本食谱的最后一步，我们调用 `state()` 方法以返回当前实例状态。

# 还有更多...

让我们通过一个完整的示例脚本结束这一章，脚本包含了我们至今使用的所有方法：

```
(kvm_python) root@kvm:~/kvm_python# cat kvm.py
import libvirt
import time

def main():

 xmlconfig = """
 <domain type='kvm' id='1'>
 <name>kvm_python</name>
 <memory unit='KiB'>1048576</memory>
 <currentMemory unit='KiB'>1048576</currentMemory>
 <vcpu placement='static'>1</vcpu>
 <resource>
 <partition>/machine</partition>
 </resource>
 <os>
 <type arch='x86_64' machine='pc-i440fx-trusty'>hvm</type>
 <boot dev='hd'/>
 </os>
 <features>
<acpi/>
 <apic/>
 <pae/>
 </features>
 <clock offset='utc'/>
 <on_poweroff>destroy</on_poweroff>
 <on_reboot>restart</on_reboot>
 <on_crash>restart</on_crash>
 <devices>
 <emulator>/usr/bin/qemu-system-x86_64</emulator>
 <disk type='file' device='disk'>
 <driver name='qemu' type='raw'/>
 <source file='/tmp/debian.img'/>
 <backingStore/>
 <target dev='hda' bus='ide'/>
 <alias name='ide0-0-0'/>
 <address type='drive' controller='0' bus='0' target='0' unit='0'/>
 </disk>
 <controller type='usb' index='0'>
 <alias name='usb'/>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x2'/>
 </controller>
 <controller type='pci' index='0' model='pci-root'>
 <alias name='pci.0'/>
 </controller>
 <controller type='ide' index='0'>
 <alias name='ide'/>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
 </controller>
 <interface type='network'>
 <mac address='52:54:00:da:02:01'/>
 <source network='default' bridge='virbr0'/>
 <target dev='vnet0'/>
 <model type='rtl8139'/>
 <alias name='net0'/>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
 </interface>
 <serial type='pty'>
 <source path='/dev/pts/5'/>
 <target port='0'/>
 <alias name='serial0'/>
 </serial>
 <console type='pty' tty='/dev/pts/5'>
 <source path='/dev/pts/5'/>
 <target type='serial' port='0'/>
 <alias name='serial0'/>
 </console>
 <input type='mouse' bus='ps2'/>
 <input type='keyboard' bus='ps2'/>
 <graphics type='vnc' port='5900' autoport='yes' listen='0.0.0.0'>
 <listen type='address' address='0.0.0.0'/>
 </graphics>
 <video>
 <model type='cirrus' vram='16384' heads='1'/>
 <alias name='video0'/>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
 </video>
 <memballoon model='virtio'>
 <alias name='balloon0'/>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
 </memballoon>
 </devices>
 </domain>
 """

 conn = libvirt.open('qemu:///system')
 if conn == None:
 print 'Failed to connecto to the hypervizor'
exit(1)

 instance = conn.defineXML(xmlconfig)
 if instance == None:
 print 'Failed to define the instance'
 exit(1)

 instances = conn.listDefinedDomains()
 print 'Defined instances: {}'.format(instances)

 time.sleep(5)

 if instance.create() < 0:
 print 'Failed to start the {} instance'.format(instance.name())
 exit(1)

 if instance.isActive():
 print 'The instance {} is running'.format(instance.name())
 else:
 print 'The instance {} is not running'.format(instance.name())

 print 'The instance state, max memory, current memory, CPUs and time is {}'.format(instance.info())

 print 'The CPU, system and user times are {}'.format(instance.getCPUStats(1))

print 'The OS type for the {} instance is {}'.format(instance.name(), instance.OSType())

 time.sleep(5)

 if instance.destroy() < 0:
 print 'Failed to stop the {} instance'.format(instance.name())
 exit(1)
 else:
 print 'The instance {} has been destroyed'.format(instance.name())

 if instance.undefine() < 0:
 print 'Failed to remove the {} instance'.format(instance.name())
 exit(1)
 else:
 print 'The instance {} has been undefined'.format(instance.name())

 conn.close()

if __name__ == "__main__":
 main()

(kvm_python) root@kvm:~/kvm_python#

```

执行后将提供以下输出，假设 `kvm_python` 实例已先被取消定义：

```
(kvm_python) root@kvm:~/kvm_python# python kvm.py
Defined instances: ['kvm_python']
The instance kvm_python is running
The instance state, max memory, current memory, CPUs and time is [1, 1048576L, 1048576L, 1, 40000000L]
The CPU, system and user times are [{'cpu_time': 42349077L, 'system_time': 0L, 'user_time': 30000000L}]
The OS type for the kvm_python instance is hvm
The instance kvm_python has been destroyed
The instance kvm_python has been undefined
(kvm_python) root@kvm:~/kvm_python# 

```

# 使用 libvirt 和 bottle 构建一个简单的 REST API 服务器

在本食谱中，我们将使用之前食谱中提到的所有 libvirt 方法，构建一个简单的 RESTful API 服务器，利用 Python 的 bottle 微框架。

Bottle 被描述为一个快速且简单的 **Web Server Gateway Interface**（**WSGI**）微型 Web 框架，用于 Python，作为一个单一的模块文件发布。

有关 bottle 微框架的更多信息，请访问官方网站：[`bottlepy.org/docs/dev/`](https://bottlepy.org/docs/dev/)。

我们实现的简单 API 服务器将接受以下请求：

+   **list**: `get` 方法，用于列出所有已定义的 libvirt 实例。

+   **define**: `post` 方法，用于定义一个新的 KVM 实例。我们将在 POST 请求的头部提供 XML 定义。

+   **start**: `post` 方法，用于启动实例。实例的名称将在请求头中提供。

+   **stop**: `post` 方法，用于停止一个 KVM 实例。

+   **undefine**: `post` 方法，用于删除实例。

# 准备中

对于本食谱，我们将需要以下内容：

+   一台已安装并配置好 libvirt 和 QEMU 的 Ubuntu 主机

+   我们在 *使用 debootstrap 安装自定义操作系统到镜像* 中构建的 `debian.img` 原始镜像文件，来自 第一章，*QEMU 和 KVM 入门*。

+   Python 2.7 和我们在本章 *安装并使用 Python libvirt 库* 中创建的虚拟环境

+   `curl` 命令行工具用于通过 URL 语法传输数据，通常由 curl 包提供

# 如何操作...

以下步骤描述了如何安装 bottle 模块及其 Python 编写的简单 RESTful API 服务器：

1.  安装 `bottle` 模块：

```
(kvm_python) root@kvm:~/kvm_python# pip install bottle
Collecting bottle
...
 Downloading bottle-0.12.13.tar.gz (70kB)
 100% |████████████████████████████████| 71kB 4.5MB/s
...
Successfully installed bottle-0.12.13
(kvm_python) root@kvm:~/kvm_python#

```

1.  创建一个新文件，导入 libvirt 和 bottle 模块，并编写 libvirt 连接方法：

```
(kvm_python) root@kvm:~/kvm_python# vim kvm_api.py
import libvirt
from bottle import run, request, get, post, HTTPResponse

def libvirtConnect():
 try:
 conn = libvirt.open('qemu:///system')
 except libvirt.libvirtError:
 conn = None

 return conn

```

1.  实现 `/define` API 路由和功能：

```
def defineKVMInstance(template):
  conn = libvirtConnect()

  if conn == None:
    return HTTPResponse(status=500, body='Error defining instance\n')
  else:
    try:
      conn.defineXML(template)
      return HTTPResponse(status=200, body='Instance defined\n')
    except libvirt.libvirtError:
      return HTTPResponse(status=500, body='Error defining instance\n')

@post('/define')
def build():
  template = str(request.headers.get('X-KVM-Definition'))
  status = defineKVMInstance(template)

  return status

```

1.  实现 `/undefine` API 路由和功能：

```
def undefineKVMInstance(name):
  conn = libvirtConnect()

  if conn == None:
    return HTTPResponse(status=500, body='Error undefining instance\n')
  else:
    try:
      instance = conn.lookupByName(name)
      instance.undefine()
      return HTTPResponse(status=200, body='Instance undefined\n')
    except libvirt.libvirtError:
      return HTTPResponse(status=500, body='Error undefining instance\n')

@post('/undefine')
def build():
  name = str(request.headers.get('X-KVM-Name'))
  status = undefineKVMInstance(name)

  return status

```

1.  实现 `/start` API 路由和功能：

```
def startKVMInstance(name):
  conn = libvirtConnect()

  if conn == None:
    return HTTPResponse(status=500, body='Error starting instance\n')
  else:
    try:
      instance = conn.lookupByName(name)
      instance.create()
      return HTTPResponse(status=200, body='Instance started\n')
    except libvirt.libvirtError:
      return HTTPResponse(status=500, body='Error starting instance\n')

@post('/start')
def build():
  name = str(request.headers.get('X-KVM-Name'))
  status = startKVMInstance(name)

  return status

```

1.  实现 `/stop` API 路由和功能：

```
def stopKVMInstance(name):
  conn = libvirtConnect()

  if conn == None:
    return HTTPResponse(status=500, body='Error stopping instance\n')
  else:
    try:
      instance = conn.lookupByName(name)
      instance.destroy()
      return HTTPResponse(status=200, body='Instance stopped\n')
    except libvirt.libvirtError:
      return HTTPResponse(status=500, body='Error stopping instance\n')

@post('/stop')
def build():
  name = str(request.headers.get('X-KVM-Name'))
  status = stopKVMInstance(name)

  return status

```

1.  实现 `/list` API 路由和功能：

```
def getLibvirtInstances():
  conn = libvirtConnect()

  if conn == None:
    return HTTPResponse(status=500, body='Error listing instances\n')
  else:
    try:
      instances = conn.listDefinedDomains()
      return instances
    except libvirt.libvirtError:
      return HTTPResponse(status=500, body='Error listing instances\n')

@get('/list')
def list():
  kvm_list = getLibvirtInstances()

  return "List of KVM instances: {}\n".format(kvm_list)

```

1.  执行时调用 `run()` 方法以启动 WSGI 服务器：

```
run(host='localhost', port=8080, debug=True)

```

# 它是如何工作的...

让我们更详细地看一下代码。首先，将前面的更改保存到一个文件中并执行脚本：

```
(kvm_python) root@kvm:~/kvm_python# python kvm_api.py
Bottle v0.12.13 server starting up (using WSGIRefServer())...
Listening on http://localhost:8080/
Hit Ctrl-C to quit.

```

在一个单独的终端中，定义一个新的实例，并传递以下 XML 定义作为头信息：

```
(kvm_python) root@kvm:~/kvm_python# curl -s -i -XPOST localhost:8080/define --header "X-KVM-Definition: <domain type='kvm'><name>kvm_api</name><memory unit='KiB'>1048576</memory><vcpu >1</vcpu><os><type arch='x86_64' machine='pc-i440fx-trusty'>hvm</type></os><devices><emulator>/usr/bin/qemu-system-x86_64</emulator><disk type='file' device='disk'><driver name='qemu' type='raw'/><source file='/tmp/debian.img'/><target dev='hda' bus='ide'/></disk><interface type='network'><mac address='52:54:00:da:02:01'/><source network='default' bridge='virbr0'/><target dev='vnet0'/></interface><graphics type='vnc' port='5900' autoport='yes' listen='0.0.0.0'><listen type='address' address='0.0.0.0'/></graphics></devices></domain>"
HTTP/1.0 200 OK
Date: Fri, 12 May 2017 20:29:14 GMT
Server: WSGIServer/0.1 Python/2.7.6
Content-Length: 17
Content-Type: text/html; charset=UTF-8

Instance defined
(kvm_python) root@kvm:~/kvm_python# 

```

我们正在使用在第一章*《开始使用 QEMU 和 KVM》*中创建的原始 Debian 镜像。XML 定义应该也很熟悉；我们在本章的大部分配方中都使用了它。

我们现在应该已经定义了一个新的 KVM 实例。让我们使用`/list`路由列出所有实例，并使用`virsh`命令确认：

```
(kvm_python) root@kvm:~/kvm_python# curl localhost:8080/list
List of KVM instances: ['kvm_api']
(kvm_python) root@kvm:~/kvm_python# virsh list --all
 Id   Name     State
----------------------------------------------------
 -   kvm_api   shut off

(kvm_python) root@kvm:~/kvm_python#

```

现在我们已经定义了一个实例，让我们使用`/start`路由启动它，并确保它在运行：

```
(kvm_python) root@kvm:~/kvm_python# curl -s -i -XPOST localhost:8080/start --header "X-KVM-Name: kvm_api"
HTTP/1.0 200 OK
Date: Fri, 12 May 2017 20:29:38 GMT
Server: WSGIServer/0.1 Python/2.7.6
Content-Length: 17
Content-Type: text/html; charset=UTF-8

Instance started
(kvm_python) root@kvm:~/kvm_python# virsh list --all
 Id   Name      State
----------------------------------------------------
 1    kvm_api   running

(kvm_python) root@kvm:~/kvm_python# 

```

要停止实例并完全删除它，我们使用脚本中的`/stop`和`/undefine`路由：

```
(kvm_python) root@kvm:~/kvm_python# curl -s -i -XPOST localhost:8080/stop --header "X-KVM-Name: kvm_api"
HTTP/1.0 200 OK
Date: Fri, 12 May 2017 20:29:52 GMT
Server: WSGIServer/0.1 Python/2.7.6
Content-Length: 17
Content-Type: text/html; charset=UTF-8

Instance stopped
(kvm_python) root@kvm:~/kvm_python#
(kvm_python) root@kvm:~/kvm_python# virsh list --all
 Id   Name      State
----------------------------------------------------
 -    kvm_api   shut off

(kvm_python) root@kvm:~/kvm_python#
(kvm_python) root@kvm:~/kvm_python# curl -s -i -XPOST localhost:8080/undefine --header "X-KVM-Name: kvm_api"
HTTP/1.0 200 OK
Date: Fri, 12 May 2017 20:30:09 GMT
Server: WSGIServer/0.1 Python/2.7.6
Content-Length: 19
Content-Type: text/html; charset=UTF-8

Instance undefined
(kvm_python) root@kvm:~/kvm_python#
(kvm_python) root@kvm:~/kvm_python# virsh list --all
 Id Name State
----------------------------------------------------

(kvm_python) root@kvm:~/kvm_python#

```

让我们更详细地了解代码。

在第 1 步中，我们在 Python 虚拟环境中安装了 bottle 模块。

在第 2 步中导入 libvirt 和 bottle 包后，我们定义了`libvirtConnect()`方法。我们的程序中的函数将使用它连接到虚拟机管理程序。

在第 3 步中，我们实现了`/define`路由及其功能。`@post`装饰器将以下函数的代码与 URL 路径关联。在我们的示例中，`/define`路由绑定到`build()`函数。将`/define`路由传递给 curl 命令将执行该函数，该函数又会调用`defineKVMInstance()`方法来定义实例。

我们在第 4 步、第 5 步和第 6 步中使用相同的代码模式来启动、停止和取消定义实例。

在第 7 步中，我们使用`@get`装饰器实现一个函数，用于列出主机上所有已定义的实例。

在第 8 步中，我们使用`run`类，它提供了我们用来启动内置服务器的`run()`方法。在我们的示例中，服务器将在本地主机（localhost）上监听，端口为`8080`。

正如我们之前看到的，执行脚本将在`8080`端口启动一个监听套接字，我们可以通过`curl`命令与之交互。

# 还有更多...

完整的代码实现如下：

```
import libvirt
from bottle import run, request, get, post, HTTPResponse

def libvirtConnect():
  try:
    conn = libvirt.open('qemu:///system')
  except libvirt.libvirtError:
    conn = None

  return conn

def getLibvirtInstances():
  conn = libvirtConnect()

  if conn == None:
    return HTTPResponse(status=500, body='Error listing instances\n')
  else:
    try:
      instances = conn.listDefinedDomains()
      return instances
    except libvirt.libvirtError:
      return HTTPResponse(status=500, body='Error listing instances\n')

def defineKVMInstance(template):
  conn = libvirtConnect()

  if conn == None:
    return HTTPResponse(status=500, body='Error defining instance\n')
  else:
    try:
      conn.defineXML(template)
      return HTTPResponse(status=200, body='Instance defined\n')
    except libvirt.libvirtError:
      return HTTPResponse(status=500, body='Error defining instance\n')

def undefineKVMInstance(name):
  conn = libvirtConnect()

  if conn == None:
    return HTTPResponse(status=500, body='Error undefining instance\n')
  else:
    try:
      instance = conn.lookupByName(name)
      instance.undefine()
      return HTTPResponse(status=200, body='Instance undefined\n')
    except libvirt.libvirtError:
      return HTTPResponse(status=500, body='Error undefining instance\n')

def startKVMInstance(name):
  conn = libvirtConnect()

  if conn == None:
    return HTTPResponse(status=500, body='Error starting instance\n')
  else:
    try:
      instance = conn.lookupByName(name)
      instance.create()
      return HTTPResponse(status=200, body='Instance started\n')
    except libvirt.libvirtError:
      return HTTPResponse(status=500, body='Error starting instance\n')

def stopKVMInstance(name):
  conn = libvirtConnect()

  if conn == None:
    return HTTPResponse(status=500, body='Error stopping instance\n')
  else:
    try:
      instance = conn.lookupByName(name)
      instance.destroy()
      return HTTPResponse(status=200, body='Instance stopped\n')
    except libvirt.libvirtError:
      return HTTPResponse(status=500, body='Error stopping instance\n')

@post('/define')
def build():
  template = str(request.headers.get('X-KVM-Definition'))
  status = defineKVMInstance(template)

  return status

@post('/undefine')
def build():
  name = str(request.headers.get('X-KVM-Name'))
  status = undefineKVMInstance(name)

  return status

@get('/list')
def list():
  kvm_list = getLibvirtInstances()

  return "List of KVM instances: {}\n".format(kvm_list)

@post('/start')
def build():
  name = str(request.headers.get('X-KVM-Name'))
  status = startKVMInstance(name)

  return status

@post('/stop')
def build():
  name = str(request.headers.get('X-KVM-Name'))
  status = stopKVMInstance(name)

  return status

run(host='localhost', port=8080, debug=True)

```
