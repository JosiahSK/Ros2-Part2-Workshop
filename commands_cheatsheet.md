# ROS 2 Workspace & URDF Command Cheatsheet

Use this quick reference cheatsheet during the workshop exercises.

---

## 📋 Commands Reference Table

| Command | Working Directory | One-Sentence Explanation |
| :--- | :--- | :--- |
| `mkdir -p ~/robot_ws/src` | `~/` | Creates the workspace folder structure (`robot_ws/src`) in a single command. |
| `cd ~/robot_ws` | Any | Navigates the terminal to your active workspace root directory. |
| `ros2 pkg create --build-type ament_python robot_description` | `~/robot_ws/src` | Generates a ROS 2 Python package template named `robot_description` inside the source directory. |
| `nano ~/robot_ws/src/robot_description/setup.py` | Any | Opens `setup.py` for editing to add `urdf/` and `launch/` data file install paths. |
| `mv launch/display.launch launch/display.launch.py` | `~/robot_ws/src/robot_description` | Renames the launch file to the `.launch.py` extension required by ROS 2. |
| `sudo apt install ros-humble-joint-state-publisher-gui` | Any | Installs the joint state GUI node used by `display.launch.py` to control robot joints interactively. |
| `rm -rf build install log` | `~/robot_ws` | Removes all cached build artifacts to force a full clean rebuild. |
| `colcon build --symlink-install` | `~/robot_ws` | Compiles the workspace using symbolic links so editing launch or URDF files takes effect without recompiling. |
| `source ~/robot_ws/install/setup.bash` | Any | Registers compiled packages in the current terminal environment. |
| `echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc` | Any | Persists ROS 2 global sourcing across all future terminal sessions. |
| `echo "source ~/robot_ws/install/setup.bash" >> ~/.bashrc` | Any | Persists workspace sourcing across all future terminal sessions. |
| `source ~/.bashrc` | Any | Reloads the shell profile immediately in the current terminal. |
| `ros2 pkg list` | Any | Lists all compiled and installed ROS 2 packages visible in your environment. |
| `check_urdf urdf/robot.urdf` | `~/robot_ws/src/robot_description` | Validates the syntax of your URDF XML file and displays the parsed kinematic tree. |
| `ros2 launch robot_description display.launch.py` | `~/robot_ws` | Executes the launch script to start the robot state publisher, joint state GUI, and RViz visualizer. |
| `ros2 topic echo /robot_description --once` | Any | Prints the active URDF XML configuration currently loaded in the ROS system. |

---

## 🔧 Fix Sequence — After Renaming `display.launch` → `display.launch.py`

If your launch file was originally created without the `.py` extension, run this full fix sequence:

```bash
# Step 1: Rename the launch file
mv ~/robot_ws/src/robot_description/launch/display.launch \
   ~/robot_ws/src/robot_description/launch/display.launch.py

# Step 2: Install missing dependency
sudo apt install ros-humble-joint-state-publisher-gui

# Step 3: Edit setup.py to use glob('launch/*.launch.py')
nano ~/robot_ws/src/robot_description/setup.py

# Step 4: Clean rebuild
cd ~/robot_ws
rm -rf build install log
colcon build --symlink-install
source install/setup.bash

# Step 5: Persist sourcing (one-time)
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
echo "source ~/robot_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

---

## ⚠️ Environmental Reminders
* **Sourcing is Local:** Sourcing only registers packages for the **current terminal window**. Open a new tab? Source again — or add it to `~/.bashrc` permanently.
* **Build from Root:** Always ensure your working directory (`pwd`) is `~/robot_ws` before running `colcon build`.
* **Clean After Changes:** After editing `setup.py` or renaming files, always run `rm -rf build install log` before rebuilding.
