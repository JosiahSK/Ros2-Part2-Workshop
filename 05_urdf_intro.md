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

## ✍️ Writing a 2WD Robot URDF

Let's write a simple URDF for our robot. Inside `urdf/robot.urdf`, copy and paste the XML structure below:

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
