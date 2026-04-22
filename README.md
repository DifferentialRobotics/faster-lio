## 概述

本功能包是为**微分智飞**公司旗下教育无人机子品牌**非凸空间**适配的激光雷达定位与建图算法。
其基于开源激光惯性里程计算法 **[Faster-LIO](https://github.com/gaoxiang12/faster-lio)**，并在原代码基础上增加了**重力对齐**功能，使得无人机以倾斜姿态启动定位时能利用IMU数据将初始化的姿态与真实姿态保持一致，避免了点云地图倾斜。同时，本功能包集成了适配**Livox-Mid360**雷达的驱动包 **[livox_ros_driver2](https://github.com/Livox-SDK/livox_ros_driver2)**，可直接将该功能包下载至自己工作空间编译。

## 使用步骤
### 1. 下载源码并编译:
```
cd ~/workspace/src/realflight_modules
git clone https://github.com/DifferentialRobotics/faster-lio.git
cd ~/workspace
catkin_make
```

### 2. 配置雷达IP:
在 [MID360_config.json](livox_ros_driver2/config/MID360_config.json) 中修改 `host_net_info` 下的各ip为自己雷达的ip；修改`lidar_configs` 下的ip为机载电脑与雷达相连接网口的ip。


### 3. 配置IMU与雷达外参

修改 [mid360.yaml](config\mid360.yaml) 文件中 `imu_topic` 与 `mapping` 下各参数来切换 IMU 配置：

**选项 1：使用雷达自带 IMU**
```yaml
common:
    imu_topic: "/livox/imu"

mapping:
    # IMU-Lidar 外参
    extrinsic_T: [-0.011, -0.02329, 0.04412]
    extrinsic_R: [1, 0, 0, 0, 1, 0, 0, 0, 1]
    # IMU-机体中心 外参（斜装15°）
    world_transform_T: [ 0.105614, 0.010778, -0.000389]
    world_transform_R: [ 0.9659258, 0, -0.2588190, 0, 1, 0, 0.2588190, 0, 0.9659258]
```

**选项 2：使用飞控 IMU**
```yaml
common:
    imu_topic: "/mavros/imu/data"

mapping:
    # IMU-LiDAR 外参（斜装15°）
    extrinsic_T: [0.094614, -0.012512, 0.043731]
    extrinsic_R: [0.9659258, 0, 0.2588190, 0, 1, 0, -0.2588190, 0, 0.9659258]
    # IMU-机体中心 外参
    world_transform_T: [ 0.0, 0.0, 0.0]
    world_transform_R: [ 1, 0, 0, 0, 1, 0, 0, 0, 1]
```

**切换方法**：注释/取消注释对应的 `imu_topic` 和 `mapping` 参数组即可。

### 4. 启动雷达驱动以及定位文件
```
roslaunch mavros px4.launch #若使用飞控imu
roslaunch faster_lio mapping_mid360.launch
roslaunch ekf ekf_lidar.launch #将里程计数据频率提高为200Hz
```
其中ekf功能包包含在仓库 [Diff-Planner](https://github.com/DifferentialRobotics/Diff-Planner) 中。

## 致谢
本功能包基于 **[Faster-LIO](https://github.com/gaoxiang12/faster-lio)** 实现并集成了 **[livox_ros_driver2](https://github.com/Livox-SDK/livox_ros_driver2)** 驱动包，特此感谢两个项目作者团队的开源贡献。
