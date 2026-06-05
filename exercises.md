# Student Lab Exercises & Troubleshooting Guide

Complete these exercises to apply the concepts covered in the workshop.

---

## 📝 Exercise 1: Create a Workspace

### Goal
Create a fresh workspace named `experimental_ws` with a source folder.

### Expected Answer
```bash
mkdir -p ~/experimental_ws/src
```

### Expected Output
Confirm the folder layout by running:
```bash
ls ~/experimental_ws
```
*Console output:*
```text
src
```

### 💡 Troubleshooting
* **Error:** `mkdir: cannot create directory ‘/home/user/experimental_ws/src’: No such file or directory`
  * **Cause:** You forgot the `-p` parent flag.
  * **Fix:** Re-run the command with the `-p` flag: `mkdir -p ~/experimental_ws/src`

---

## 📝 Exercise 2: Create a Package

### Goal
Navigate to your new workspace's source directory and create an `ament_python` package named `my_robot_description`.

### Expected Answer
```bash
cd ~/experimental_ws/src
ros2 pkg create --build-type ament_python my_robot_description
```

### Expected Output
```text
going to create a new package
package name: my_robot_description
...
creating ./package.xml
creating ./setup.py
```

### 💡 Troubleshooting
* **Error:** `ros2: command not found`
  * **Cause:** Your terminal is not sourced with the global ROS 2 Humble setup script.
  * **Fix:** Run `source /opt/ros/humble/setup.bash` and retry.

---

## 📝 Exercise 3: Build Workspace

### Goal
Compile your new workspace from the correct root directory.

### Expected Answer
```bash
cd ~/experimental_ws
colcon build
```

### Expected Output
```text
Starting >>> my_robot_description
Finished <<< my_robot_description [1.80s]

Summary: 1 package finished [2.00s]
```

### 💡 Troubleshooting
* **Error:** `colcon: command not found`
  * **Cause:** `colcon` is not installed or your global environment is unsourced.
  * **Fix:** Run `sudo apt update && sudo apt install -y python3-colcon-common-extensions` to install the build tool.

---

## 📝 Exercise 4: Source Workspace

### Goal
Source your new workspace and verify that your package `my_robot_description` is visible to the system.

### Expected Answer
```bash
cd ~/experimental_ws
source install/setup.bash
ros2 pkg list | grep my_robot_description
```

### Expected Output
```text
my_robot_description
```

### 💡 Troubleshooting
* **Error:** `grep: command not found` (or empty line returned)
  * **Cause:** You ran `source install/setup.bash` from inside `src/` instead of the workspace root.
  * **Fix:** Navigate to the root: `cd ~/experimental_ws` and run `source install/setup.bash` again.

---

## 📝 Exercise 5: Create a Launch Folder

### Goal
Create `urdf/`, `launch/`, and `rviz/` directories inside your package.

### Expected Answer
```bash
cd ~/experimental_ws/src/my_robot_description
mkdir urdf launch rviz
```

### Expected Output
Running `ls` should list:
```text
launch  my_robot_description  package.xml  resource  rviz  setup.cfg  setup.py  urdf
```

---

## 📝 Exercise 6: Create a Simple URDF

### Goal
Create a URDF file named `minimal.urdf` inside your package's `urdf` folder containing a single rectangular box link named `main_body`.

### Expected Answer
```bash
touch urdf/minimal.urdf
```
Open `urdf/minimal.urdf` and paste the following XML:
```xml
<?xml version="1.0"?>
<robot name="minimal_robot">
  <material name="green">
    <color rgba="0.0 0.8 0.0 1.0"/>
  </material>
  
  <link name="main_body">
    <visual>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <box size="0.2 0.2 0.2"/>
      </geometry>
      <material name="green"/>
    </visual>
  </link>
</robot>
```
Validate your file structure:
```bash
check_urdf urdf/minimal.urdf
```

### Expected Output
```text
robot name is: minimal_robot
---------- Successfully Parsed XML ----------
root Link: main_body has 0 child(ren)
```

---

## 📝 Exercise 7: Launch Robot in RViz

### Goal
Write a launch file `minimal.launch.py` to display the robot in RViz, build your workspace, and launch it.

### Expected Answer
1. Create `launch/minimal.launch.py` with this code:
```python
import os
from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch_ros.actions import Node

def generate_launch_description():
    pkg_share = get_package_share_directory('my_robot_description')
    urdf_path = os.path.join(pkg_share, 'urdf', 'minimal.urdf')
    with open(urdf_path, 'r') as file:
        urdf_content = file.read()

    return LaunchDescription([
        Node(
            package='robot_state_publisher',
            executable='robot_state_publisher',
            parameters=[{'robot_description': urdf_content}]
        ),
        Node(
            package='rviz2',
            executable='rviz2'
        )
    ])
```
2. Edit `setup.py` to copy these assets during the build process:
```python
# Add this inside the data_files list in setup.py:
(os.path.join('share', package_name, 'urdf'), glob('urdf/*')),
(os.path.join('share', package_name, 'launch'), glob('launch/*')),
```
3. Compile and launch:
```bash
cd ~/experimental_ws
colcon build --symlink-install
source install/setup.bash
ros2 launch my_robot_description minimal.launch.py
```

### Expected Output
* RViz opens.
* Once the `Fixed Frame` is set to `main_body` and the **RobotModel** display is added, a green cube appears centered on the origin grid.

### 💡 Troubleshooting
* **Error:** `Package 'my_robot_description' not found`
  * **Cause:** You edited `setup.py` but forgot to compile the changes using `colcon build`.
  * **Fix:** Re-run `colcon build --symlink-install` and source the workspace.
