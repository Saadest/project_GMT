# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Planned
- Gazebo / `ros2_control` hardware plugin for the simulation backend.
- Repository-level integration tests for the commander nodes.
- Package metadata cleanup (`package.xml` descriptions and maintainer fields).

## [0.1.0] - 2026-11-04

### Added
- `my_robot_interfaces`: `PoseCommand` message covering Cartesian pose
  (position + RPY) and a `cartesian_path` flag.
- `my_robot_description`: xacro-based 6-DoF arm plus parallel gripper, RViz2
  display launch file and view configuration.
- `my_robot_moveit_config`: MoveIt 2 configuration generated with the MoveIt
  Setup Assistant (SRDF, joint limits, KDL kinematics, OMPL pipeline,
  controller mapping, RViz motion-planning layout).
- `my_robot_bringup`: single-entry launch file that starts
  `robot_state_publisher`, `ros2_control_node`, the `joint_state_broadcaster`,
  `arm_controller`, `gripper_controller`, `move_group` and RViz2.
- `my_robot_commander_py` / `my_robot_commander_cpp`: commander nodes exposing
  named-target, joint-space and Cartesian-pose motion plus gripper control.
- Repository tooling: CI workflow (colcon build/test + advisory lint),
  issue/PR templates, `.editorconfig`, `.gitattributes`, `pre-commit` hooks.

[Unreleased]: https://github.com/Saadest/project_GMT/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/Saadest/project_GMT/releases/tag/v0.1.0
