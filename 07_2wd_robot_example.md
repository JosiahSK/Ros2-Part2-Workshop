# Module 07: Real 2WD Robot Integration

In this module, we will explore how our URDF file fits together with the software nodes we built in previous classes.

---

## 🏎️ Complete 2WD Robot Integration architecture

Let's look at the software architecture of our robot. We have three nodes communicating via topics:

```text
                      [ PHYSICAL ROBOT CONTROLLER ]
                                   │
                                   v
  +───────────────────+   (topic)  +───────────────────+   (topic)  +───────────────────+
  |  Ultrasonic Node  | ─────────> |   Decision Node   | ─────────> |    Motor Node     |
  | (Hardware Driver) |  /distance | (Obstacle Avoid)  |  /cmd_vel  | (Hardware Driver) |
  +───────────────────+            +───────────────────+            +───────────────────+
```

1. **Ultrasonic Node:** Reads raw sensor data and publishes the measured distance to the `/distance` topic.
2. **Decision Node:** Subscribes to `/distance`. If the distance is less than the safety threshold, it publishes a stop command to the `/cmd_vel` topic.
3. **Motor Node:** Subscribes to `/cmd_vel` and translates velocity requests into voltage signals for the motors.

---

## 🗺️ ROS Concepts to Robot Hardware Mapping Table

To help you understand how these concepts relate to a physical robot, let's map them to their hardware equivalents:

| ROS Concept | Robot Hardware Example | Description |
| :--- | :--- | :--- |
| **Node** | `ultrasonic_node` | A software process running on the robot's CPU (e.g., Raspberry Pi) that reads sensor data from GPIO pins. |
| **Topic** | `/distance` | A data channel where the sensor driver publishes measurements. |
| **Publisher** | `ultrasonic_node` | The driver code sending raw sensor measurements. |
| **Subscriber** | `decision_node` | The navigation script listening to sensor topics to make movement decisions. |
| **Message** | `sensor_msgs/msg/Range` | The standard data structure containing distance, field of view, and range limits. |
| **Service** | `/calibrate_sensor` | A request-response trigger used to calibrate the sensor's baseline. |
| **Parameter** | `safety_distance = 0.2` | A configurable variable (like stopping threshold) stored in memory. |
| **Package** | `robot_description` | The directory containing our URDF blueprint and launch scripts. |
| **Workspace** | `robot_ws` | The main folder on the robot's filesystem containing all your compiled code. |

---

## 🔗 Connecting URDF Transforms to Sensor Data

Why do we need a URDF when we already have the software nodes above?

Imagine the `ultrasonic_node` publishes a distance of `0.15 meters`. The decision node needs to know: **"Where is this reading measured from?"**

By referencing the **Transform Tree (TF)** published by `robot_state_publisher` from our URDF, the system knows that `ultrasonic_link` is offset exactly `0.2 meters` forward of `base_link`. 

```text
        [base_link Frame]              [ultrasonic_link Frame]      Detected Wall
             (0, 0)                        (X = 0.2m)                  (0.15m away)
               +────────────────────────────────+─────────────────────────|
               |<───── URDF Joint Offset ──────>|<── Sensor distance ────>|
               |                                |        reading          |
               |<──────────────── Total Distance ───────────────────────>|
                                          (0.35m)
```

The system automatically calculates the distance relative to the robot's center:
$$\text{Total Distance} = \text{URDF Joint Offset} + \text{Sensor Reading} = 0.2\text{m} + 0.15\text{m} = 0.35\text{m}$$

This coordinate mapping is essential for path planning, navigation, and obstacle avoidance.

---

## 🚀 Running Verification Command: Check Topic State

To verify that your nodes are publishing the correct sensor coordinates, you can inspect the active topics.

```bash
ros2 topic echo /robot_description --once
```

### 1. Purpose
Reads and outputs the published URDF string from the `/robot_description` topic.

### 2. Line-by-Line Explanation
* `ros2 topic echo`: Command used to print messages published to a topic.
* `/robot_description`: The target topic containing the URDF string.
* `--once`: Instructs the command to print a single message and exit.

### 3. Expected Output
Prints your complete URDF XML file configuration:
```xml
<robot name="two_wheel_robot">
  <material name="blue">
    <color rgba="0.1 0.2 0.8 1.0"/>
  </material>
  ...
</robot>
```

### 4. Why It Is Needed
It allows you to verify that `robot_state_publisher` is successfully broadcasting the URDF configuration to other nodes.

### 5. What Happens If Skipped
You cannot verify what model configuration the ROS system has loaded in memory.

### 6. Common Beginner Mistakes
* Running the command without starting the launch file first. If the launch file is not running, the command will hang because no node is publishing to `/robot_description`.

### 7. Real Robot Relevance
When debugging a robot on the field, you echo `/robot_description` or `/tf` to verify that all sensor mounts and joint transforms are reporting correct, calibrated offsets.

---

*Now let's review the commands we used in this workshop. Move to [commands_cheatsheet.md](file:///d:/Documents/Antigravity_project/ros2-day3-workshop/commands_cheatsheet.md).*
