---
cover: "[[assets/Project Technical Document/file-20260214014017498.jpeg]]"
---
# 📔 Issue & Solution

> [!tip] 💡
> 核心模块：Jetson Orin Nano Super (8GB) CLB
> 历史系统版本：JetPack 5.1.x | Ubuntu 20.04
> 现行系统版本：JetPack 6.2.1 | Ubuntu 22.04

## ❌ Issue：CSI摄像头测试命令报错
- 环境：使用MobaXterm同一局域网SSH远程连接Jetson
- 现象：输入`nvgstcapture-1.0`测试摄像头报错/黑屏 
	![](assets/Project%20Technical%20Document/file-20260214014016570.jpg)![](assets/Project%20Technical%20Document/file-20260214014016587.jpg)
- 分析：远程协议无法直接承载这种高带宽的底层硬件渲染，不能使用SSH测试需连接到显示器桌面才可以。另外，使用NoMachine测试也是如此，远程协议（X11/、NoMachine）不兼容硬件加速的渲染器

## ✅ Solution：连接物理显示屏使用测试命令 / 插入HDMI欺骗器模拟物理显示屏（针对NoMachine上不显示）

## ❌ Issue：CSI摄像头无法识别
- 环境：Jetson连接物理显示屏/使用NoMachine远程连接Jetson图形化界面
- 现象：重装系统后终端输入查找连接设备命令`ls /dev/video*`显示无法访问
	![](assets/Project%20Technical%20Document/file-20260214014016605.jpg)
- 分析：系统首次连接CSI摄像头需要在CSI Connector界面中进行软件配置

## ✅ Solution：参考技术文档进行软件配置
- 详细操作参考如下技术文档： [Jetson Orin Nano super 系统配置调用 CSI 摄像头](Jetson%20Orin%20Nano%20super%20系统配置调用%20CSI%20摄像头.md)
 ⚠️ ==CSI摄像头不能热插拔！需要断电后插线再上电使用，否则也会导致检测不到 CSI摄像头！==

## ❌ Issue：Python调用CSI摄像头报错
- 环境：通过HDMI欺骗器连接NoMachine使用，且使用 `nvgstcapture-1.0` 和 `ls /dev/video*` 均有显示和输出
- 现象：在 `~/CSI-Camera` 目录下输入 `python3 simple_camera.py` 命令报错  `Error: Unable to open camera` 
	![](assets/Project%20Technical%20Document/file-20260214014016619.png)
	![](assets/Project%20Technical%20Document/file-20260214014016638.png)
- 分析：使用NumPy 1.x编译的模块无法在NumPy 2.2.6中运行，NumPy 2.0是一个重大更新，破坏了许多旧版本二进制文件的兼容性。在Jetson这种嵌入式系统上，系统自带的python3-opencv通常比较保守，只支持NumPy 1.x

## ✅ Solution: 降级NumPy / 配置Miniforge
- **卸载当前的 NumPy：**`pip uninstall numpy`
- **安装兼容的 NumPy 1.x 版本：**`pip install "numpy<2"`
- **验证修复：** 再次运行之前的检测命令，看是否还会报错`python3 -c "import cv2; print(cv2.getBuildInformation())" | grep -i gstreamer`
	![](assets/Project%20Technical%20Document/file-20260214014016653.png)

# 🏗️ Environment Setup

## ⚙️ 系统刷写与Super模式开启
### 📌 配置目的
  将系统从不稳定的旧版（JetPack 5.1.x / Ubuntu 20.04）升级至最新的开发环境（JetPack 6.2.1 / Ubuntu 22.04），并确保CLB Super 板的硬件特性被正确激活
 
### 🛠️ 准备工作
- 准备一个合适的Ubuntu环境（我用的是VMware虚拟机做的Ubuntu 22.04）
- ==⚠️ 若要刷较新的系统则需选择版本较高的Ubuntu，比如我刷写的是JetPack6.2.1则需选择Ubuntu 20.04以上的，而不能选择Ubuntu 18.04，故我虚拟机使用的是Ubuntu 22.04==
	![](assets/Project%20Technical%20Document/file-20260214014016664.png)
	==⚠️ 虚拟机配置中，硬盘需分配100GB以上（前提是虚拟机所在磁盘空间充足），因为烧录过程中会产生大量的临时文件和缓存 ，并且虚拟机最好装在本机磁盘，不要装在移动硬盘上，否则就会如下图所示：==即将虚拟机安装在了移动硬盘，并且移动硬盘的剩余空间不足！！！
	![](assets/Project%20Technical%20Document/file-20260214014016678.jpg)
	==**因此会导致虚拟机强制暂停，且硬盘由于空间不足会自动断开连接，此时如果立即插上是检测不到的，遇到这种情况先别着急，先让硬盘缓会再插上或将电脑重启再插上就能检测到了**==
