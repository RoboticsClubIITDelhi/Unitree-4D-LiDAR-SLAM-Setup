<p align="center">
  <img src="roboclub_logo.png" alt="Robotics Club IITD" height="120">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <img src="iitd_logo.png" alt="IIT Delhi" height="120">
</p>

<h1 align="center">3D LiDAR SLAM Setup Guide</h1>

<p align="center">
  <strong>Unitree L2 with FAST_LIO (HKU-MARS)</strong><br>
  ROS 2 Humble &nbsp;|&nbsp; Ubuntu 22.04
</p>

<p align="center">
  <a href="#prerequisites"><img src="https://img.shields.io/badge/Ubuntu-22.04-E95420?logo=ubuntu&logoColor=white" alt="Ubuntu"></a>
  <a href="#prerequisites"><img src="https://img.shields.io/badge/ROS_2-Humble-22314E?logo=ros&logoColor=white" alt="ROS 2"></a>
  <a href="#step-4--fastlio-package"><img src="https://img.shields.io/badge/SLAM-FAST__LIO-blue" alt="FAST_LIO"></a>
  <a href="#step-1--unitree-lidar-sdk"><img src="https://img.shields.io/badge/LiDAR-Unitree_L2-orange" alt="Unitree L2"></a>
</p>

<p align="center"><em>Robotics Club, Indian Institute of Technology Delhi</em></p>

---

## Contributors

<table>
  <tr>
    <td align="center"><a href="https://github.com/jpmiitd"><strong>Jyotiprakash Mallik</strong></a><br><sub>@jpmiitd</sub></td>
    <td align="center"><a href="https://github.com/KeshavRaj0019"><strong>Keshav Raj</strong></a><br><sub>@KeshavRaj0019</sub></td>
    <td align="center"><a href="https://github.com/Krishanth-S-7"><strong>Krishanth S</strong></a><br><sub>@Krishanth-S-7</sub></td>
    <td align="center"><a href="https://github.com/TanviShinde1729"><strong>Tanvi Shinde</strong></a><br><sub>@TanviShinde1729</sub></td>
  </tr>
