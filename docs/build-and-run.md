# Build and run

## 1. Prerequisites

| Requirement | Version / package |
| --- | --- |
| OS | Ubuntu 24.04 LTS (Noble) |
| ROS 2 | Jazzy — `ros-jazzy-desktop` |
| MoveIt 2 | `ros-jazzy-moveit`, `ros-jazzy-moveit-py` |
| Build tools | `python3-colcon-common-extensions`, `python3-rosdep`, `python3-vcstool` |
| Python deps | `tf_transformations` (`sudo apt install python3-tf-transformations` or `pip install transforms3d`) |

Minimal install:

```bash
sudo apt update
sudo apt install -y ros-jazzy-desktop ros-jazzy-moveit ros-jazzy-moveit-py \
                    python3-colcon-common-extensions python3-rosdep \
                    python3-tf-transformations
```

## 2. Get the sources

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
git clone https://github.com/Saadest/project_GMT.git
```

> The six packages live at the repository root, so the repository itself is the
> `src` subtree. Do not clone it into `src/src`.

## 3. Resolve dependencies

```bash
cd ~/ros2_ws
sudo rosdep init 2>/dev/null || true   # first run only
rosdep update
rosdep install --from-paths src --ignore-src -r -y
```

## 4. Build

```bash
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash
```

Selective rebuild while iterating:

```bash
colcon build --symlink-install --packages-select my_robot_description
```

## 5. Run

### 5.1 Model only (fastest sanity check)

```bash
ros2 launch my_robot_description display.launch.xml
```

Starts `robot_state_publisher`, `joint_state_publisher_gui` and RViz2. Drag the
sliders to verify every joint direction, limit and mesh.

### 5.2 Full stack

```bash
ros2 launch my_robot_bringup my_robot.launch.xml
```

Expected sequence in the log: `robot_state_publisher` loads the URDF → the
controller manager loads `ros2_controllers.yaml` → the three spawners report
`Configured and activated` → `move_group` prints the planning pipeline loading
→ RViz2 opens with the motion-planning display.

Verify from another terminal:

```bash
ros2 control list_controllers      # 3 controllers, all active
ros2 topic hz /joint_states        # ~100 Hz
ros2 action list | grep follow_joint_trajectory
```

### 5.3 MoveIt 2 demo

```bash
ros2 launch my_robot_moveit_config demo.launch.py
```

Useful when you only want planning and visualisation with the fake controllers,
without the full bringup.

### 5.4 Commander nodes

```bash
ros2 run my_robot_commander_py commander     # Python / moveit_py
ros2 run my_robot_commander_cpp commander    # C++ / MoveGroupInterface
```

With a commander running, publish a goal:

```bash
ros2 topic pub --once /joint_command example_interfaces/msg/Float64MultiArray \
  "{data: [0.0, 0.5, -0.5, 0.0, 0.5, 0.0]}"
```

More examples in [`interfaces.md`](interfaces.md).

The C++ package also ships `test_moveit`, a bare probe that plans and executes a
named target — handy to confirm the planning stack before adding logic:

```bash
ros2 run my_robot_commander_cpp test_moveit
```

## 6. Tests

```bash
cd ~/ros2_ws
colcon test --packages-select my_robot_commander_py
colcon test-result --verbose
```

The Python package runs the standard `ament_copyright`, `ament_flake8` and
`ament_pep257` checks. Lint failures in generated code are expected until the
sources are formatted; see *Known limitations* in the README.

## 7. Troubleshooting

| Symptom | Cause | Fix |
| --- | --- | --- |
| `Package 'my_robot_description' not found` | Workspace not sourced | `source install/setup.bash` in every new shell |
| `Controller manager not available` | Spawner started before `ros2_control_node` | Relaunch; ensure only one bringup instance is running |
| RViz shows a red robot | Fixed frame is not `world` / `base_link` | Set *Fixed Frame* to `world` |
| `Could not find the kinematics plugin` | `ros-jazzy-moveit-kinematics` missing | `sudo apt install ros-jazzy-moveit-kinematics` |
| `ModuleNotFoundError: moveit` | MoveIt Python bindings absent | `sudo apt install ros-jazzy-moveit-py` |
| `ModuleNotFoundError: tf_transformations` | Missing dependency | `sudo apt install python3-tf-transformations` |
| Planning fails with "start state in collision" | Initial positions collide, or joint limits disagree | Check `initial_positions.yaml` against the SRDF states; re-check `joint_limits.yaml` |
| `xacro` errors about `$(find pkg)` | Old xacro invocation | Build with Jazzy; the launch files use `$(find-pkg-share ...)` |
| Gripper opens in the wrong direction | Mimic multiplier sign | `gripper_right_finger_joint` mimics with multiplier `-1.0`; keep it when editing |

Collect diagnostics for a bug report:

```bash
ros2 doctor --report > ros-doctor.txt
colcon build --symlink-install --event-handlers console_direct+ 2> build.log
```
