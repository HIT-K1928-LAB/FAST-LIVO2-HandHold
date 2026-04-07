# FAST-LIVO2 handhold复现指南
本项目基于[`liv-handhold2 (LIV-Eye)`](https://github.com/hku-mars/LIV_handhold_2)、 [`liv-handhold`](https://github.com/xuankuzcr/LIV_handhold)的硬件，运行[`FAST-LIVO2`](https://github.com/hku-mars/FAST-LIVO2)。
- wiki可参考: [简达智能的wiki](https://gitee.com/gwmunan/ros2/wikis)
- 参考视频：[GundaSmart: SLAM系列之Fast Livo复现](https://www.bilibili.com/video/BV1T142197ci)

### 项目相关仓库readme导航

> #### 驱动
> - [HIKROBOT-MVS-CAM](src/HIKROBOT-MVS-CAMERA-ROS/README.md)
>> - 海康威视工业相机SDK的ros驱动依赖MVS的库文件。因此请先安装[MVS客户端](https://www.hikrobotics.com/cn/machinevision/service/download/?module=0)，根据自己的平台架构下载安装，或使用`additional_software`中预先下载的版本。
> - [livox_ros_driver2](src/livox_ros_driver2/README.md)
> - [Livox-SDK2](src/Livox-SDK2/README.md)

> #### 标定与启动
> - [FAST-Calib readme](src/FAST-Calib/README.md)
> - [FAST-Calib workflow](src/FAST-Calib/workflow.md)
> - [FAST-LIVO2 readme](src/FAST-LIVO2/README.md)

## catkin工作空间初始化
从github clone本仓库后，需要cd到工作空间根目录，执行`catkin init`完成工作空间初始化。

## 编译
如果需要编译，在src所在的目录执行 `catkin_make` 即可，必要时删除`build` 和 `devel`目录。

## 硬件
### 硬件结构
* 本项目使用的是基于GitHub [`liv-handhold2 (LIV-Eye)`](https://github.com/hku-mars/LIV_handhold_2)仓库提供文件修改的3D打印件，3D打印模型位于`CAD_models`。
* 硬件部分使用MID-360以及海康工业相机MV-CS020-10UC。
* 项目运行在Nvidia Jetson Orin NX（aarch64架构）。

### 同步器
* 本项目对应使用的是GitHub [`liv-handhold`](https://github.com/xuankuzcr/LIV_handhold)仓库中使用的自制STM32硬件同步器。  
* 关于引脚图，工业相机的引脚可直接使用仓库对应的引脚，MID-360使用`MID360-pin8`对应`STM32-PB5`、`MID360-pin10`对应`STM32-PA9`。  
* 目前不确定PA9输出对LiDAR是否真正有效。也不确定硬件同步器对LiDAR的作用有效性。
<center>

![硬件连线示意图](./readme_img/hardware-line.png)   
硬件连线示意图

![](./readme_img/livox-mid360-1.png)
![](./readme_img/livox-mid360-2.png)  
MID360引脚图

<table>
  <tr>
    <th>MVS Camera 6PIN</th>
    <th>Name</th>
    <th>I/O Type</th>
    <th>Description</th>
    <th>Peripheral Function</th>
    <th>Diagram</th>
  </tr>
  <tr>
    <td>PIN 1</td>
    <td>DC_PWR</td>
    <td>--</td>
    <td>Power Supply</td>
    <td></td>
    <td rowspan="8" style="text-align: center;">
      <img src="./readme_img/CAM-6PIN.jpg" alt="Diagram" style="width: auto; height: auto; max-width: 100%;"><br>
      <span style="display: block; text-align: center;">MVS Camera 6-Pin Interface</span>
    </td>
  </tr>
  <tr>
    <td>PIN 2</td>
    <td>OPTO_IN</td>
    <td>Line 0+</td>
    <td>Optical Isolation Input</td>
    <td>STM32 PA1</td>
  </tr>
  <tr>
    <td>PIN 3</td>
    <td>GPIO</td>
    <td>Line 2+</td>
    <td>General Purpose Input/Output</td>
    <td></td>
  </tr>
  <tr>
    <td>PIN 4</td>
    <td>OPTO_OUT</td>
    <td>Line 1+</td>
    <td>Optical Isolation Output</td>
    <td></td>
  </tr>
  <tr>
    <td>PIN 5</td>
    <td>OPTO_GND</td>
    <td>Line 0- / 1-</td>
    <td>Optical Isolation Ground</td>
    <td>STM32 GND</td>
  </tr>
  <tr>
    <td>PIN 6</td>
    <td>GND</td>
    <td>Line 2-</td>
    <td>Ground</td>
    <td></td>
  </tr>
</table>
<hr>
海康工业相机引脚图
<hr>

<table>
  <tr>
    <th>STM32</th>
    <th>Peripheral Function</th>
  </tr>
  <tr>
    <td>PA1</td>
    <td>MVS camera PIN2 (OPTO_IN)</td>
  </tr>
  <tr>
    <td>PB5</td>
    <td>MID-360 PIN8</td>
  </tr>
  <tr>
    <td>PA9</td>
    <td>MID-360 PIN10</td>
  </tr>
  <tr>
    <td>VCC</td>
    <td>所有设备共用VCC</td>
  </tr>
  <tr>
    <td>GND</td>
    <td>所有设备共用GND | MVS camera PIN5 (GND)</td>
  </tr>
</table>

</center>

## 关于海康工业相机的硬件同步
`HIKROBOT-MVS-CAMERA-ROS` 仓库本身代码没有写硬件同步  
修改软硬件同步模式需要修改 `src/HIKROBOT-MVS-CAMERA-ROS/include/hikrobot_camera.hpp` 207行附近，并重新编译：
``` hpp
        //软件触发0，硬件触发1
        // TODO： 按需修改
        // ********** frame **********/
        nRet = MV_CC_SetEnumValue(handle, "TriggerMode", 1);    // 这里修改0/1
``` 
配套使用  
`src/HIKROBOT-MVS-CAMERA-ROS/launch/hikrobot_camera_rviz_trigger.launch`   
`src/HIKROBOT-MVS-CAMERA-ROS/config/camera-trigger.yaml`  
这里外部硬件触发使用的是线路0，上升沿触发。

如果是软触发，使用  
`src/HIKROBOT-MVS-CAMERA-ROS/launch/hikrobot_camera_rviz.launch`   
`src/HIKROBOT-MVS-CAMERA-ROS/config/camera.yaml`

## MID-360的硬件同步测试

## LIVOX MID-360与海康MV-CS020-10UC一同启动
在 `src/FAST-Calib/launch_sensors/rviz_MID360_HIK.launch` 中配置了MID360和HIK-MV-CS020-10UC（对应软触发，检查`HIKROBOT-MVS-CAMERA-ROS` 仓库`hikrobot_camera.hpp` 代码并编译）  

执行 `roslaunch rviz_MID360_HIK.launch` 即可启动，（带rviz），如果不需要rviz，在launch文件里面把rviz disable即可。

相关rviz配置文件在 `src/FAST-Calib/rviz_cfg/MID360_HIKCS020.rviz` 

## rosbag文件录制
要先启动相应传感器的topic发布（见上节），再录制。   

`src/FAST-Calib/calib_data/record_MID360_HIK.sh` 是录制脚本，具体cat一下就知道

输出文件名格式为 20260401_2156.bag

## 关于LiDAR-CAM联合标定
录制完成rosbag之后，分别新建三个终端窗口，依次执行：
``` bash
roscore

rosbag play -l 20260401_2156.bag

rqt_image_view
```
保存录制好的bag中的image到`src/FAST-Calib/calib_data/`，与bag文件放在一起。

修改`src/FAST-Calib/config/qr_params.yaml` 中的参数，设置相机内参、标定板参数、以及点云约束范围、bag路径等。  
其中点云约束范围可以通过`src/FAST-Calib/scripts/distance_filter_tool.py` 得到。   
用法示例：
``` bash
python3 distance_filter_tool.py
#或
python3 distance_filter_tool.py /path/to/data.bag /path/to/output_dir
python3 distance_filter_tool.py ../calib_data/20260401_2156.bag ../output/scripts/
```
按console提示选取平面上四个点，按Q即可选取范围，填到yaml文件里面即可。

``` bash
python distance_filter_tool.py ../calib_data/20260401_2156.bag ../output/scripts/
```

设置好yaml以后，运行标定：
``` bash
roslaunch fast-calib calib.launch
```

## fastlivo2配置
得到相机-LiDAR标定结果后，填入`src/FAST-LIVO2/config/mid360.yaml`。  
相机内参填入`src/FAST-LIVO2/config/camera_pinhole_MV-CS020.yaml`  

如果更换传感器型号，可以新建yaml文件，并修改launch文件。  
launch文件这里对应`src/FAST-LIVO2/launch/mapping_mid360.launch`。

修改完配置，回到src同级目录，重新编译。


## fastlivo2启动
先启动雷达

再启动相机（记得改成trigger模式）

这里写了一个集成启动的launch脚本，文件位置： `src/FAST-Calib/launch_sensors/rviz_MID360_HIK.launch`

最后启动fastlivo2主节点
``` bash
roslaunch src/FAST-LIVO2/launch/mapping_mid360.launch
```

## 踩坑记录
### 启动fastlivo2什么也看不到  
一定记得把launch文件里面的livox点云数据结构改为自定义格式！！！  
否则启动fastlivo2什么也看不到     
文件位置 `src/FAST-Calib/launch_sensors/rviz_MID360_HIK.launch`
```
<arg name="xfer_format" default="1"/>
```

### 贴图不准（尤其是物体边缘）
这时要调整`src/FAST-LIVO2/config/mid360.yaml`的img_time_offset

先用rosbag record命令录制lidar和相机的数据（10s左右即可），然后使用`rqt_bag`工具：
``` bash
rqt_bag src/FAST-Calib/calib_data/20260402_1835.bag 
```
这里可以看到时间轴，计算数据之间的平均差写入yaml文件，效果会有所改善。
