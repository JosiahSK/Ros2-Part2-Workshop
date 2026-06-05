# Module 05: Introduction to URDF

In this module, we will learn how to write a Unified Robot Description Format (URDF) file to describe our physical 2WD robot car.

---

## 🤖 What is a URDF?

**URDF (Unified Robot Description Format)** is an XML representation used to define the physical layout of a robot's body. In ROS, we model the robot as a tree of rigid parts (**Links**) connected by moving or static elements (**Joints**).

### ASCII Kinematic Tree Diagram
```
                     [ Link: base_footprint ]  (Ground Project Floor)
                               │
                       [ Joint: base_joint ]   (Fixed)
                               │
                               v
                       [ Link: base_link ]     (Chassis Box)
                     /         |         \
                    /          |          \
          (left_joint)   (right_joint)   (sensor_joint)
           [Continuous]   [Continuous]      [Fixed]
              /                |                \
             v                 v                 v
     [Link: left_wheel] [Link: right_wheel] [Link: ultrasonic_link]
```

### Key Structural Concepts:
* **Link:** A rigid body. It defines visual properties (colors, 3D meshes), collision boundaries, and inertial properties (mass, center of mass).
* **Joint:** The connection between two links. It defines how the child link moves relative to the parent link.
  * **Parent Link:** The reference link upstream.
  * **Child Link:** The link downstream that moves relative to the parent.
* **Coordinate origin (`xyz` / `rpy`):** Specifies where a link's frame is offset relative to its parent frame.
  * `xyz`: Translation offsets in meters along X (forward), Y (left), and Z (up) axes.
  * `rpy`: Rotation offsets in radians (Roll, Pitch, Yaw).

---

## 💻 Command 1: Navigating to the Package Directory

```bash
cd ~/robot_ws/src/robot_description
```

### 1. Purpose
Moves the active terminal context into the root directory of your `robot_description` package.

### 2. Line-by-Line Explanation
* `cd`: Change Directory.
* `~/robot_ws/src/robot_description`: Path to the package location.

### 3. Expected Output
Terminal path updates:
```bash
user@computer:~/robot_ws/src/robot_description$
```

### 4. Why It Is Needed
You must be inside the package directory to create relative asset folders (like `urdf/`) and build configurations without cluttering your system directories.

### 5. What Happens If Skipped
You will create the URDF file in whatever folder your terminal is currently pointing to, making it invisible to the package setup configurations.

### 6. Common Beginner Mistakes
* Running the command without compiling the workspace first. If the directory is typed incorrectly, it returns: `cd: no such file or directory`.

### 7. Real Robot Relevance
To debug robot physical parameters, developers must navigate directly to the description package on the onboard computer to inspect local URDF configs.

---

## 💻 Command 2: Creating the Empty URDF File

```bash
touch urdf/robot.urdf
```

### 1. Purpose
Creates a new, blank file named `robot.urdf` inside your package's `urdf` folder.

### 2. Line-by-Line Explanation
* `touch`: System command to create empty files or update timestamps.
* `urdf/robot.urdf`: Target directory and file name.

### 3. Expected Output
This command runs silently. Running `ls urdf` will confirm the file creation:
```bash
ls urdf
```
*Expected Output:*
```text
robot.urdf
```

### 4. Why It Is Needed
The file must exist on the disk before we can open it in text editors to write XML nodes.

### 5. What Happens If Skipped
The text editor will have no target file to open, and you cannot write the robot's description.

### 6. Common Beginner Mistakes
* **Running the command before creating the `urdf` folder:** If you skip creating the directory, it returns: `touch: cannot touch 'urdf/robot.urdf': No such file or directory`.
* **Typing the wrong path:** Ensure you are in the package root.

### 7. Real Robot Relevance
When adding new hardware accessories (like a GPS antenna or camera mount), you create new URDF or Xacro files in this folder to maintain clean physical configuration models.

---

## 💻 Command 3: Opening the URDF in a Text Editor

```bash
nano urdf/robot.urdf
```

### 1. Purpose
Opens the command-line text editor `nano` to edit the empty `robot.urdf` file.

### 2. Line-by-Line Explanation
* `nano`: A simple, built-in terminal text editor.
* `urdf/robot.urdf`: The target file to modify.

### 3. Expected Output
The terminal view switches to the nano editor screen:
```text
  GNU nano 6.2                   urdf/robot.urdf                             
[ Paste URDF XML here ]
^G Help      ^O WriteOut  ^R Read File ^Y Prev Pg   ^K Cut       ^C Cur Pos
^X Exit      ^J Justify   ^W Where Is  ^V Next Pg   ^U Paste     ^T To Spell
```

### 4. Why It Is Needed
It allows you to paste or write the robot's physical layout XML directly from the terminal without needing a graphical desktop interface.

### 5. What Happens If Skipped
The file remains empty, and the robot description topic will contain nothing.

### 6. Common Beginner Mistakes
* **Not knowing how to save and exit:** In `nano`, you press **Ctrl+O** then **Enter** to save (WriteOut), and **Ctrl+X** to exit. 

### 7. Real Robot Relevance
Most industrial robots run headless (no screen, keyboard, or mouse). Developers configure them using SSH terminal editors like `nano` or `vim`.

---

## ✍️ The Complete 2WD Robot URDF Code

*Open `nano urdf/robot.urdf` as described above, paste this entire XML configuration, save (Ctrl+O, Enter), and exit (Ctrl+X):*