- 在虚拟机Ubuntu系统上从[https://developer.nvidia.com/sdk-manager](https://developer.nvidia.com/sdk-manager)中下载.deb(x86_64)的安装包，并按提示安装好SDK Manager烧写工具
	![](assets/Project%20Technical%20Document/file-20260214014016698.png) 

### 🚀 核心步骤 
1. 让Jetson Orin Nano开发板进入APX模式（烧录模式），即用杜邦线短接GND和REC排针；用配套的USB转Type-c线连接电脑自带的USB口和Jetson的系统烧录接口==（最好不要用电脑外接拓展坞连接USB线，防止出现传输不稳定）==
2. 接好线上电后，在电脑端会弹出“检测到新的USB设备”，选择连接到虚拟机
	可以在虚拟机系统设置中添加USB设备筛选器，使得只要Jetson接入就能被虚拟机抓取到
	![](assets/Project%20Technical%20Document/file-20260214014016714.png)
3. 选择好对应的开发板和JetPack版本，点击“CONTINUE”下一步
	选择Jetson Orin Nano 8GB（官方核心板），不要去选择developer Kit（开发套件）
	![](assets/Project%20Technical%20Document/file-20260214014016727.png)
	选择自己需要烧录的JetPack版本，我这最后选择的是JetPack6.2.1
	![](assets/Project%20Technical%20Document/file-20260214014016746.jpg)
4. 选择好需要烧录的基本系统（必选），勾选同意对应条款，点击“CONTINUE”下一步
	此处只勾选JETSON LINUX及其下属文件，其余JetPack组件等待完成系统安装后根据需要再下载，由此也能加快下载的进度
	![](assets/Project%20Technical%20Document/file-20260214014016764.png)
5. 点击下一步后可能会需要我们输入Ubuntu系统密码，进入到STEP 03后会有弹窗让你选择，我们直接点击弹窗右下角的“Skip”跳过烧写这步
	正常在SDK Manager是需要在这选择相关配置信息的，但是我们后面会用指定命令行的方式去刷写系统（即刷Super模式的系统），故此处跳过 
	![](assets/Project%20Technical%20Document/file-20260214014016789.png)
6. 此时可以关闭SDK Manager，我们所需的系统文件已经下载到了Ubuntu的主文件夹下
	打开“主文件夹”我们会看到有一个名为“nvidia”的文件夹
	![](assets/Project%20Technical%20Document/file-20260214014016831.png)
	我们一直打开到该目录下，也可以在终端用cd命令行跳转至此处目录下`/nvidia/nvidia_sdk/JetPack_6.2.1_Linux_JETSON_ORIN_NANO_TARGETS/Linux_for_Tegra`
	![](assets/Project%20Technical%20Document/file-20260214014016892.png)
7. 根据Nvidia官方文档，我们找到对应开发套件刷写Super模式的指定命令行
	即图中框出的Jetson Orin Nano Developer Kit with Super Configuration (NVMe) ，因为我手上这块Jetson只接了一块NVMe固态硬盘，故我选择这部分指定命令行
	![](assets/Project%20Technical%20Document/file-20260214014016905.png)
8. 将指定命令行拷贝到之前目录下新建的.txt文件中，删除原命令行的换行符号，整理好并复制
	在该目录下打开终端，我们使用`touch 1.txt` 命令创建.txt文件
	![](assets/Project%20Technical%20Document/file-20260214014016913.png)
	打开创建的.txt文件并整理成以空格分隔
	![](assets/Project%20Technical%20Document/file-20260214014016927.png)
9. 复制整理后的命令行，从当前目录下打开终端，粘贴命令行并点击Enter键运行，此时需要输入Ubuntu的密码
	==⚠️  运行前最好确保Jetson还在虚拟机USB设备的连接状态，没问题即可运行命令行==

	```javascript
		sudo ./tools/kernel_flash/l4t_initrd_flash.sh --external-device nvme0n1p1 -c tools/kernel_flash/flash_l4t_t234_nvme.xml -p "-c bootloader/generic/cfg/flash_t234_qspi.xml" --showlogs --network usb0 jetson-orin-nano-devkit-super internal
	```
	

==**开始烧录后务必不要离开，需等候在电脑前，因为到Step 3后系统会弹出“检测到新的USB设备”，此时我们应尽快点击连接到虚拟机，否则连接将会超时导致烧录失败！**==
1. 烧录完成后终端会跳回命令行，此时拔掉短接的杜邦线，HDMI线连接上屏幕，将键盘鼠标等外设连接到Jetson上。进入新系统界面完成相关设置后，即可在界面右上角看到

> [!note]+ ## ⚙️ 配置Miniforge
> ### 📌 配置目的 
> 
> 通过配置虚拟环境（Miniforge），可以让标定程序运行在NumPy 1.x环境下，而让未来的SAM3D或高级AI模型运行在NumPy 2.x环境下，互不干扰。将Jetson变成一个“模块化、高性能且不容易崩”的开发平台
> 
> ### 🛠️ 准备工作
> 
> - **网络环境**：确保能够稳定连接GitHub下载安装脚本与Conda镜像源
> - **系统确认**：核对系统为JetPack 6.2.1，确认基础Python 版本为3.10，确保后续虚拟环境版本一致
> - **冲突清理**：卸载之前全局pip安装的NumPy 2.x版本，避免其干扰虚拟环境对系统OpenCV库的调用
> - **路径定位**：确定系统预装OpenCV的.so文件路径（通常在`/usr/lib/python3/dist-packages/cv2`），用于后续在虚拟环境中建立软链接
> 
> ### 🚀 核心步骤
> 
> 1. 安装Minifoge
> > [!note]+ 输入以下代码，一路输入yes或按回车，最后重启终端或执行`source ~/.bashrc`
> > ```javascript
> > wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-aarch64.sh
> > bash Miniforge3-Linux-aarch64.sh
> > ```
> 2. 创建并激活虚拟环境
> > [!note]+ 创建环境
> > ```javascript
> > # 创建名为 arm_vision 的 Python 3.10 环境
> > conda create -n arm_vision python=3.10
> > # 激活环境
> > conda activate arm_vision
> > ```
> > 
> > ==注：使用Python 3.10是为了匹配JetPack 6.2.1的系统默认版本==
> 3. 桥接系统OpenCV（解决GStreamer问题）
> > [!note]+ 安装兼容的NumPy
> > ```javascript
> > pip install "numpy<2"
> > ```
> > 
> > ==注：必须使用1.x版本，否则会与系统OpenCV产生二进制冲突==
> 
> > [!note]+ **注入路径**：通过.pth文件直接让虚拟环境识别到带GStreamer支持的系统级 OpenCV
> > ```javascript
> > echo "/usr/lib/python3/dist-packages" > $CONDA_PREFIX/lib/python3.10/site-packages/opencv_path.pth
> > ```
> 4. 验证环境
> > [!note]+ **运行测试**：检查OpenCV是否加载成功且支持硬件加速
> > ```javascript
> > python3 -c "import cv2; print('✅ 成功:', cv2.__version__); print(cv2.getBuildInformation())" | grep -i gstreamer
> > ```
> > 
> > **预期结果**：输出应包含 `GStreamer: YES` 
> > 
> > ![](assets/Project%20Technical%20Document/file-20260214014016936.png)
> > 
> > 💡 激活专属环境：在（base）环境下输入命令行激活（arm_vision）环境
> > 
> > ```javascript
> > conda activate arm_vision
> > ```
> > 
> > ![](assets/Project%20Technical%20Document/file-20260214014016954.png)
> > 
> > ==注：转换成其它环境同理==

> [!note]+ ## ⚙️ Matlab相机内参标定
> ### 📌 配置目的 
> 
> - **获取核心参数**：计算相机的内参矩阵 (K) 和畸变系数 (D)，这是视觉抓取的基础
> - **消除图像畸变**：纠正镜头造成的边缘形变，确保“所见即所得”，提高定位精度
> - **坐标系转换**：建立像素坐标与物理空间坐标的映射关系，为后续手眼标定提供准确的输入
> 
> ### 🛠️ 准备工作
> 
> - **图像采集**：使用 Jetson 上的 CSI 摄像头拍摄 20 张以上不同角度、覆盖全场的棋盘格照片
> - **物理测量**：精准测量打印标定板的方格边长（如：我打印的标定板方格实际边长是23.88 mm），确保比例尺准确
> - **工具选择**：在 Windows 运行 MATLAB Camera Calibrator 插件进行高精度计算
> - **结果验证**：确保平均重投影误差（Mean Reprojection Error）小于 0.5 像素（本项目实测为 0.15 像素，极优）
> 
> ### 🚀 核心步骤
> 
> 1. 打开Matlab中的Camera Calibrator
> > [!note]+ 点击Matlab中APP一栏，找到“图像处理和计算机视觉”下的“Camera Calibrator”
> > ![](assets/Project%20Technical%20Document/file-20260214014016965.png)
> > 
> > ![](assets/Project%20Technical%20Document/file-20260214014016990.png)
> 2. 添加图片
> > [!note]+ 打开“Camera Calibrator”后，点击“Add Images”中的“From file”从本地添加采集好的20张以上标定板图像
> > ![](assets/Project%20Technical%20Document/file-20260214014017013.png)
> 
> > [!note]+ 全选添加图像后，在弹窗输入标定板方格实际边长（如：我打印的标定板方格实际边长是23.88 mm），选择图像畸变（如：我使用的是 **77度镜头**，**故我选择图像低畸变。**==高畸变====**通常用于鱼眼镜头或 120 度以上的超广角镜头**==**）**
> > ![](assets/Project%20Technical%20Document/file-20260214014017032.png)
> 
> > [!note]+ 导入图像数据后，一般选择“Standard”标准模式，在“Options”设置中选择“3 Coefficients”3参数和“Tangential Distortion”切向畸变
> > ![](assets/Project%20Technical%20Document/file-20260214014017096.png)
> 3. 开始标定
> > [!note]+ 完成设置后点击绿色按钮“Calibrate”开始标定
> > ![](assets/Project%20Technical%20Document/file-20260214014017146.png)
> 4. 手动剔除高重投影误差样本
> > [!note]+ 图像完成标定后在界面右上方显示的是“Reprojection Errors”重投影误差，右下方显示的是“Camera-centtric”以相机为中心视角展示的各个图像三维位置关系
> > ![](assets/Project%20Technical%20Document/file-20260214014017184.png)
> 
> > [!note]+ 拖动重投影误差图中的红线选取极端值（如图重投影误差最高值是0.26像素，总平均重投影误差值是0.17像素），==若重投影误差<0.3 像素一般评估为标定效果良好==，若重投影误差>0.3像素，则需手动在柱状图中点击需要剔除的高误差柱，左侧会自动跳转到对应图像，右击该图像点击“Remove and Recalibrate”移除并重新标定
> > ![](assets/Project%20Technical%20Document/file-20260214014017300.png)
> 
> > [!note]+ 例如：在剔除1张较高重投影误差的图像后，重投影误差最高值降低至0.25像素
> > ![](assets/Project%20Technical%20Document/file-20260214014017312.png)
> 5. 导出相机参数至工作区
> > [!note]+ 完成手动剔除后，点击上方“Export Camera Parameters”导出相机参数，选择“Export Parameters to Workspace”导出参数至工作区。上方还有一个“Show Undistorted”的按钮，点击可查看畸变校正后的图像
> > ![](assets/Project%20Technical%20Document/file-20260214014017323.png)
> 6. 查看相机参数
> > [!note]+ 将参数导出至工作区后我们回到Matlab主页，点击工作区中的相机参数一栏，此时我们找到变量中的“IntrinsicMatrix”内参矩阵
> > ![](assets/Project%20Technical%20Document/file-20260214014017337.png)
> 
> > [!note]+ 点击内参矩阵具体数值进入查看
> > ![](assets/Project%20Technical%20Document/file-20260214014017353.png)
> > 
> > ![](assets/Project%20Technical%20Document/file-20260214014017367.png)
> > 
> > **⚠️ 此处需要注意的是，Matlab中的内参矩阵和OpenCV中的内参矩阵不同，Matlab中采用的内参矩阵格式是OpenCV中内参矩阵的转置 **
> > 
> > <!-- Column 1 -->
> > > [!note]+ **OpenCV中的内参矩阵 **
> > > $$
> > > \color{red}
> > > \begin{bmatrix}
> > > f_x & 0 & c_x\\
> > >  0  &f_y& c_y\\
> > >  0  & 0 &  1
> > > \end{bmatrix}
> > > $$
> > 
> > <!-- Column 2 -->
> > > [!note]+ **Matlab中的内参矩阵**
> > > $$
> > > \begin{bmatrix}
> > > f_x & 0 &  0\\
> > >  0  &f_y&  0\\
> > > c_x &c_y&  1
> > > \end{bmatrix}
> > > $$
> > > 
> > 
> > 因此我们需要在Matlab内参矩阵中一一记下$f_x、f_y、c_x、c_y$对应的四个参数值，从而在 Python 脚本里用 NumPy 手动填进去
> 7. 保存相机参数
> > [!note]+ 在工作区右键相机参数值，选择另存为至本地以供后面使用
> > ![](assets/Project%20Technical%20Document/file-20260214014017482.png)
> 
> 内参矩阵（K）：
> $$
> K=
> \begin{bmatrix}
> 1190.8 &   0    &  618.6248\\
>    0   & 1189.2 &  355.4293\\
>    0   &   0    &      1
> \end{bmatrix}
> $$
> 畸变系数（D）：
>     - $k_1=-0.0093,k_2=0.2953,k_3=-0.9149$
>     - $p_1=0.0022,p_2=-0.0014$