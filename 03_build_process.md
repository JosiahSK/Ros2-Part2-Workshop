# Module 03: The Build Process and Sourcing

In this module, we will learn how to compile our workspace and register packages with the ROS 2 runtime.

---

## 💻 Command 1: Navigating to the Workspace Root for Build

```bash
cd ~/robot_ws
```

### 1. Purpose
Returns the active terminal directory back to the root of the workspace (`~/robot_ws`).

### 2. Line-by-Line Explanation
* `cd`: Change Directory.
* `~/robot_ws`: Target workspace folder.

### 3. Expected Output
Terminal path updates:
```bash
user@computer:~/robot_ws$
```

### 4. Why It Is Needed
The ROS 2 build tool (`colcon`) is designed to look at the directory structure relative to the workspace root. It searches for `src` relative to the folder where it is called.

### 5. What Happens If Skipped
If you try to run a build inside `~/robot_ws/src/` or `~/robot_ws/src/robot_description/`, the build tool will throw errors because it cannot locate the correct relative paths.

### 6. Common Beginner Mistakes
* Running `colcon build` inside the `src` folder.

### 7. Real Robot Relevance
To compile changes to a robot's navigation code, you must SSH into the onboard computer, navigate to the workspace root, and run the compiler.

---

## 🧹 Command 2: Clean Build (Remove Old Artifacts)

Whenever you make significant changes — such as renaming files, editing `setup.py`, or adding new folders — always perform a **clean build** to avoid stale cache errors:

```bash
cd ~/robot_ws
rm -rf build install log
```

### 1. Purpose
Deletes the three build output directories (`build/`, `install/`, `log/`) so that the next `colcon build` compiles the entire workspace from scratch.

### 2. Line-by-Line Explanation
* `rm -rf`: Recursively and forcefully removes directories.
* `build`: Temporary compiler artifacts.
* `install`: Installed executables and share files. Deleting this forces a fresh install.
* `log`: Build log files.

### 3. Why It Is Needed
Colcon caches previous build states. If you rename a launch file (e.g., `display.launch` → `display.launch.py`) or change `setup.py`, the old cached name may still exist in `install/`. A clean build guarantees the new file name is the only one registered.

### 4. Common Beginner Mistakes
* **Skipping the clean build after renaming files:** The old `.launch` file remains in `install/share/robot_description/launch/`, and ROS 2 may still try to run it instead of the corrected `.launch.py` file.

---

## 💻 Command 3: Compiling the Workspace

```bash
colcon build --symlink-install
```

### 1. Purpose
Compiles all packages inside the `src` directory, creating executable binaries and config assets.

### 2. Line-by-Line Explanation
* `colcon`: **Collective Construction**. The standardized build tool used in ROS 2. It compiles CMake, Python, and other package formats in parallel.
* `build`: The command action to start compiling.
* `--symlink-install`: Instead of copying Python scripts and config files into `install/`, colcon creates **symbolic links** pointing back to the originals in `src/`. This means edits to Python launch files and URDF files take effect **immediately** without requiring a rebuild.

### 3. Expected Output
```text
Starting >>> robot_description
Finished <<< robot_description [1.89s]

Summary: 1 package finished [2.12s]
```

### 4. Why It Is Needed
Before building, your package is just raw code in the `src` folder. Sourcing and running packages requires files to be registered and formatted inside the `install/` directory.

### 5. What Happens If Skipped
The package remains uncompiled. Sourcing paths will not register it, and trying to launch nodes from this package will fail.

### 6. Common Beginner Mistakes
* **Running `colcon build` without ROS sourced:** If you open a fresh terminal and run `colcon build` without sourcing the global ROS setup (`source /opt/ros/humble/setup.bash`), you will see:
  `colcon: command not found`
* **Compiling unnecessary packages:** If your workspace has many packages, running a full build takes time. You can compile a single package using:
  `colcon build --packages-select robot_description`

### 7. Real Robot Relevance
When implementing a new feature (like updating camera calibration details in a URDF), you must run `colcon build` to apply the changes to the active robot drivers.

---

## 📁 Understanding the Generated Workspace Folders

After running `colcon build`, run `ls` in `~/robot_ws`. You will see three new directories:

1. **`build/`:** Temporary workspace used by the compiler. You can ignore this.
2. **`log/`:** Contains output logs for every compilation step. Helpful for debugging build failures.
3. **`install/`:** **This is the target location.** Sourced executable files, setup scripts, and configurations are moved here. Sourcing setup files targets this folder.

---

## 💻 Command 4: Sourcing the Local Workspace

```bash
source ~/robot_ws/install/setup.bash
```

### 1. Purpose
Injects your custom workspace packages into the active terminal's environmental variables, making them visible to ROS 2 command-line utilities.

### 2. Line-by-Line Explanation
* `source`: Shell command that reads and executes contents of a file inside the current terminal session.
* `install/setup.bash`: The target script generated by `colcon` inside the `install` folder.

### 3. Expected Output
This command runs silently.

### 4. Why It Is Needed
ROS 2 uses the environment variable `$AMENT_PREFIX_PATH` to resolve packages. Sourcing appends `~/robot_ws/install/` to this path variable.

### 5. What Happens If Skipped
ROS 2 commands like `ros2 launch` or `ros2 run` will return package not found errors, even if compilation was successful.

### 6. Common Beginner Mistakes
* **Sourcing from the wrong folder:** Sourcing `install/setup.bash` requires you to be in the workspace root. If you are inside `src` and run `source install/setup.bash`, it will fail because the relative path is incorrect. Use `source ~/robot_ws/install/setup.bash` to avoid this.
* **Forgetting to source new terminals:** Sourcing only affects the current terminal tab. If you open a new tab, you must source the workspace again.

### 7. Real Robot Relevance
On real robots, you configure the shell to source your active workspace automatically upon startup.

---

## 💻 Command 5: Persist Sourcing to `.bashrc` (One-Time Setup)

To avoid sourcing manually in every new terminal, append both source commands to your shell profile:

```bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
echo "source ~/robot_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 1. Purpose
Makes both the global ROS 2 environment and your custom workspace automatically active in every new terminal session.

### 2. Line-by-Line Explanation
* `echo "..." >> ~/.bashrc`: Appends the `source` command as a new line at the end of `~/.bashrc`.
* `source ~/.bashrc`: Immediately reloads `~/.bashrc` in the current terminal so the change takes effect without logging out.

### 3. Expected Output
This command runs silently. Verify it worked by opening a new terminal and running:
```bash
ros2 pkg list | grep robot_description
```
Expected output: `robot_description`

### 4. Common Beginner Mistakes
* **Running this command more than once:** Each run appends another line to `.bashrc`. Running it twice results in duplicate source lines, which is harmless but clutters the file. Check with `tail -5 ~/.bashrc`.
* **Wrong workspace path:** Ensure the path matches your actual workspace name (`robot_ws`).

---

## 🔍 Sourcing Verification Check

Let's test this behavior. Open a fresh terminal tab (unsourced) and run:

```bash
ros2 pkg list | grep robot_description
```
*Expected Output:* **Empty (Nothing is returned)**.

Now, source your workspace and run the command again:

```bash
source ~/robot_ws/install/setup.bash
ros2 pkg list | grep robot_description
```
*Expected Output:*
```text
robot_description
```

---

*Now that our package is compiled and visible, let's learn how to write a launch file. Move to [04_launch_files.md](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/04_launch_files.md).*
