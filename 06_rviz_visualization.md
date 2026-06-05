# Module 06: RViz Visualization

In this module, we will learn how to load our robot model into RViz and verify its visual coordinate frames.

---

## 🖥️ What is RViz?

**RViz (Robot Visualization)** is a tool that allows you to see the robot model and sensor data inside a unified 3D virtual environment. It reads:
* The robot's structural layout (`/robot_description` topic).
* Joint positions (provided by sensor feedback or GUIs).
* Transform frames (TF trees showing link positions relative to each other).
* Raw sensor streams (LiDAR laser scans, point clouds, camera feeds).

---

## ✍️ Creating the Display Launch File

Before running RViz, we need to write a launch file that loads our URDF XML file, passes it to the `robot_state_publisher` node, and starts RViz. 

Inside `launch/display.launch.py`, update the code to match this configuration:

```python
# display.launch.py
# Standard visualization template for ROS 2 Humble.

import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    # 1. Resolve package installation directory
    package_name = 'robot_description'
    pkg_share = get_package_share_directory(package_name)

    # 2. Get target paths for the URDF
    urdf_path = os.path.join(pkg_share, 'urdf', 'robot.urdf')

    # 3. Read the XML content
    with open(urdf_path, 'r') as file:
        robot_urdf_content = file.read()

    # Define Node 1: robot_state_publisher
    # Reads the URDF string and publishes to /robot_description and TF frames.
    robot_state_publisher_node = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        name='robot_state_publisher',
        output='screen',
        parameters=[{
            'robot_description': robot_urdf_content
        }]
    )

    # Define Node 2: joint_state_publisher_gui
    # Provides slider UI elements to rotate continuous joint frames.
    joint_state_publisher_gui_node = Node(
        package='joint_state_publisher_gui',
        executable='joint_state_publisher_gui',
        name='joint_state_publisher_gui',
        output='screen'
    )

    # Define Node 3: rviz2
    # Renders the actual 3D interface.
    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        name='rviz2',
        output='screen'
    )

    return LaunchDescription([
        robot_state_publisher_node,
        joint_state_publisher_gui_node,
        rviz_node
    ])
```

---

## 💻 Command: Sourcing the Launch File

First, ensure your workspace is compiled:
```bash
cd ~/robot_ws
colcon build --symlink-install
source install/setup.bash
```

Now, launch the visualizer:
```bash
ros2 launch robot_description display.launch.py
```

### 1. Purpose
Launches `robot_state_publisher`, `joint_state_publisher_gui`, and `rviz2` in parallel.

### 2. Line-by-Line Explanation
* `ros2 launch`: Runs a ROS 2 launch file.
* `robot_description`: Package containing our launch script.
* `display.launch.py`: The name of the launch script.

### 3. Expected Output
Your terminal prompt will output startup logs, and **two window displays will open**:
1. The **RViz** window.
2. The **joint_state_publisher_gui** slider window.

---

## 🛠️ Step-by-Step RViz Configuration Guide

When RViz opens, it starts with an empty layout. Follow these steps to configure your displays:

```
+───────────────────────────────────────────────────────────+
| Displays Panel               3D Canvas Viewport           |
|                                                           |
| 1. Fixed Frame: base_link       +----------------+        |
|                                 |                |        |
| 2. Add -> RobotModel            |  Chassis Box   |        |
|                                 |  (Blue)        |        |
| 3. Add -> TF                    +----------------+        |
|                                     O        O            |
|                                 (Wheels) (Wheels)         |
+───────────────────────────────────────────────────────────+
```

1. **Configure the Fixed Frame:**
   * In the left **Displays** panel, locate the `Global Options` section.
   * Change `Fixed Frame` from `map` to `base_link` (or `base_footprint`).
   * *If skipped:* The robot model will show error status because RViz doesn't know what coordinate frame to use as the center origin.
2. **Add the Robot Visual Model:**
   * Click the **Add** button in the bottom-left corner of the Displays panel.
   * Scroll down, select **RobotModel**, and click **OK**.
   * *Expected Result:* The blue rectangular chassis and the black wheels will render on the grid canvas.
3. **Add Coordinate Frames (TF Trees):**
   * Click **Add** again.
   * Select **TF** from the list and click **OK**.
   * *Expected Result:* A colored axis marker (Red X, Green Y, Blue Z) will appear at the center of the chassis, wheels, and sensor links.
4. **Move the Joint Sliders:**
   * Find the small slider window labeled `joint_state_publisher_gui`.
   * Drag the slider handles. You should see the wheels rotate around their axes in RViz.

---

## ⚠️ Common Beginner Mistakes

* **RViz remains empty or shows errors:** Check the `Fixed Frame` parameter. If it is set to `map`, change it to `base_link`.
* **Robot looks white or uncolored:** Check the URDF file for typos in material declarations, or ensure you added the `RobotModel` display in the RViz displays panel.

---

*Now that we have successfully visualized our robot model, let's look at how all these concepts map to a real robot stack. Move to [07_2wd_robot_example.md](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/07_2wd_robot_example.md).*
