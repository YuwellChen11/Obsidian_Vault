1. Jetson对应目录下创建“Calibration”文件夹
2. 在该目录下打开终端，`nano capture_calib.py` 创建“capture_calib.py”文件
```bash
import cv2
import os
from datetime import datetime

# 创建保存图片的子文件夹
save_dir = "images"
os.makedirs(save_dir, exist_ok=True)

# 打开摄像头（0 是设备索引，如果不行试试 1）
cap = cv2.VideoCapture(0, cv2.CAP_V4L2)
if not cap.isOpened():
    print("无法打开摄像头，请检查设备索引或连接。")
    exit()

# 尝试设置 720p 分辨率
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)

# 获取实际分辨率（验证是否设置成功）
width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
print(f"摄像头分辨率: {width} x {height}")

print("\n按空格键拍照保存，按 'q' 退出预览")
count = 0

while True:
    ret, frame = cap.read()
    if not ret:
        print("无法获取画面")
        break

    # 显示当前拍摄数量
    cv2.putText(frame, f"Captured: {count}  SPACE: save, Q: quit", (20, 40),
                cv2.FONT_HERSHEY_SIMPLEX, 0.8, (0, 255, 0), 2)
    cv2.imshow("Camera Calibration", frame)

    key = cv2.waitKey(1) & 0xFF
    if key == ord(' '):
        filename = os.path.join(save_dir, f"calib_{count:03d}.jpg")
        cv2.imwrite(filename, frame)
        print(f"已保存 {filename}")
        count += 1
    elif key == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
```
3. 保存后在终端输入 `python capture_calib.py` 命令行运行.py文件
4. 弹出摄像头显示画面后，手动变换标定板不同位置，按空格键采集图像
5. 完成采集后按“Q”键退出
6. 使用MobaXterm连接Jetson，将采集的图像文件夹“images”下载到Windows本地中
	![](assets/Matlab标定USB摄像头（720p）/file-20260306213258254.png)
7. 打开Matlab中的“Camera Calibrator”进行标定
	![](assets/Matlab标定USB摄像头（720p）/file-20260306213400332.png)
8. 打开Camera Calibrator后添加下载下来的采集图像
	![](assets/Matlab标定USB摄像头（720p）/file-20260306213547339.png)
9. 实际测量标定板后填写方格的边长（比如我实际测量就是23.9mm）
	![](assets/Matlab标定USB摄像头（720p）/file-20260306213701047.png)
10. 导入图像后选择相机模型、径向畸变参数个数和是否计算切向畸变，选好后点击“Calibrate”标定
	![](assets/Matlab标定USB摄像头（720p）/file-20260306214146366.png)
11. 标定后页面右上角展示的是每张图像的重投影误差，右下角展示的是在相机视角下不同图像的位置关系
	![](assets/Matlab标定USB摄像头（720p）/file-20260306214704924.png)
12. 在图像的标定方法中，一般认为重投影误差<0.3pixel则标定质量良好
13. 通常若>0.3pixel则可以进行人工手动剔除误差较大图像，剔除并重新进行标定后则可提高标定的相机参数精度
	![](assets/Matlab标定USB摄像头（720p）/file-20260306215345701.png)
14. 导出相机标定参数至工作区
	![](assets/Matlab标定USB摄像头（720p）/file-20260306215647710.png)
15. 回到Matlab主页，双击相机参数
	![](assets/Matlab标定USB摄像头（720p）/file-20260306220326135.png)
	参考图![](assets/Matlab标定USB摄像头（720p）/file-20260306220204093.png)
	参考图![](assets/Matlab标定USB摄像头（720p）/file-20260306220445078.png)
16. 双击内参矩阵，查看矩阵形式。发现该内参矩阵和成像模型中的内参矩阵并不一致，这是因为Matlab中内参矩阵采用的是成像模型内参矩阵
	![](assets/Matlab标定USB摄像头（720p）/file-20260306220613092.png)
	![](assets/Matlab标定USB摄像头（720p）/file-20260306220752382.png)
	参考图![](assets/Matlab标定USB摄像头（720p）/file-20260306220641678.png)