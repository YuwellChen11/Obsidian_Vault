---

---
# 1. 安装 

## 1.1 硬件安装 

==**注意事项：断电插线，不要带电工作**==

摄像头安装：22 转15pin 排线，建议连接CAM0 端口

![](assets/Jetson%20Orin%20Nano%20super%20系统配置调用%20CSI%20摄像头/file-20260214014020356.png)

## 1.2 软件配置 

NVIDIA: Configuring the CSI Connector

Step 1：打开Terminal 命令行输入启动配置CSI Connector 界面

```javascript
$ sudo /opt/nvidia/jetson-io/jetson-io.py 
```

Step 2：选择配置Jetson 24pin CSI Connector，按enter 键继续 

![](assets/Jetson%20Orin%20Nano%20super%20系统配置调用%20CSI%20摄像头/file-20260214014020388.png)

Step 3：确认当前CSI Connector IO配置; （对硬件不熟悉的就继续吧，熟悉的可以看看是否当前就是正确的） 

![](assets/Jetson%20Orin%20Nano%20super%20系统配置调用%20CSI%20摄像头/file-20260214014020403.png)

Step 4：选择满足摄像头型号需求的CSI Connector IO 配置（一般套餐内配的摄像头型号都是imx219） 

![](assets/Jetson%20Orin%20Nano%20super%20系统配置调用%20CSI%20摄像头/file-20260214014020422.png)

==注意：硬件连接到CAM0 这个22pin 接口上的摄像头，但是选择“Camera IMX219-cam0"是不行的，要选“Camera IMX219 Dual"或者“Camera IMX219-cam1"。（不同JetPcak 版本显示界面也会有所不同） ==

Step 5：保存CSI Connector IO 配置

![](assets/Jetson%20Orin%20Nano%20super%20系统配置调用%20CSI%20摄像头/file-20260214014012129.png)

Step 6：确认保存，并重启生效CSI Connector IO 配置 

![](assets/Jetson%20Orin%20Nano%20super%20系统配置调用%20CSI%20摄像头/file-20260214014012151.png)

Step 7：任意键执行重启，一般回车 

![](assets/Jetson%20Orin%20Nano%20super%20系统配置调用%20CSI%20摄像头/file-20260214014012170.png)

Step 8 ：打开Terminal 终端，输入如下命令安装JetPack 组件 

```javascript
$ sudo apt-get update
$ sudo apt install nvidia-jetpack
```

==注意：中途不要换源，就可以安装成功，出现选择输入Y 回车，后期出现错误重复以上指令一遍即可，等待安装完成。==

## 1.3 摄像头测试 

### 1.3.1 预览摄像头显示窗口 

打开Terminal 终端查连接的设备，接入1 个摄像头会显示video0，2 个摄像头分别为video0 video1 

```javascript
$ ls /dev/video* 
```

![](assets/Jetson%20Orin%20Nano%20super%20系统配置调用%20CSI%20摄像头/file-20260214014012189.png)

```javascript
$ nvgstcapture-1.0 
```

![](assets/Jetson%20Orin%20Nano%20super%20系统配置调用%20CSI%20摄像头/file-20260214014012211.png)

系统将自动弹出摄像头显示窗口:打开/dev/video0 默认设备,执行Enter + C 退出

### 1.3.2 指定摄像头和显示分辨率 

如果接入多个摄像头，可以指定摄像头预览窗口指定摄像头输出分辨率

```javascript
$ nvgstcapture-1.0 -sensor-id=0 --cus-prev-res=1920x1080 
```

![](assets/Jetson%20Orin%20Nano%20super%20系统配置调用%20CSI%20摄像头/file-20260214014012236.png)