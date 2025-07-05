# 第六章：为你的双足机器人添加视觉功能

现在你的双足机器人已经可以启动并移动，能够找到障碍物，并且知道如何规划路径，你现在可以让它开始自主移动了。不过，你可能希望让你的机器人跟随某种颜色或运动。

在本章中，你将学习：

+   如何将摄像头添加到你的双足机器人

+   如何将 RaspiCam 添加到你的双足机器人

+   如何安装和使用 OpenCV，一个开源的视觉软件包

+   如何让你的双足机器人跟随运动

# 在你的双足机器人上安装摄像头

拥有视觉能力对于你的双足机器人来说是一个真正的优势；你将在许多不同的应用中使用这个功能。幸运的是，添加视觉硬件和软件既简单又便宜。关于视觉硬件，有两种选择。你可以向系统中添加一个 USB 摄像头，或者添加一个专为 Raspberry Pi 设计的摄像头 RaspiCam。

## 在 Raspberry Pi 上安装 USB 摄像头

连接 USB 摄像头非常简单。只需将其插入 USB 插槽。为了确保设备已连接，输入 `lsusb`。你应该能看到如下内容：

![在 Raspberry Pi 上安装 USB 摄像头](img/B04591_06_01.jpg)

这显示了一个 Creative 摄像头，位于总线 001 设备 004: ID 041e:4095。为了确保系统将其识别为视频设备，输入 `ls /dev/v*` 命令，你应该看到类似下面的内容：

![在 Raspberry Pi 上安装 USB 摄像头](img/B04591_06_02.jpg)

`/dev/video0` 是摄像头设备。现在设备已经连接，让我们实际看看你是否可以捕捉图像和视频。有几种工具可以让你访问摄像头，但一个带有视频控制的简单程序叫做 luvcview。要安装这个程序，输入 `sudo apt-get install luvcview`。安装完成后，你需要运行它。为此，你要么需要直接连接显示器，要么能通过远程 VNC 连接访问 Raspberry Pi，例如使用 vncserver，因为显示图像需要图形界面。

一旦以这种方式连接，打开 Raspberry Pi 上的终端窗口并运行 `luvcview`。你应该看到类似以下内容：

![在 Raspberry Pi 上安装 USB 摄像头](img/B04591_06_03.jpg)

不必担心图像质量，你将会在 OpenCV 内部捕捉和处理图像，OpenCV 是一个视觉框架。

## 在 Raspberry Pi 上安装 RaspiCam

另一个在 Raspberry Pi 上查看外部世界的选择是使用 RaspiCam。安装这个摄像头稍微复杂一些；你将需要将其连接到 Raspberry Pi 上的特殊接口。下面是带有特殊连接器的摄像头图片：

![在 Raspberry Pi 上安装 RaspiCam](img/B04591_06_04.jpg)

你也可能需要为摄像头添加保护罩；其组装方式如下图所示：

![在 Raspberry Pi 上安装 RaspiCam](img/B04591_06_05.jpg)

