# ROS 2 Development Workflow: Workspace, Package, Build, URDF and RViz

Welcome to the **ROS 2 Part 2 Workshop** repository. This workspace-ready collection of educational modules is designed specifically for students and beginners to ROS 2. 

Our goal is to build a solid foundation in the ROS 2 ecosystem by constructing a physical description (URDF) of a two-wheel differential drive (2WD) robot car and visualizing it inside RViz.

---

## 🎓 Workshop Overview
In robotics, software control loops (nodes, topics) must map to the physical geometry of the robot. This workshop takes a step-by-step approach to showing you how the ROS 2 software stack interacts with a physical robot layout. We cover:
1. Creating a clean developer workspace.
2. Generating a description package.
3. Compiling using the `colcon` build tool.
4. Structuring URDF coordinate frames (TFs) for a 2WD robot.
5. Launching and debugging the robot visual model in RViz.

---

## 📂 Repository Structure

Each file in this repository is designed as an interactive, standalone tutorial module:

* **[`01_workspace.md`](01_workspace.md):** Learn how to create and manage a clean ROS 2 directory structure.
* **[`02_package_creation.md`](02_package_creation.md):** Understand package metadata, structural layouts, and the corrected `setup.py`.
* **[`03_build_process.md`](03_build_process.md):** Explore workspace compilation, clean builds, and environment sourcing.
* **[`04_launch_files.md`](04_launch_files.md):** Write launch descriptions and create `.launch.py` files with the correct name.
* **[`05_urdf_intro.md`](05_urdf_intro.md):** Define links, joints, and geometries for a 2WD robot.
* **[`06_rviz_visualization.md`](06_rviz_visualization.md):** Visualizing coordinate transforms and visual properties.
* **[`07_2wd_robot_example.md`](07_2wd_robot_example.md):** Trace a complete, end-to-end integration mapping control nodes to physical URDF joints.
* **[`08_lidar_rplidar_a1.md`](08_lidar_rplidar_a1.md):** Connect, configure, and visualize the RP-LiDAR A1 — with full commands for both **ROS 2 Humble** and **ROS 2 Jazzy**.
* **[`commands_cheatsheet.md`](commands_cheatsheet.md):** Quick command reference sheet with expected output highlights.
* **[`exercises.md`](exercises.md):** Hands-on student challenges, solutions, and troubleshooting guides.

---

## 🛠️ Prerequisites
Before starting, ensure your system has:

**ROS 2 Humble (Ubuntu 22.04 LTS):**
* **Ubuntu 22.04 LTS** installed.
* **ROS 2 Humble Hawksbill** Desktop installation.
  ```bash
  source /opt/ros/humble/setup.bash
  ```

**ROS 2 Jazzy (Ubuntu 24.04 LTS):**
* **Ubuntu 24.04 LTS** installed.
* **ROS 2 Jazzy Jalisco** Desktop installation.
  ```bash
  source /opt/ros/jazzy/setup.bash
  ```

**Common tools (both distributions):**
```bash
# For Humble:
sudo apt-get update && sudo apt-get install -y ros-humble-urdf-parser-plugin liburdfdom-tools ros-humble-joint-state-publisher-gui

# For Jazzy:
sudo apt-get update && sudo apt-get install -y ros-jazzy-urdf-parser-plugin liburdfdom-tools ros-jazzy-joint-state-publisher-gui
```

---

*Let's get started. Open [01_workspace.md](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/01_workspace.md) to begin.*
