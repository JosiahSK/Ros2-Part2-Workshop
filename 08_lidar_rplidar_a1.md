# Module 08: RP-LiDAR A1 — Setup & Integration

In this module, we will connect an RP-LiDAR A1 sensor, install its ROS 2 driver, publish laser scan data, and add the LiDAR to our robot URDF. All commands are provided for both **ROS 2 Humble** and **ROS 2 Jazzy**.

---

## 🔵 ROS 2 Distribution Differences at a Glance

| Feature | ROS 2 Humble (Ubuntu 22.04) | ROS 2 Jazzy (Ubuntu 24.04) |
| :--- | :--- | :--- |
| **Ubuntu Version** | 22.04 LTS (Jammy) | 24.04 LTS (Noble) |
| **LTS Support Until** | May 2027 | May 2029 |
| **LiDAR Driver Package** | `sllidar_ros2` (GitHub clone) | `sllidar_ros2` (GitHub clone) |
| **Source ROS** | `source /opt/ros/humble/setup.bash` | `source /opt/ros/jazzy/setup.bash` |
| **Default DDS** | FastDDS | FastDDS |
| **Python Version** | Python 3.10 | Python 3.12 |

> **⚠️ Rule of Thumb:** Use **Humble** if you need maximum community packages and tutorials. Use **Jazzy** if you are starting fresh and want the longest support window.

---

## 🔌 Step 1: Connect the RP-LiDAR A1

Physically plug the RP-LiDAR A1 USB cable into your computer. Verify the OS detects it:

```bash
ls /dev/ttyUSB*
```

**Expected output:**
```text
/dev/ttyUSB0
```

If nothing appears, check `dmesg` for USB events:
```bash
dmesg | tail -20
```
Look for a line containing: `cp210x converter now attached to ttyUSB0`

---

## 🔧 Step 2: Grant USB Permissions

By default, only `root` can access `/dev/ttyUSB*`. Grant permanent access with udev rules.

### ✅ Humble & Jazzy — Same Command

```bash
sudo usermod -aG dialout $USER
```

Then **log out and log back in** (or reboot) for group changes to apply.

Verify:
```bash
groups $USER
```
Expected output includes: `dialout`

---

## 📦 Step 3: Install the LiDAR Driver

The official Slamtec ROS 2 driver (`sllidar_ros2`) is installed by cloning from GitHub. The process is identical on both distributions — only the ROS source path differs.

### 🟡 Humble

```bash
# Source ROS 2 Humble
source /opt/ros/humble/setup.bash

# Navigate to workspace src
cd ~/robot_ws/src

# Clone the official Slamtec driver
git clone https://github.com/Slamtec/sllidar_ros2.git

# Build
cd ~/robot_ws
colcon build --symlink-install --packages-select sllidar_ros2

# Source workspace
source install/setup.bash
```

### 🟢 Jazzy

```bash
# Source ROS 2 Jazzy
source /opt/ros/jazzy/setup.bash

# Navigate to workspace src
cd ~/robot_ws/src

# Clone the official Slamtec driver
git clone https://github.com/Slamtec/sllidar_ros2.git

# Build
cd ~/robot_ws
colcon build --symlink-install --packages-select sllidar_ros2

# Source workspace
source install/setup.bash
```

> **💡 Note:** The driver source code is identical for both distributions. Only the `source` command path differs.

---

## 🔑 Step 4: Apply udev Rules (Permanent Port Access)

The `sllidar_ros2` package ships a convenience script to create persistent udev rules.

### ✅ Humble & Jazzy — Same Command

```bash
cd ~/robot_ws/src/sllidar_ros2
source scripts/create_udev_rules.sh
```

**Then physically unplug and replug the USB cable.**

Verify permissions on the device:
```bash
ls -la /dev/ttyUSB0
```
Expected:
```text
crw-rw-rw- 1 root dialout 188, 0 Jun 17 10:00 /dev/ttyUSB0
```

---

## 🚀 Step 5: Launch the LiDAR Node

### 🟡 Humble

```bash
source /opt/ros/humble/setup.bash
source ~/robot_ws/install/setup.bash

ros2 launch sllidar_ros2 view_sllidar_a1_launch.py
```

### 🟢 Jazzy

```bash
source /opt/ros/jazzy/setup.bash
source ~/robot_ws/install/setup.bash

ros2 launch sllidar_ros2 view_sllidar_a1_launch.py
```

