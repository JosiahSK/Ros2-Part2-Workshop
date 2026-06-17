# Module 02: Creating a ROS 2 Package

In this module, we will learn how to generate a ROS 2 package containing our physical robot description.

---

## 🏢 The Package Analogy: The Factory Department

Inside our **Factory** (`robot_ws`), we need specialized **Departments** to handle specific tasks:
* **`robot_description` Package:** This is the **CAD & Modeling Department**. Its sole job is to store the 3D blueprints (URDF files) and visual layout parameters.
* **`robot_control` Package (Future):** This is the **Electrical & Motor Driver Department**. It contains python scripts and nodes to drive physical actuators.
* **`robot_navigation` Package (Future):** This is the **Brain & Path Planning Department**. It handles mapping and autonomous obstacle avoidance.

By separating code into **Packages**, we ensure that engineers working on mapping don't accidentally break the low-level motor drivers.

---

## 💻 Command 1: Navigating to the Design Department (src)

```bash
cd ~/robot_ws/src
```

### 1. Purpose
Moves the active terminal context into the source directory (`src`) of your workspace.

### 2. Line-by-Line Explanation
* `cd`: Change Directory.
* `~/robot_ws/src`: The target folder where source code packages must be located.

### 3. Expected Output
Terminal path updates:
```bash
user@computer:~/robot_ws/src$
```

### 4. Why It Is Needed
ROS 2 packages **must** be created inside the `src` directory. The build system scans this folder to index packages. If you create a package in the root directory `~/robot_ws/`, the build tool will ignore it.

### 5. What Happens If Skipped
If you run the package creation command in the root folder (`~/robot_ws`), the package files will clutter the workspace root, and subsequent compilation steps will fail to find it.

### 6. Common Beginner Mistakes
* Running the package creation command from `~/robot_ws` instead of `~/robot_ws/src`.

### 7. Real Robot Relevance
Organizing all sensor drivers (LiDAR, IMU, Camera) as distinct packages under `src/` makes it easy to share them on GitHub, allowing other developers to download and compile only the modules they need.

---

## 💻 Command 2: Creating the Package Scaffolding

```bash
ros2 pkg create --build-type ament_python robot_description
```

### 1. Purpose
Generates a structured ROS 2 package skeleton named `robot_description` configured for Python-based build tools.

### 2. Line-by-Line Explanation
* `ros2`: The entry-point command-line tool for ROS 2.
* `pkg`: Command category for package-specific actions.
* `create`: Instructs ROS to generate a new package.
* `--build-type ament_python`: Configures the package to use the `ament_python` build system. This is suitable for packages containing python launch scripts, configurations, and python nodes.
* `robot_description`: The unique name of the package.

### 3. Expected Output
```text
going to create a new package
package name: robot_description
destination directory: /home/user/robot_ws/src/robot_description
package format: 3
version: 0.0.0
description: TODO: Package description
maintainer: ['user <user@todo.todo>']
licenses: ['TODO: License declaration']
build type: ament_python
creating folder robot_description
creating ./package.xml
creating ./setup.py
creating ./setup.cfg
creating ./robot_description/__init__.py
creating ./resource/robot_description
```

### 4. Why It Is Needed
Manually creating package structural files and folders from scratch is error-prone. This command guarantees that the internal package registry configurations are formatted correctly for the compiler.

### 5. What Happens If Skipped
You will not have a valid ROS 2 package, and you will not be able to write configuration code that the ROS runtime can resolve.

### 6. Common Beginner Mistakes
* **Misspelling `--build-type`:** Forgetting the double hyphens, or writing `python_ament` instead of `ament_python`.
* **Invalid characters in name:** Package names must use lowercase letters and underscores. If you name it `RobotDescription` (CamelCase) or `robot-description` (hyphenated), the command will throw an error.

### 7. Real Robot Relevance
Every sensor driver and hardware control node on a real robot is packaged this way. Clear package division allows modular assembly of complex robot software stacks.

---

## 📁 Understanding the Generated Structure

Let's look at the generated files inside `src/robot_description/`:

1. **`package.xml`:** The package index file. It lists package details, maintainers, and runtime dependencies. If your robot description depends on another package (like `gazebo_ros`), you must declare it here.
2. **`setup.py`:** A Python build script. It specifies which files and scripts (like URDFs and launch files) must be installed to the system target directory when compiling.
3. **`setup.cfg`:** Simple configuration metadata specifying python script installation directories.
4. **`resource/robot_description`:** A marker file used by the ROS 2 index system to locate the package quickly.
5. **`robot_description/` (Subfolder):** A Python package directory. If you write custom python helper nodes, they are placed in this folder.

---

## ✏️ Correcting `setup.py` — Installing URDF and Launch Files

The default `setup.py` generated by `ros2 pkg create` does **not** install `urdf/` or `launch/` files. You must edit it manually to include those data directories.

Open `setup.py` for editing:
```bash
nano ~/robot_ws/src/robot_description/setup.py
```

Replace the entire file content with the corrected version below:

```python
from setuptools import find_packages, setup
import os
from glob import glob

package_name = 'robot_description'

setup(
    name=package_name,
    version='0.0.0',

    packages=find_packages(exclude=['test']),

    data_files=[
        # ROS 2 package index
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),

        # package.xml install
        ('share/' + package_name, ['package.xml']),

        # URDF files
        (os.path.join('share', package_name, 'urdf'),
            glob('urdf/*')),

        # Launch files (.launch.py only)
        (os.path.join('share', package_name, 'launch'),
            glob('launch/*.launch.py')),
    ],

    install_requires=['setuptools'],
    zip_safe=True,

    maintainer='evolve06',
    maintainer_email='evolve06@todo.todo',

    description='Robot description package with URDF and launch files',
    license='Apache-2.0',

    extras_require={
        'test': ['pytest'],
    },

    entry_points={
        'console_scripts': [],
    },
)
```

### Key Changes Explained

| Addition | Why It Is Needed |
|----------|------------------|
| `import os` | Required to use `os.path.join()` for cross-platform paths |
| `from glob import glob` | Required to use wildcard file patterns |
| `glob('urdf/*')` | Installs all files in the `urdf/` folder to `share/robot_description/urdf/` |
| `glob('launch/*.launch.py')` | Installs only `.launch.py` files (not misnamed `.launch` files) |

> **⚠️ Important:** After editing `setup.py`, always perform a **clean rebuild**:
> ```bash
> cd ~/robot_ws
> rm -rf build install log
> colcon build --symlink-install
> source install/setup.bash
> ```

---

*Now let's learn how to compile this package. Navigate to [03_build_process.md](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/03_build_process.md).*
