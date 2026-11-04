## What changed

<!-- One or two sentences. Link the issue: Closes #123 -->

## Why

<!-- The problem this solves, or the capability it adds. -->

## How it was verified

<!-- Commands run, launch files checked, RViz screenshots for visual changes. -->

```bash
colcon build --symlink-install
colcon test --packages-select <touched packages>
colcon test-result --verbose
pre-commit run --all-files
```

- [ ] Built the touched packages locally without new warnings
- [ ] `colcon test` passes for the touched packages
- [ ] `CHANGELOG.md` updated under `Unreleased`
- [ ] Documentation updated (`README.md`, `docs/`)
- [ ] `moveit_controllers.yaml` and `ros2_controllers.yaml` stay in sync (if controllers changed)

## Scope

<!-- Anything a reviewer should look at closely, plus follow-ups you deliberately left out. -->
