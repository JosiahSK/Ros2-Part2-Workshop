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

* **[`01_workspace.md`](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/01_workspace.md):** Learn how to create and manage a clean ROS 2 directory structure.
* **[`02_package_creation.md`](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/02_package_creation.md):** Understand package metadata and structural layouts.
* **[`03_build_process.md`](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/03_build_process.md):** Explore workspace compilation and environment sourcing.
* **[`04_launch_files.md`](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/04_launch_files.md):** Write launch descriptions to run multiple nodes together.
* **[`05_urdf_intro.md`](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/05_urdf_intro.md):** Define links, joints, and geometries for a 2WD robot.
* **[`06_rviz_visualization.md`](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/06_rviz_visualization.md):** Visualizing coordinate transforms and visual properties.
* **[`07_2wd_robot_example.md`](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/07_2wd_robot_example.md):** Trace a complete, end-to-end integration mapping control nodes to physical URDF joints.
* **[`commands_cheatsheet.md`](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/commands_cheatsheet.md):** Quick command reference sheet with expected output highlights.
* **[`exercises.md`](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/exercises.md):** Hands-on student challenges, solutions, and troubleshooting guides.

---

## 🛠️ Prerequisites
Before starting, ensure your system has:
* **Ubuntu 22.04 LTS** (or equivalent environment).
* **ROS 2 Humble Hawksbill** (Desktop installation containing RViz2).
* **urdfdom-headers** verification tools. Install via:
  ```bash
  sudo apt-get update && sudo apt-get install -y ros-humble-urdf-parser-plugin liburdfdom-tools
  ```

---

*Let's get started. Open [01_workspace.md](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/01_workspace.md) to begin.*