</table>

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Workspace Setup](#workspace-setup)
- [Step 1 — Unitree LiDAR SDK](#step-1--unitree-lidar-sdk)
- [Step 2 — Livox SDK 2](#step-2--livox-sdk-2)
- [Step 3 — Livox ROS 2 Driver](#step-3--livox-ros-2-driver)
- [Step 4 — FAST_LIO Package](#step-4--fastlio-package)
- [Running the SLAM Pipeline](#running-the-slam-pipeline)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

---

## Overview

This repository provides a complete, step-by-step guide for setting up 3D LiDAR-based SLAM using the **Unitree L2** LiDAR sensor and the **FAST_LIO** algorithm (developed by HKU-MARS Lab). By the end of this guide, you will have a fully functional SLAM pipeline running on ROS 2 Humble that produces real-time 3D maps and odometry data.

### Setup Pipeline

1. Set up the Unitree LiDAR SDK (hardware configuration, point cloud & IMU data acquisition)
2. Install the Livox SDK 2
3. Set up the Livox ROS 2 Driver package
4. Set up and configure the FAST_LIO package for Unitree L2

---

## Prerequisites

### Operating System & ROS

- **Ubuntu 22.04 LTS**
- **ROS 2 Humble Hawksbill** with **RViz**

### ROS 2 Additional Packages

```bash
sudo apt-get install ros-humble-pcl-ros
sudo apt-get install ros-humble-pcl-conversions
sudo apt-get install ros-humble-visualization-msgs
```

### Eigen Library

```bash
sudo apt-get install libeigen3-dev
```

---

## Workspace Setup

Create the directory structure that will hold all SDK builds and ROS 2 packages. The `FL_ws` sub-workspace is where FAST_LIO and the Livox driver will live.

```bash
cd ~
mkdir -p LiDAR_ws/FL_ws/src
```

> [!NOTE]
> All subsequent paths in this guide are relative to `~/LiDAR_ws` unless stated otherwise.

---

## Step 1 — Unitree LiDAR SDK

The Unitree LiDAR SDK allows you to configure hardware parameters (communication mode, scan range, etc.) and provides raw point cloud and IMU data from the Unitree L2 sensor.

### 1.1 Clone and Build the SDK

```bash
cd ~/LiDAR_ws
git clone https://github.com/unitreerobotics/unilidar_sdk2.git
cd unilidar_sdk2
mkdir build
cd build
cmake .. && make -j2
```

> [!NOTE]
> The SDK supports switching between serial and Ethernet communication modes. Refer to the [GitHub repository README](https://github.com/unitreerobotics/unilidar_sdk2) for detailed instructions on mode switching and range configuration.

### 1.2 Build the ROS 2 Visualization Workspace

```bash
cd ~/LiDAR_ws/unilidar_sdk2/unitree_lidar_ros2
colcon build
```

### 1.3 Configure Launch Parameters

Open the launch file at:

```
~/LiDAR_ws/unilidar_sdk2/unitree_lidar_ros2/src/unitree_lidar_ros2/launch/launch.py
```

Set the following parameter values:

```python
{'initialize_type': 1},
{'work_mode': 8}
```

> [!WARNING]
> If the serial port has changed (verify with `ls /dev/tty*`), the LiDAR will not respond. The default port is `/dev/ttyACM0`. Update the launch file if your device enumerates on a different port.

### 1.4 Test the SDK Setup

Rebuild after parameter changes, then launch the test visualization:

```bash
cd ~/LiDAR_ws/unilidar_sdk2/unitree_lidar_ros2
colcon build
source install/setup.bash
ros2 launch unitree_lidar_ros2 launch.py
```

An RViz window should open displaying a vector visualization for IMU data and a 3D point cloud from the LiDAR sensor. If both are visible, the SDK is configured correctly.

---

## Step 2 — Livox SDK 2

The Livox SDK 2 is a dependency required by the Livox ROS Driver. It provides the low-level communication layer for Livox-compatible LiDAR sensors.

```bash
cd ~/LiDAR_ws
git clone https://github.com/Livox-SDK/Livox-SDK2.git
cd Livox-SDK2
mkdir build && cd build
cmake .. && make -j$(nproc)
sudo make install
```

---

## Step 3 — Livox ROS 2 Driver

The Livox ROS Driver 2 publishes LiDAR data as ROS 2 topics. It includes a custom build script that handles the ROS 2 Humble build structure.

```bash
cd ~/LiDAR_ws/FL_ws/src
git clone https://github.com/Livox-SDK/livox_ros_driver2.git
cd ~/LiDAR_ws/FL_ws/src/livox_ros_driver2
source /opt/ros/humble/setup.bash
./build.sh humble
```

---

## Step 4 — FAST_LIO Package

FAST_LIO (Fast LiDAR-Inertial Odometry) is a computationally efficient LiDAR-inertial odometry package developed by HKU-MARS Lab. This step installs the package and configures it specifically for the Unitree L2 sensor.

### 4.1 Clone and Build FAST_LIO

```bash
cd ~/LiDAR_ws/FL_ws/src
git clone https://github.com/Ericsii/FAST_LIO.git --recursive
cd ..
rosdep install --from-paths src --ignore-src -y
colcon build --symlink-install
. ./install/setup.bash
```

### 4.2 Create the Unitree L2 Configuration File

Create a new file at:

```
~/LiDAR_ws/FL_ws/src/FAST_LIO/config/unilidar_l2.yaml
```

Paste the following configuration:

```yaml
/**:
    ros__parameters:
        feature_extract_enable: false
        point_filter_num: 4
        max_iteration: 3
        filter_size_surf: 0.5
        filter_size_map: 0.5
        cube_side_length: 1000.0
        runtime_pos_log_enable: false
        map_file_path: "./test.pcd"

        common:
            lid_topic: "/unilidar/cloud"
            imu_topic: "/unilidar/imu"
            con_frame: false
            con_frame_num: 1
            cut_frame: false
            cut_frame_time_interval: 0.1
            time_lag_imu_to_lidar: 0.085

        preprocess:
            lidar_type: 5
            scan_line: 18
            timestamp_unit: 0
            blind: 0.5

        mapping:
            imu_en: true
            start_in_aggressive_motion: true
            extrinsic_est_en: false
            imu_time_inte: 0.004
            satu_acc: 30.0
            satu_gyro: 35.0
            acc_norm: 9.81
            lidar_meas_cov: 0.01
            acc_cov_output: 500.0
            gyr_cov_output: 1000.0
            b_acc_cov: 0.0001
            b_gyr_cov: 0.0001
            imu_meas_acc_cov: 0.1
            imu_meas_omg_cov: 0.1
            gyr_cov: 0.01
            acc_cov: 0.1
            plane_thr: 0.1
            match_s: 81.0
            fov_degree: 180.0
            det_range: 100.0
            gravity_align: true
            gravity: [0.0, 0.0, -9.810]
            gravity_init: [0.0, 0.0, -9.810]

            # Transform from IMU to LiDAR
            extrinsic_T: [0.007698, 0.014655, -0.00667]
            extrinsic_R: [1.0, 0.0, 0.0,
                          0.0, 1.0, 0.0,
                          0.0, 0.0, 1.0]

        odometry:
            publish_odometry_without_downsample: true

        publish:
            path_en: true
            scan_publish_en: true
            scan_bodyframe_pub_en: false
            dense_publish_en: true

        pcd_save:
            pcd_save_en: true
            interval: -1
```

### 4.3 Rebuild the Workspace

```bash
cd ~/LiDAR_ws/FL_ws
colcon build --symlink-install
```

---

## Running the SLAM Pipeline

The SLAM pipeline requires **two terminal sessions** running simultaneously.

### Terminal 1 — LiDAR Data Publisher

Launch the Unitree SDK node to publish raw point cloud and IMU data:

```bash
cd ~/LiDAR_ws/unilidar_sdk2/unitree_lidar_ros2
source install/setup.bash
ros2 launch unitree_lidar_ros2 launch.py
```

### Terminal 2 — FAST_LIO SLAM

Launch FAST_LIO with the Unitree L2 configuration:

```bash
cd ~/LiDAR_ws/FL_ws
source install/setup.bash
ros2 launch fast_lio mapping.launch.py config_file:=unilidar_l2.yaml
```

### Accessing Odometry Data

Once the pipeline is running, pose data (position + orientation) is published on:

| Field    | Value                                              |
| -------- | -------------------------------------------------- |
| **Topic**    | `/Odometry`                                        |
| **Use Case** | Robot localization, navigation, and path planning  |

You can subscribe to this topic from any ROS 2 node for downstream localization and navigation tasks.

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
| ------- | ------------ | --- |
| LiDAR not responding | Serial port changed | Run `ls /dev/tty*` and update port in `launch.py` |
| No point cloud in RViz | SDK not rebuilt after config change | Run `colcon build` again in the SDK workspace |
| FAST_LIO crashes on start | Missing rosdep dependencies | Run `rosdep install --from-paths src --ignore-src -y` |
| Odometry drift | IMU calibration off | Verify extrinsic parameters in `unilidar_l2.yaml` |
| `colcon build` fails | Missing ROS 2 packages | Install all prerequisites from [Prerequisites](#prerequisites) |

---

## Resources

- [Unitree 4D LiDAR L2 User Manual (PDF)](https://oss-global-cdn.unitree.com/static/Unitree%204D%20LiDAR%20L2%20User%20Manual.pdf)
- [Unitree LiDAR SDK 2 — GitHub](https://github.com/unitreerobotics/unilidar_sdk2)
- [Livox SDK 2 — GitHub](https://github.com/Livox-SDK/Livox-SDK2)
- [Livox ROS Driver 2 — GitHub](https://github.com/Livox-SDK/livox_ros_driver2)
- [FAST_LIO ROS 2 Branch — GitHub](https://github.com/hku-mars/FAST_LIO/tree/ROS2)

---

<p align="center">
  <strong>Robotics Club, IIT Delhi</strong><br>
  For questions, contact the club coordinators.
</p>
