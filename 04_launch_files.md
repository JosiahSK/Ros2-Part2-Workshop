# Module 04: Launch Files in ROS 2

In this module, we will learn how to orchestrate multiple ROS 2 nodes using a single Python launch script.

---

## 🚀 Why Do We Need Launch Files?

Imagine our physical 2WD robot is navigating a room. To operate, it must run:
1. A LiDAR driver node to read distance data.
2. A motor driver node to spin the wheels.
3. An IMU sensor node to measure tilt.
4. A SLAM mapping node to build a map.
5. An obstacle avoidance control node.

### The Manual Way (No Launch Files)
Without launch files, you would have to open **five separate terminal windows** on the robot's onboard computer and manually run each command:
```bash
# Terminal 1
ros2 run lidar_driver lidar_node
# Terminal 2
ros2 run motor_driver motor_node
# Terminal 3
ros2 run imu_driver imu_node
# Terminal 4
ros2 run mapping_pack mapping_node
# Terminal 5
ros2 run controller_pack control_node
```
This is tedious and inefficient. If one node crashes, restarting the system is difficult.

### The Automated Way (With Launch Files)
With launch files, we combine these nodes into a single configuration script. Sourcing this script starts all nodes together:
```bash
ros2 launch robot_description display.launch.py
```

---

## 💻 Command: Sourcing a Launch File

```bash
ros2 launch robot_description display.launch.py
```

### 1. Purpose
Executes the launch description script (`display.launch.py`) from package `robot_description` to run all configured nodes in parallel.

### 2. Line-by-Line Explanation
* `ros2`: The entry command tool.
* `launch`: Command category to start launch configurations.
* `robot_description`: Name of the source package containing the launch script.
* `display.launch.py`: The name of the launch script.

### 3. Expected Output
The console will start outputting logger outputs from all launched nodes simultaneously:
```text
[INFO] [launch]: All log files can be found below /home/user/.ros/log/...
[INFO] [launch]: Default logging verbosity is set to INFO
[INFO] [robot_state_publisher-1]: process started with pid [12345]
[INFO] [rviz2-2]: process started with pid [12346]
```

### 4. Why It Is Needed
It allows you to configure node parameters, map topics, and start processes with a single command.

### 5. What Happens If Skipped
You must run every node manually in separate terminals, configuring all parameters on the command line every time.

### 6. Common Beginner Mistakes
* **Wrong launch file extension:** ROS 2 **requires** the launch file to end with `.launch.py`. A file named `display.launch` (without the `.py`) will **not** be recognized by `ros2 launch`. If you created your file without the `.py` extension, rename it:
  ```bash
  mv ~/robot_ws/src/robot_description/launch/display.launch \
     ~/robot_ws/src/robot_description/launch/display.launch.py
  ```
* **Forgetting to update `setup.py`:** After renaming, verify that `setup.py` uses a glob pattern that only matches `.launch.py` files:
  ```python
  (os.path.join('share', package_name, 'launch'),
      glob('launch/*.launch.py')),
  ```
  If `setup.py` used `glob('launch/*.launch')` it would no longer find the renamed file.
* **Forgetting to install `joint-state-publisher-gui`:** The `display.launch.py` file launches `joint_state_publisher_gui`. Install it with:
  ```bash
  sudo apt install ros-humble-joint-state-publisher-gui
  ```

### 7. Real Robot Relevance
All production robots use launch files. When starting a robot, you run a master launch file (e.g., `bringup.launch.py`) that initializes the low-level hardware connections.

---

## ✍️ Creating a Launch File — Step-by-Step

Follow these steps exactly to create the launch file with the **correct name and location**.

---

### Step 1: Create the `launch/` Directory

The launch file must live inside a folder called `launch/` at the root of your package:

```bash
mkdir -p ~/robot_ws/src/robot_description/launch
```

**Why:** ROS 2 (via `setup.py`) looks for launch files inside this specific folder. If it does not exist, the build system has nothing to install.

---

### Step 2: Create the Launch File with the Correct Name

> ⚠️ **Critical:** The file name **must** end with `.launch.py`. Do NOT name it `display.launch`, `display.py`, or anything else.

Create the file using `touch` to guarantee the correct name:

```bash
touch ~/robot_ws/src/robot_description/launch/display.launch.py
```

Verify the file was created with the right name:

```bash
ls ~/robot_ws/src/robot_description/launch/
```

**Expected output:**
```text
display.launch.py
```

If you see `display.launch` (missing `.py`), rename it immediately:
```bash
mv ~/robot_ws/src/robot_description/launch/display.launch \
   ~/robot_ws/src/robot_description/launch/display.launch.py
```

---

### Step 3: Open the File for Editing

```bash
nano ~/robot_ws/src/robot_description/launch/display.launch.py
```

---

### Step 4: Write the Launch Script

Copy and paste this complete script into the file:

```python
# display.launch.py
# A python script that returns a LaunchDescription containing target nodes.

from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    """
    The ROS 2 launch system calls this entry-point function.
    It must return a LaunchDescription object containing actions to execute.
    """

    # Define Node 1: robot_state_publisher
    # This node reads the URDF file and publishes it to the /robot_description topic.
    state_publisher_node = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        name='robot_state_publisher',
        output='screen'
    )

    # Define Node 2: RViz Visualizer
    # This node opens the 3D visualization dashboard.
    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        name='rviz2',
        output='screen'
    )

    # Return the LaunchDescription object containing our node configurations
    return LaunchDescription([
        state_publisher_node,
        rviz_node
    ])
```

Save and exit: press `Ctrl+X`, then `Y`, then `Enter`.

---

### Step 5: Verify the Final Package Structure

Before building, confirm your package looks like this:

```bash
ls ~/robot_ws/src/robot_description/
```

**Expected structure:**
```text
robot_description/
├── launch/
│   └── display.launch.py      ← ✅ Must end with .launch.py
├── urdf/
│   └── robot.urdf             ← (created in Module 05)
├── robot_description/
│   └── __init__.py
├── package.xml
├── setup.cfg
└── setup.py
```

---

### Step 6: Build and Run

After creating the file, perform a clean build and launch:

```bash
# Clean and rebuild
cd ~/robot_ws
rm -rf build install log
colcon build --symlink-install
source install/setup.bash

# Launch the file
ros2 launch robot_description display.launch.py
```



---

## 🔍 Understanding the Launch File API

* **`from launch import LaunchDescription`:** Imports the core launch framework class.
* **`from launch_ros.actions import Node`:** Imports the ROS 2 node manager action.
* **`generate_launch_description()`:** The entry function required by the launch parser.
* **`Node(...)` arguments:**
  * **`package`:** The name of the package containing the node executable (e.g., `rviz2`).
  * **`executable`:** The binary name of the node (e.g., `rviz2`).
  * **`name`:** Renames the running node instance inside the ROS network graph.
  * **`output='screen'`:** Directs standard outputs to print directly to your terminal.

---

*Now that we know how to run launch scripts, let's build the robot URDF model. Move to [05_urdf_intro.md](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/05_urdf_intro.md).*
