# Interfaces

## Command topics

Both commander nodes (`my_robot_commander_py` and `my_robot_commander_cpp`)
expose the same three subscriptions. Topic names are relative, so they resolve
against the node namespace.

| Topic | Type | Payload | Effect |
| --- | --- | --- | --- |
| `open_gripper` | `example_interfaces/msg/Bool` | `data: true` / `false` | Plans and executes `gripper_open` / `gripper_closed`. |
| `joint_command` | `example_interfaces/msg/Float64MultiArray` | exactly 6 values | Plans and executes the joint target `[joint1 … joint6]` (rad). |
| `pose_command` | `my_robot_interfaces/msg/PoseCommand` | `x, y, z, roll, pitch, yaw, cartesian_path` | Plans a Cartesian goal for `tool_link` in `base_link`; RPY in radians. |

Notes:

- `joint_command` is silently ignored unless it carries exactly six values.
- `pose_command` currently ignores `cartesian_path` and always performs a
  free-space plan to the pose.
- Goals are executed as soon as they arrive; there is no queueing, preemption
  policy or goal feedback topic yet.

### Examples

Open the gripper:

```bash
ros2 topic pub --once /open_gripper example_interfaces/msg/Bool "{data: true}"
```

Move to a joint configuration (must be inside the limits in
[`packages.md`](packages.md)):

```bash
ros2 topic pub --once /joint_command example_interfaces/msg/Float64MultiArray \
  "{data: [0.0, 0.5, -0.5, 0.0, 0.5, 0.0]}"
```

Send the tool to a Cartesian pose:

```bash
ros2 topic pub --once /pose_command my_robot_interfaces/msg/PoseCommand \
  "{x: 0.30, y: 0.0, z: 0.35, roll: 0.0, pitch: 1.57, yaw: 0.0, cartesian_path: false}"
```

## Named planning states

Defined in `my_robot_moveit_config/config/my_robot.srdf`; used with the MoveIt
API, the RViz *Planning* panel or `MoveGroupInterface::setNamedTarget`.

| Group | State | Value |
| --- | --- | --- |
| `arm` | `home` | all joints 0 |
| `arm` | `pose_1` | `1.1705, 1.1686, 0.9172, 1.2449, 0.9940, 1.0596` |
| `arm` | `pose_2` | `−1.5421, 1.5828, 0.5030, −0.2787, 1.3285, −1.7659` |
| `gripper` | `gripper_open` | `gripper_left_finger_joint = 0.0` |
| `gripper` | `gripper_closed` | `0.06` |
| `gripper` | `gripper_half_closed` | `0.04` |

## Standard ROS 2 interfaces used

| Interface | Provider | Purpose |
| --- | --- | --- |
| `/joint_states` (`sensor_msgs/msg/JointState`) | `joint_state_broadcaster` | Feedback for the planning scene monitor and TF. |
| `/tf`, `/tf_static` | `robot_state_publisher` | Frame tree (`world` → `base_link` → … → fingers). |
| `/robot_description` | `robot_state_publisher` | URDF for RViz and MoveIt. |
| `/move_action` (`moveit_msgs/action/MoveGroup`) | `move_group` | Planning + execution goal interface. |
| `/arm_controller/follow_joint_trajectory`, `/gripper_controller/follow_joint_trajectory` | `joint_trajectory_controller` | Trajectory execution. |
| `/controller_manager/*` services | `controller_manager` | Load, switch and query controllers. |

## Inspecting at runtime

```bash
ros2 node list
ros2 topic list -t
ros2 control list_controllers
ros2 action list -t
ros2 topic echo /joint_states --once
ros2 param get /controller_manager update_rate
```