**Expected terminal output:**
```text
[sllidar_ros2_node]: RPLIDAR running on /dev/ttyUSB0 at 115200 bps
[sllidar_ros2_node]: RPlidar health status : 0 Good
[sllidar_ros2_node]: Firmware Ver: 1.29
[sllidar_ros2_node]: Hardware Rev: 7
```

RViz2 opens automatically showing the 360° laser scan ring.

---

## 🔍 Step 6: Verify Laser Scan Data

In a new terminal, confirm the `/scan` topic is publishing:

### ✅ Humble & Jazzy — Same Command

```bash
# Check the topic exists
ros2 topic list | grep scan

# Check publish rate (should be ~7.5 Hz)
ros2 topic hz /scan

# Print one full LaserScan message
ros2 topic echo /scan --once
```

**Expected output from `ros2 topic hz /scan`:**
```text
average rate: 7.504
  min: 0.126s  max: 0.141s  std dev: 0.00234s  window: 25
```

**LaserScan message fields explained:**

| Field | Value | Description |
| :--- | :--- | :--- |
| `frame_id` | `laser` | TF frame this scan belongs to |
| `angle_min` | `-3.14` rad | Start of sweep (−180°) |
| `angle_max` | `3.14` rad | End of sweep (+180°) |
| `angle_increment` | `~0.017` rad | Angular step between samples (~1°) |
| `range_min` | `0.15` m | Minimum detectable distance |
| `range_max` | `12.0` m | Maximum detectable distance |
| `ranges[]` | array of floats | Distance to nearest surface at each angle |
| `intensities[]` | array of floats | Return signal strength at each angle |

---

## 🌐 Step 7: Create the Static TF Transform

SLAM and navigation require the LiDAR scan frame (`laser`) to be connected to the robot body frame (`base_link`). Publish a static transform:

### ✅ Humble & Jazzy — Same Command

```bash
ros2 run tf2_ros static_transform_publisher \
  0 0 0.18 0 0 0 base_link laser
```

**Arguments explained:**

| Argument | Value | Meaning |
| :--- | :--- | :--- |
| `x y z` | `0 0 0.18` | LiDAR is 18 cm above `base_link` center |
| `roll pitch yaw` | `0 0 0` | LiDAR is level, facing forward |
| `parent_frame` | `base_link` | Robot body frame |
| `child_frame` | `laser` | LiDAR sensor frame |

> **📌 Tip:** Adjust `z` to match your robot's actual LiDAR mounting height.

**Verify the TF tree:**
```bash
ros2 run tf2_tools view_frames
```

Expected tree:
```text
base_link
 └── laser
```

---

## 📝 Step 8: Add LiDAR to the URDF

Add a `lidar_link` and `lidar_joint` to your `robot.urdf` file to register the sensor in the robot model. Open your URDF:

```bash
nano ~/robot_ws/src/robot_description/urdf/robot.urdf
```

Add the following XML **inside** the `<robot>` tag, after the `ultrasonic_joint` block:

```xml
  <!-- LiDAR Material -->
  <material name="green">
    <color rgba="0.1 0.8 0.1 1.0"/>
  </material>

  <!-- LiDAR Sensor Link -->
  <link name="lidar_link">
    <visual>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <!-- Cylinder approximating the RP-LiDAR A1 body: 7cm radius, 4cm tall -->
        <cylinder radius="0.035" length="0.04"/>
      </geometry>
      <material name="green"/>
    </visual>
  </link>

  <!-- LiDAR Joint: mount on top of chassis, centered -->
  <joint name="lidar_joint" type="fixed">
    <parent link="base_link"/>
    <child link="lidar_link"/>
    <!-- Centered on chassis (X=0, Y=0), 18cm above base_link origin -->
    <origin xyz="0.0 0.0 0.18" rpy="0 0 0"/>
  </joint>
```

**Validate the updated URDF:**
```bash
check_urdf ~/robot_ws/src/robot_description/urdf/robot.urdf
```

Expected output now includes `lidar_link`:
```text
robot name is: two_wheel_robot
---------- Successfully Parsed XML ----------
root Link: base_footprint has 1 child(ren)
    child(1):  base_link
        child(1):  left_wheel
        child(2):  right_wheel
        child(3):  ultrasonic_link
        child(4):  lidar_link         ← New!
```

---

## 🔄 Step 9: Rebuild After URDF Changes

After editing the URDF, rebuild and source the workspace:

### 🟡 Humble

```bash
cd ~/robot_ws
rm -rf build install log
colcon build --symlink-install
source install/setup.bash
```

### 🟢 Jazzy

```bash
cd ~/robot_ws
rm -rf build install log
colcon build --symlink-install
source install/setup.bash
```

> **💡 Note:** With `--symlink-install`, URDF edits take effect immediately **without** a rebuild. Only run the clean rebuild when `setup.py` or `package.xml` changes.

---

## 🖥️ Step 10: Visualize LiDAR in RViz2

Launch the full visualization stack:

### 🟡 Humble

```bash
source /opt/ros/humble/setup.bash
source ~/robot_ws/install/setup.bash
ros2 launch robot_description display.launch.py
```

### 🟢 Jazzy

```bash
source /opt/ros/jazzy/setup.bash
source ~/robot_ws/install/setup.bash
ros2 launch robot_description display.launch.py
```

In RViz2, add a **LaserScan** display:
1. Click **Add** → **By Topic** → `/scan` → **LaserScan** → **OK**
2. Set **Fixed Frame** to `base_link`
3. Set **Size (m)** to `0.03`
4. Set **Color** to cyan

You should see:
- The 3D robot model with the green LiDAR cylinder on top
- A 360° ring of cyan dots (the live laser scan) surrounding the robot

---

## 📋 Full Command Sequence Reference

### 🟡 Humble — Complete LiDAR Setup Sequence

```bash
# 1. Source ROS
source /opt/ros/humble/setup.bash

# 2. Clone driver (first time only)
cd ~/robot_ws/src
git clone https://github.com/Slamtec/sllidar_ros2.git

# 3. Apply udev rules (first time only)
cd ~/robot_ws/src/sllidar_ros2
source scripts/create_udev_rules.sh
# Replug USB cable

# 4. Build workspace
cd ~/robot_ws
colcon build --symlink-install
source install/setup.bash

# 5. Launch LiDAR driver + RViz
ros2 launch sllidar_ros2 view_sllidar_a1_launch.py

# 6. (New terminal) Publish static TF
source /opt/ros/humble/setup.bash
source ~/robot_ws/install/setup.bash
ros2 run tf2_ros static_transform_publisher 0 0 0.18 0 0 0 base_link laser

# 7. (New terminal) Verify scan
ros2 topic hz /scan
```

---

### 🟢 Jazzy — Complete LiDAR Setup Sequence

```bash
# 1. Source ROS
source /opt/ros/jazzy/setup.bash

# 2. Clone driver (first time only)
cd ~/robot_ws/src
git clone https://github.com/Slamtec/sllidar_ros2.git

# 3. Apply udev rules (first time only)
cd ~/robot_ws/src/sllidar_ros2
source scripts/create_udev_rules.sh
# Replug USB cable

# 4. Build workspace
cd ~/robot_ws
colcon build --symlink-install
source install/setup.bash

# 5. Launch LiDAR driver + RViz
ros2 launch sllidar_ros2 view_sllidar_a1_launch.py

# 6. (New terminal) Publish static TF
source /opt/ros/jazzy/setup.bash
source ~/robot_ws/install/setup.bash
ros2 run tf2_ros static_transform_publisher 0 0 0.18 0 0 0 base_link laser

# 7. (New terminal) Verify scan
ros2 topic hz /scan
```

---

## ⚠️ Troubleshooting

| Problem | Cause | Fix |
| :--- | :--- | :--- |
| `No such file or directory: /dev/ttyUSB0` | LiDAR not detected by OS | Replug USB; run `dmesg \| tail -20` to check |
| `Permission denied: /dev/ttyUSB0` | User not in `dialout` group | Run `sudo usermod -aG dialout $USER` then log out/in |
| `/scan` not publishing | Node not started | Run `ros2 node list \| grep sllidar` |
| `health status: Error` | Hardware or cable fault | Try a different USB cable or port |
| RViz shows no scan | Wrong Fixed Frame or topic | Set Fixed Frame to `base_link`; verify `/scan` topic QoS |
| `colcon: command not found` | ROS not sourced | Run `source /opt/ros/humble/setup.bash` (or `jazzy`) first |

---

*The LiDAR is now fully integrated into the robot stack. For SLAM mapping with this sensor, see the companion repository: [ros2_humble_rplidar_a1_slam](https://github.com/JosiahSK/ros2_humble_rplidar_a1_slam).*
