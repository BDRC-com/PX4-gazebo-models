# x500 Stereo Camera (1280x480) Configuration Guide

This document details the extrinsic parameters and configuration for the `x500_stereo_camera_1280_480` simulation model.

## 1. Camera Specifications

*   **Model Name**: `stereo_camera_1280_480`
*   **Resolution**: 640 x 480 (per eye)
*   **Horizontal FOV**: 1.3962634 rad (~80 degrees)
*   **Focal Length (approx)**: 381.36 pixels
*   **Baseline**: 0.07 meters (70 mm)

## 2. Extrinsic Parameters (Mounting Position)

The stereo camera assembly is mounted on the `x500` frame relative to the center of gravity (CoG) / `base_link`.

### Assembly Position (relative to `base_link`)
*   **X (Forward)**: 0.24 m
*   **Y (Left)**: 0.00 m
*   **Z (Up)**: 0.242 m
*   **Rotation (R, P, Y)**: (0, 0, 0)

### Camera Positions (relative to Assembly Center)
*   **Left Camera**:  Y = +0.035 m
*   **Right Camera**: Y = -0.035 m

## 3. ROS TF Configuration

To use this camera with ROS packages (like VINS-Fusion, ORB-SLAM, or stereo_image_proc), you must publish the static transforms. Gazebo does not publish these to `/tf` by default.

### Coordinate Frames
*   **Gazebo Frame**: X-Forward, Y-Left, Z-Up
*   **ROS Optical Frame**: Z-Forward, X-Right, Y-Down

### Static Transform Publisher Commands

#### 1. Base to Camera Assembly Center
```bash
# Format: x y z yaw pitch roll frame_id child_frame_id
ros2 run tf2_ros static_transform_publisher 0.24 0 0.242 0 0 0 base_link stereo_camera_link
```

#### 2. Assembly to Left/Right Optical Frames
*Note: The Gazebo model uses `camera_left_link` and `camera_right_link` as the physical mounting points. The ROS messages will use these as frame IDs.*

**Left Camera Optical Frame:**
```bash
# 1. Publish transform from base to left camera link (Physical position)
# Offset: x=0.24, y=0.035, z=0.242
ros2 run tf2_ros static_transform_publisher 0.24 0.035 0.242 0 0 0 base_link camera_left_link

# 2. Publish transform from left camera link to optical frame (Rotation for ROS)
ros2 run tf2_ros static_transform_publisher 0 0 0 -1.5708 0 -1.5708 camera_left_link camera_left_optical_frame
```

**Right Camera Optical Frame:**
```bash
# 1. Publish transform from base to right camera link (Physical position)
# Offset: x=0.24, y=-0.035, z=0.242
ros2 run tf2_ros static_transform_publisher 0.24 -0.035 0.242 0 0 0 base_link camera_right_link

# 2. Publish transform from right camera link to optical frame (Rotation for ROS)
ros2 run tf2_ros static_transform_publisher 0 0 0 -1.5708 0 -1.5708 camera_right_link camera_right_optical_frame
```

## 4. Calibration Matrix (Approximate)

If you need a `yaml` calibration file, use these theoretical values derived from the SDF:

```yaml
image_width: 640
image_height: 480
camera_name: stereo_camera
camera_matrix:
  rows: 3
  cols: 3
  data: [381.36, 0, 320.5, 0, 381.36, 240.5, 0, 0, 1]
distortion_model: plumb_bob
distortion_coefficients:
  rows: 1
  cols: 5
  data: [0, 0, 0, 0, 0] # Ideal simulation lens
rectification_matrix:
  rows: 3
  cols: 3
  data: [1, 0, 0, 0, 1, 0, 0, 0, 1]
projection_matrix:
  rows: 3
  cols: 4
  # P[0,3] (Tx) = -fx * baseline
  # Left Camera Tx = 0
  # Right Camera Tx = -381.36 * 0.07 = -26.695
  data: [381.36, 0, 320.5, 0, 0, 381.36, 240.5, 0, 0, 0, 1, 0]
```

## 5. IMU Extrinsics

The IMU is located at the center of the vehicle (`base_link`), which corresponds to the Center of Gravity (CoG).

*   **Frame ID**: `base_link`
*   **Position**: (0, 0, 0)
*   **Rotation**: (0, 0, 0)

The IMU data is typically bridged from the Gazebo topic `/imu` to ROS 2. The `frame_id` in the ROS message will be `base_link`.

