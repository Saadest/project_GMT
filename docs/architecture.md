# Architecture

This document explains how the packages fit together at runtime and why the
stack is split the way it is.

## Layered view

```
┌──────────────────────────────────────────────────────────────────┐
│  Command layer        PoseCommand · Float64MultiArray · Bool     │
├──────────────────────────────────────────────────────────────────┤
│  Application layer    my_robot_commander_py  (moveit_py)         │
│                       my_robot_commander_cpp (MoveGroupInterface)│
├──────────────────────────────────────────────────────────────────┤
│  Planning layer       move_group · OMPL pipeline · KDL IK        │
│                       my_robot.srdf (arm / gripper groups)       │
├──────────────────────────────────────────────────────────────────┤
│  Control layer        controller_manager                         │
│                       joint_state_broadcaster                    │
│                       arm_controller, gripper_controller          │
│                       (joint_trajectory_controller)              │
├──────────────────────────────────────────────────────────────────┤
│  Hardware layer       mock_components/GenericSystem (simulation) │
├──────────────────────────────────────────────────────────────────┤
│  Model layer          xacro → URDF → robot_state_publisher → /tf │
└──────────────────────────────────────────────────────────────────┘
```

Each layer is a separate package so that a layer can be replaced without
touching its neighbours — for example, swapping the simulated hardware for a
vendor driver, or replacing the OMPL pipeline with Pilz or CHOMP, only affects
the configuration package.

## Runtime node graph

`my_robot_bringup/launch/my_robot.launch.xml` is the single entry point. It
starts:

| Node | Package | Responsibility |
| --- | --- | --- |
| `robot_state_publisher` | `robot_state_publisher` | Expands `my_robot.urdf.xacro` and publishes the static/dynamic TF tree and `robot_description`. |
| `ros2_control_node` | `controller_manager` | Loads the hardware interface and controller configuration from `config/ros2_controllers.yaml`. |
| `spawner` × 3 | `controller_manager` | Activates `joint_state_broadcaster`, `arm_controller`, `gripper_controller`. |
| `move_group` | `my_robot_moveit_config` | Planning scene, IK, trajectory generation, action servers. |
| `rviz2` | `rviz2` | Visualization with `rivz/urdf_config.rviz`. |

The commander nodes are **not** started by the bringup; they are run separately
so that the same stack can serve a real application, a test script or a manual
operator session.

## Data flow of a motion request

1. **Command.** A client publishes one of the command topics (see
   [`interfaces.md`](interfaces.md)).
2. **Intent translation.** The commander converts the command into a MoveIt
   goal:
   - joint command → `RobotState` with six joint positions;
   - pose command → `PoseStamped` in `base_link` with RPY converted to a
     quaternion and `tool_link` as the target link;
   - gripper command → the named state `gripper_open` / `gripper_closed`.
3. **Planning.** `move_group` runs the configured OMPL pipeline; request
   adapters validate frames, bounds and start-state collision; response adapters
   time-parameterize and validate the trajectory.
4. **Execution.** The trajectory is sent to `arm_controller` /
   `gripper_controller` as a `FollowJointTrajectory` goal.
5. **Actuation.** The joint trajectory controller writes position commands to
   the `ros2_control` hardware interface (`mock_components/GenericSystem` in
   simulation) and reads back joint states.
6. **Feedback.** `joint_state_broadcaster` publishes `/joint_states`, which
   feeds both the planning scene monitor and `robot_state_publisher`.

## Planning model

The SRDF defines two groups and the named states used by the commanders:

| Group | Joints | Named states |
| --- | --- | --- |
| `arm` | `virtual_joint` (fixed), `joint1`…`joint6`, `hand_tool_joint` (fixed) | `home`, `pose_1`, `pose_2` |
| `gripper` | `gripper_base_joint` (fixed), `gripper_left_finger_joint`, `gripper_right_finger_joint` (mimic, × −1) | `gripper_open`, `gripper_closed`, `gripper_half_closed` |

- The end effector `gripper` is attached to `tool_link` and belongs to `arm`.
- `virtual_joint` fixes `base_link` to the `world` frame, so `move_group` and
  RViz share one inertial frame.
- Kinematics: `kdl_kinematics_plugin/KDLKinematicsPlugin`, search resolution
  `0.005`, timeout `0.005` s.
- Allowed-planning-time and velocity/acceleration scaling default to `1.0` in
  the planner configuration.

## Design decisions

**Why two commanders?** The Python node is the quick path for prototyping with
`moveit_py`'s `PlanningComponent`; the C++ node is the path for deployment with
`MoveGroupInterface` and no Python runtime overhead. They deliberately expose
the **same topic contract**, so a client can switch between them without
changes.

**Why a custom `PoseCommand` message?** A Cartesian goal plus a
`cartesian_path` flag does not map cleanly onto any `geometry_msgs` type, and
the flag leaves room for a future `compute_cartesian_path` implementation
without another interface change.

**Why `mock_components/GenericSystem`?** It lets the full control chain run in
CI and on any developer laptop without a simulator, while keeping
`ros2_control` as the single integration seam for real hardware.

**Why is the description split into three xacro files?** `arm.xacro`,
`gripper.xacro` and `my_robot.ros2_control.xacro` are independent of each other
and of `common_properties.xacro` (materials). A different end effector only
needs a new `gripper.xacro` plus its joint in `my_robot.urdf.xacro`.

## Extension points

| Goal | Where to change |
| --- | --- |
| Real hardware | Replace the `<hardware><plugin>` block in `my_robot_description/urdf/my_robot.ros2_control.xacro` with your `ros2_control` plugin and add its parameters. |
| New end effector | Add a xacro file, include it from `my_robot.urdf.xacro`, regenerate the SRDF. |
| Different planner | Edit `planning_pipelines` in the MoveIt configuration or add `ompl_planning.yaml` overrides; the Pilot/Pilz limits file is already present. |
| New command | Add a message in `my_robot_interfaces` and a subscription in both commanders to keep parity. |
| Cartesian motions | `PoseCommand.cartesian_path` is reserved; implement `compute_cartesian_path` in the commanders and document the behaviour here. |