现在你已经准备好将相机连接到树莓派。相机通过将其安装到树莓派上标有“Camera”的连接器上来连接树莓派。要查看具体操作，请参见[`www.raspberrypi.org/help/camera-module-setup/`](http://www.raspberrypi.org/help/camera-module-setup/)上的视频。

相机连接后，你需要使用`raspi-config`工具启用相机。输入`sudo raspi-config`，然后选择**启用相机**，如以下截图所示：

![在树莓派上安装 RaspiCam](img/B04591_06_06.jpg)

现在重启树莓派。如果你是从远程计算机进行开发并希望查看你的图像，你需要在计算机与树莓派之间建立一个 vncserver 连接。详情请参见第一章，*配置和编程树莓派*。要使用相机拍照，只需输入`raspistill -o image.jpg`。这将用相机拍照，并将图像存储在`image.jpg`文件中。一旦获得照片，你可以通过打开树莓派的图像查看器来查看它，方法是点击左下角的**菜单**图标，然后选择**附件**，接着选择**图像查看器**，如以下截图所示：

![在树莓派上安装 RaspiCam](img/B04591_06_07.jpg)

打开**image.jpg**文件，你应该能看到你拍摄的照片：

![在树莓派上安装 RaspiCam](img/B04591_06_08.jpg)

在你能使用树莓派相机访问 OpenCV 之前，你需要做两件事。首先，你需要添加一个 Python 库，叫做`picamera`。要获取此库和所需的其他库，输入`sudo apt-get install python-picamera python3-picamera python-rpi.gpio`。其次，你需要输入`sudo modprobe bcm2835-v4l2`。现在，树莓派相机可以在下一节的 OpenCV 示例中使用。

# 下载并安装 OpenCV——一个功能齐全的视觉库

现在相机已经连接好，你可以开始使用开源社区提供的一些令人惊叹的功能。打开终端窗口并输入以下命令：

1.  `sudo apt-get update`：你将要下载一些新的软件包，所以最好确保所有内容都是最新的。

1.  `sudo apt-get install build-essential`：虽然你可能之前已经执行过此步骤，但这个库对于构建 OpenCV 是必不可少的。

1.  `sudo apt-get install libavformat-dev`：此库提供编码和解码音频及视频流的功能。

1.  `sudo apt-get install ffmpeg`：此库提供转码音频和视频流的功能。

1.  `sudo apt-get install libcv2.4 libcvaux2.4 libhighgui2.4`：此命令展示了基本的 OpenCV 库。注意命令中的数字。随着 OpenCV 新版本的发布，这个数字几乎肯定会发生变化。如果 2.4 版本不起作用，可以尝试 3.0 版本或在 Google 上搜索 OpenCV 的最新版本。

1.  `sudo apt-get install python-opencv`：这是 OpenCV 所需的 Python 开发工具包，因为你将使用 Python。

1.  `sudo apt-get install opencv-doc`：这个命令会展示 OpenCV 的文档，以防你需要它。

1.  `sudo apt-get install libcv-dev`：这个命令展示了编译 OpenCV 所需的头文件和静态库。

1.  `sudo apt-get install libcvaux-dev`：这个命令展示了更多用于编译 OpenCV 的开发工具。

1.  `sudo apt-get install libhighgui-dev`：这是另一个提供头文件和静态库以编译 OpenCV 的软件包。

1.  现在输入`cp -r /usr/share/doc/opencv-doc/examples /home/pi/`。这将把所有示例复制到你的主目录。

    现在 OpenCV 已安装，你可以尝试其中一个示例。进入`/home/pi/examples/python`目录。如果你输入`ls`，你会看到一个名为`camera.py`的文件。这个文件包含了最基本的代码，用于捕捉并显示一系列图像。在运行代码之前，使用`cp camera.py myCamera.py`命令复制一份文件。然后，编辑文件使其如下所示：

    ![下载并安装 OpenCV——一个功能完备的视觉库](img/B04591_06_09.jpg)

    你将添加的两行代码是带有`cv.SetCaptureProperty`的两行；它们将把图像的分辨率设置为 360 x 240。要运行这个程序，你需要连接显示器和键盘到树莓派，或者使用 vncviewer。当你运行代码时，你应该能看到显示的窗口，如下图所示：

    ![下载并安装 OpenCV——一个功能完备的视觉库](img/B04591_06_10.jpg)

    如果你使用的是 RaspiCam 但没有看到图像，你需要运行`sudo modprobe bcm2835-v4l2`命令。现在你可以看到外部世界了！

    你可能想调整分辨率，以找到最适合你应用的设置。更大的图像很棒——它们能让你更详细地看到世界——但也会消耗更多的处理能力。当你实际要求系统进行一些真实的图像处理时，你会更多地调整这一点。如果你打算使用 vncserver 来了解系统性能，要小心，因为这会显著减慢更新率。一个宽高是原来两倍的图像（宽度/高度）将涉及四倍的处理量。现在，你可以利用这个功能来完成一些令人印象深刻的任务。

# 边缘检测与 OpenCv

幸运的是，OpenCV Python 示例集中有一个名为`edge.py`的程序。以下是该文件的内容（去掉了空行）：

![边缘检测与 OpenCv](img/B04591_06_11.jpg)

该程序使用 OpenCV 实现的 Canny 图像检测算法来查找图像中的边缘。更多关于 Canny 边缘算法的内容，请参考[`dasl.mem.drexel.edu/alumni/bGreen/www.pages.drexel.edu/_weg22/can_tut.html`](http://dasl.mem.drexel.edu/alumni/bGreen/www.pages.drexel.edu/_weg22/can_tut.html) 或 [`opencv-python-tutroals.readthedocs.org/en/latest/py_tutorials/py_imgproc/py_canny/py_canny.html`](http://opencv-python-tutroals.readthedocs.org/en/latest/py_tutorials/py_imgproc/py_canny/py_canny.html)。你之前已经捕获了一张图片；你可以使用这个程序查看边缘，并且还可以看到设置不同的阈值如何展示更多或更少的边缘。使用之前捕获的图像运行程序，你将看到以下内容：

![边缘检测与 OpenCv](img/B04591_06_12.jpg)

你会注意到顶部有一个阈值滑块设置。如果你将这个阈值调高，它会找到更少的边缘——那些具有较大阈值的边缘。设置为 30 时的图片如下所示：

![边缘检测与 OpenCv](img/B04591_06_13.jpg)

现在你可以看到这个过程如何转换为一张空白地板和障碍物的图像。以下是带有可能障碍物的图像：

![边缘检测与 OpenCv](img/B04591_06_14.jpg)

你可以根据像素和相机的位置校准物体的距离。

# 颜色与运动检测

OpenCV 和你的网络摄像头还可以跟踪彩色物体。如果你希望你的双足机器人跟踪一个彩色物体，这将非常有用。OpenCV 通过提供一些高级库，使得这个任务变得非常简单。为了实现这一点，你需要编辑一个文件，它的内容看起来应该类似于以下屏幕截图所示：

![颜色与运动检测](img/B04591_06_15.jpg)

让我们具体看一下使得能够隔离彩色球的代码：

+   `hue_img = cv.CvtColor(frame, cv.CV_BGR2HSV):` 这一行创建了一张新的图像，将图像按**色相**（颜色）、**饱和度**和**亮度**（**HSV**）的值存储，而不是原始图像的**红色**、**绿色**和**蓝色**（**RGB**）像素值。转换为 HSV 让我们的处理更加关注颜色，而不是照射到它上的光线量。

+   `threshold_img = cv.InRangeS(hue_img, low_range, high_range`): `low_range, high_range` 参数决定了颜色范围。在这种情况下，它是一个橙色的球，因此你想检测橙色。关于使用色相来指定颜色的好教程，请参考[`www.tomjewett.com/colors/hsb.html`](http://www.tomjewett.com/colors/hsb.html)。此外，[`www.shervinemami.info/colorConversion.html`](http://www.shervinemami.info/colorConversion.html) 包含一个程序，你可以使用它来通过选择特定的颜色来确定你的值。

    运行程序。如果你看到一张单独的黑色图像，移动这个窗口，你将看到原始图像窗口被露出。现在，拿起你的目标物体（在本例中是一个橙色的乒乓球），并把它移动到画面中。你应该会看到类似于以下截图的内容：

    ![颜色和运动检测](img/B04591_06_16.jpg)

    请注意，我们在阈值图像中显示的白色像素，标示出球的位置。你可以添加更多 OpenCV 代码来显示球的实际位置。在我们原始图像中的球的位置，你实际上可以在球周围画一个矩形作为指示。编辑文件，使其如下所示：

    ![颜色和运动检测](img/B04591_06_17.jpg)

添加的线条看起来像这样：

+   `hue_image = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV):` 这行代码将捕获的 RGB 图像转换为色相图像。在捕获真实世界图像时，色相处理起来更加方便；有关详细信息，请参阅[`www.bogotobogo.com/python/OpenCV_Python/python_opencv3_Changing_ColorSpaces_RGB_HSV_HLS.php`](http://www.bogotobogo.com/python/OpenCV_Python/python_opencv3_Changing_ColorSpaces_RGB_HSV_HLS.php)。

+   `threshold_img = cv2.inRange(hue_image, low_range, high_range):` 这会创建一张新图像，只包含那些位于`low_range`和`high_range`元组之间的像素。

+   `contour, hierarchy = cv2.findContours(threshold_img, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE):` 这段代码查找`threshold_img`图像中的轮廓，即一组相似的像素。

+   `center = contour[0]:` 这识别了第一个轮廓。

+   `moment = cv2.moments(center):` 这段代码计算这一组像素的矩。

+   `(x,y),radius = cv2.minEnclosingCircle(center):` 这会给出包围这组像素的最小圆的*x*和*y*位置以及半径。

+   `center = (int(x),int(y)):` 找到*x*和*y*位置的中心。

+   `radius = int(radius):` 圆的整数半径。

+   `img = cv2.circle(frame,center,radius,(0,255,0),2):` 在图像上画一个圆。

现在代码已经准备好，你可以运行它。你应该会看到类似于以下截图的内容：

![颜色和运动检测](img/B04591_06_18.jpg)

现在你可以跟踪你的物体了。你可以通过更改`low_range`和`high_range`元组来修改颜色。你还知道了物体的位置，可以利用这个位置为机器人进行路径规划。

# 总结

你的双足机器人可以行走，使用传感器避开障碍物，规划路径，甚至能够看到障碍物或目标。在最后一章中，你将学会如何远程连接你的双足机器人，从而控制和监视它，而无需任何电线。
