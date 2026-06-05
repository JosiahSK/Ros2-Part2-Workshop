# ROS 2 Workspace & URDF Command Cheatsheet

Use this quick reference cheatsheet during the workshop exercises.

---

## 📋 Commands Reference Table

| Command | Working Directory | One-Sentence Explanation |
| :--- | :--- | :--- |
| `mkdir -p ~/robot_ws/src` | `~/` | Creates the workspace folder structure (`robot_ws/src`) in a single command. |
| `cd ~/robot_ws` | Any | Navigates the terminal to your active workspace root directory. |
| `ros2 pkg create --build-type ament_python robot_description` | `~/robot_ws/src` | Generates a ROS 2 Python package template named `robot_description` inside the source directory. |
| `colcon build` | `~/robot_ws` | Compiles all ROS 2 packages inside the workspace source folder. |
| `colcon build --symlink-install` | `~/robot_ws` | Compiles the workspace using symbolic links so editing launch or configuration files takes effect without recompiling. |
| `source install/setup.bash` | `~/robot_ws` | Registers compiled packages in the current terminal environment. |
| `ros2 pkg list` | Any | Lists all compiled and installed ROS 2 packages visible in your environment. |
| `check_urdf urdf/robot.urdf` | `~/robot_ws/src/robot_description` | Validates the syntax of your URDF XML file and displays the parsed kinematic tree. |
| `ros2 launch robot_description display.launch.py` | `~/robot_ws` | Executes the launch script to start the robot state publisher, joint state GUI, and RViz visualizer. |
| `ros2 topic echo /robot_description --once` | Any | Prints the active URDF XML configuration currently loaded in the ROS system. |

---

## ⚠️ Environmental Reminders
* **Sourcing is Local:** Sourcing (`source install/setup.bash`) only registers packages for the **current terminal window**. If you open a new tab, you must source the environment again.
* **Build from Root:** Always ensure your working directory (`pwd`) is `~/robot_ws` before running `colcon build`.