```xml
<?xml version="1.0"?>
<!-- robot.urdf -->
<robot name="two_wheel_robot">

  <!-- Color Materials -->
  <material name="blue">
    <color rgba="0.1 0.2 0.8 1.0"/>
  </material>
  <material name="black">
    <color rgba="0.1 0.1 0.1 1.0"/>
  </material>
  <material name="red">
    <color rgba="0.8 0.1 0.1 1.0"/>
  </material>

  <!-- Root Link: ground level projection -->
  <link name="base_footprint"/>

  <!-- Joint linking base_footprint to base_link -->
  <joint name="base_joint" type="fixed">
    <parent link="base_footprint"/>
    <child link="base_link"/>
    <!-- Raise chassis bottom 5cm off the ground -->
    <origin xyz="0 0 0.05" rpy="0 0 0"/>
  </joint>

  <!-- Chassis Link -->
  <link name="base_link">
    <visual>
      <!-- Center of box geometry raised to sit on top of bottom clearance -->
      <origin xyz="0 0 0.075" rpy="0 0 0"/>
      <geometry>
        <!-- 40cm long, 30cm wide, 15cm high box -->
        <box size="0.4 0.3 0.15"/>
      </geometry>
      <material name="blue"/>
    </visual>
  </link>

  <!-- Left Wheel Link -->
  <link name="left_wheel">
    <visual>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <!-- Cylinder representing the tire: radius 10cm, width 5cm -->
        <cylinder radius="0.1" length="0.05"/>
      </geometry>
      <material name="black"/>
    </visual>
  </link>

  <!-- Joint left wheel -->
  <joint name="left_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="left_wheel"/>
    <!-- Position wheel on chassis left side, slightly forward of center.
         Rotate Roll -90 degrees (-1.570796 rad) to flip vertical cylinder horizontal. -->
    <origin xyz="0.1 0.175 0.0" rpy="-1.570796 0 0"/>
    <axis xyz="0 0 1"/>
  </joint>

  <!-- Right Wheel Link -->
  <link name="right_wheel">
    <visual>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <cylinder radius="0.1" length="0.05"/>
      </geometry>
      <material name="black"/>
    </visual>
  </link>

  <!-- Joint right wheel -->
  <joint name="right_wheel_joint" type="continuous">
    <parent link="base_link"/>
    <child link="right_wheel"/>
    <!-- Position wheel on chassis right side. -->
    <origin xyz="0.1 -0.175 0.0" rpy="-1.570796 0 0"/>
    <axis xyz="0 0 1"/>
  </joint>

  <!-- Ultrasonic Sensor Link -->
  <link name="ultrasonic_link">
    <visual>
      <origin xyz="0 0 0" rpy="0 0 0"/>
      <geometry>
        <!-- Small sensor box: 4cm x 8cm x 4cm -->
        <box size="0.04 0.08 0.04"/>
      </geometry>
      <material name="red"/>
    </visual>
  </link>

  <!-- Joint sensor -->
  <joint name="ultrasonic_joint" type="fixed">
    <parent link="base_link"/>
    <child link="ultrasonic_link"/>
    <!-- Mount to the front edge (X=0.2m), centered (Y=0.0), 5cm high (Z=0.05m) -->
    <origin xyz="0.2 0.0 0.05" rpy="0 0 0"/>
  </joint>

</robot>
```

---

## 💻 Command 4: Validating the URDF Structure

```bash
check_urdf urdf/robot.urdf
```

### 1. Purpose
Parses the URDF file to verify its XML syntax, joint links, and coordinate connections.

### 2. Line-by-Line Explanation
* `check_urdf`: A ROS command-line utility used to parse and validate URDF files.
* `urdf/robot.urdf`: Path to the target file.

### 3. Expected Output
```text
robot name is: two_wheel_robot
---------- Successfully Parsed XML ----------
root Link: base_footprint has 1 child(ren)
    child(1):  base_link
        child(1):  left_wheel
        child(2):  right_wheel
        child(3):  ultrasonic_link
```

### 4. Why It Is Needed
XML syntax errors (such as missing brackets or tag typos) can crash your visualization scripts. Running this tool catches structural mistakes instantly.

### 5. What Happens If Skipped
You will launch your ROS nodes without knowing if the URDF is correctly written, resulting in visualization errors or node crashes that are hard to diagnose.

### 6. Common Beginner Mistakes
* **Parsing with unclosed tags:** If your URDF has a typo, the parser will fail with a message like: `Error: XML parsing error...` or `fatal: link 'base_link' has no parent`.

### 7. Real Robot Relevance
Before compiling packages on real robots, developers parse models through `check_urdf` to ensure the transform links form a valid, non-circular kinematic tree.

---

## 🔍 Line-by-Line URDF Code Explanation

* **`<robot name="two_wheel_robot">`:** Defines the start of the robot description layout.
* **`<material name="blue">`:** Declares a color variable to reuse in visual definitions. Colors use `rgba` (Red, Green, Blue, Alpha) format, with values from `0.0` to `1.0`.
* **`base_footprint`:** A virtual placeholder link representing ground level. Having a virtual link makes it easy to map navigation coordinates.
* **`base_joint`:** A `fixed` joint because the chassis doesn't move relative to the ground projection plane.
* **`<box size="0.4 0.3 0.15"/>`:** Creates a rectangular solid shape where parameters specify length along X, width along Y, and height along Z.
* **`<cylinder radius="0.1" length="0.05"/>`:** Creates a cylindrical shape representing a wheel. Cylinders align along their local Z-axis by default.
* **`rpy="-1.570796 0 0"`:** To make the wheel roll forward, we must align the cylinder's axis with the robot's lateral axis (Y). Rotating the joint's Roll by -90 degrees flips the cylinder horizontal.
* **`<axis xyz="0 0 1"/>`:** Specifies that the wheel rotates around the local Z-axis (which now points along the lateral axis).

---

*Now that we have written the URDF, let's learn how to load and view it in RViz. Move to [06_rviz_visualization.md](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/06_rviz_visualization.md).*
