# Contributing to project_GMT

Thanks for taking the time to contribute. This document describes the workflow,
the coding conventions and the checks a change has to pass.

## Code of conduct

Be respectful and constructive. Harassment or personal attacks are not
tolerated in issues, pull requests or any other project space.

## Ways to contribute

- Report a defect or request a feature with one of the issue templates.
- Improve documentation (clarity, accuracy, missing run recipes).
- Add or extend tests for the description, configuration or commander nodes.
- Propose a hardware or simulation backend for `ros2_control`.

## Development setup

```bash
mkdir -p ~/ros2_ws/src && cd ~/ros2_ws/src
git clone https://github.com/Saadest/project_GMT.git
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash

# Optional but recommended: repository git hooks
pip install --user pre-commit
cd src/project_GMT && pre-commit install
```

## Branching and commits

| Branch | Purpose |
| --- | --- |
| `main` | Always buildable, protected, squash-merged into. |
| `feat/<scope>` | New functionality. |
| `fix/<scope>` | Defect fixes. |
| `docs/<scope>` | Documentation only. |
| `chore/<scope>` | Tooling, CI, housekeeping. |

Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <short imperative summary>

<optional body: what changed and why, wrapped at 72 columns>

<optional footer: Refs #123, BREAKING CHANGE: ...>
```

Allowed types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`,
`build`, `ci`, `chore`, `revert`. Valid scopes match package names without the
`my_robot_` prefix — `interfaces`, `description`, `moveit`, `bringup`,
`commander`, `repo`.

Examples:

```
feat(commander): expose named-target goals on a `named_target` topic
fix(moveit): raise joint_limits velocity for joint3 to 1.5 rad/s
docs(build): document the ros-jazzy-moveit-py install step
```

## Coding conventions

**General**

- 4-space indentation for Python, XML/xacro and launch files; 2 spaces for
  YAML and C++ headers. See `.editorconfig`.
- LF line endings, a trailing newline, no trailing whitespace.
- Do not commit build artifacts (`build/`, `install/`, `log/`) or credentials.

**Python**

- Target Python 3.12 (ROS 2 Jazzy), PEP 8, max line length 120.
- Node classes inherit from `rclpy.node.Node`, use type hints in new code, and
  keep callbacks free of blocking work.
- Every package ships the standard `test_copyright`, `test_flake8` and
  `test_pep257` checks.

**C++**

- C++17, `rclcpp` style, `//` comments, `snake_case` for members declared with a
  trailing underscore is **not** used here — keep the existing project style
  within a file rather than mixing conventions.
- Prefer `std::make_shared`, avoid raw `new`, keep headers in the package's
  `include/` directory when a package grows beyond a single translation unit.

**xacro / URDF**

- One concern per file; include files from `my_robot.urdf.xacro` only.
- Name links `<part>_link` and joints `<part>_joint`; use `joint1`…`joint6` for
  the actuated arm chain.
- Every inertial/visual/collision element must exist for a link that is used in
  planning.

**MoveIt configuration**

- Regenerate with the Setup Assistant rather than hand-editing the SRDF when
  possible; if a manual edit is unavoidable, note it in the PR description.
- Keep `moveit_controllers.yaml` in sync with `ros2_controllers.yaml`; a
  controller renamed in one file must be renamed in the other.

## Before opening a pull request

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select \
  my_robot_interfaces my_robot_description my_robot_moveit_config \
  my_robot_bringup my_robot_commander_py my_robot_commander_cpp
colcon test --packages-select my_robot_commander_py
colcon test-result --verbose

cd src/project_GMT && pre-commit run --all-files
```

Checklist:

- [ ] The workspace builds without warnings introduced by the change.
- [ ] `colcon test` passes for the touched packages.
- [ ] New behaviour is documented in `README.md` / `docs/`.
- [ ] `CHANGELOG.md` has an entry under `Unreleased`.
- [ ] The PR description states **what** changed, **why**, and **how it was
      verified** (commands run, launch files checked, screenshots for RViz).

## Review and merge

- One approving review is required; CI must be green (lint is advisory).
- PRs are **squash-merged**; the PR title becomes the commit subject, so keep it
  in Conventional Commit form.
- Releases are cut by tagging `main` with `vMAJOR.MINOR.PATCH` and promoting the
  `Unreleased` changelog section.
