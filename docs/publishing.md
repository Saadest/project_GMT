# Publishing and versioning

This repository is prepared as a standalone git repository with a linear,
scoped history. Nothing else is required to publish it.

## 1. Canonical repository

Badges, compare links and the issue-template contact links point at
[`Saadest/project_GMT`](https://github.com/Saadest/project_GMT). If the
repository is renamed, transferred or forked into a place where the badges
should resolve, update those URLs once:

```bash
grep -rl 'Saadest/project_GMT' . --exclude-dir=.git \
  | xargs sed -i 's|Saadest/project_GMT|<owner>/project_GMT|g'
git commit -am "docs: point badges and links at the canonical repository"
```

## 2. Create the remote repository

With the GitHub CLI:

```bash
gh repo create project_GMT --public --source=. --remote=origin --push
```

Or manually: create an empty repository on GitHub (**no** README, license or
`.gitignore` — those already exist here), then:

```bash
git remote add origin git@github.com:<your-user>/project_GMT.git
git branch -M main
git push -u origin main
```

## 3. Recommended repository settings

| Setting | Value |
| --- | --- |
| Visibility | Public (contains no credentials) |
| Default branch | `main` |
| Branch protection on `main` | Require PR review, require status checks (`build-and-test`), disallow force pushes |
| Merge strategy | Squash merge only; delete branch after merge |
| Topics | `ros2`, `ros2-jazzy`, `moveit2`, `robotics`, `motion-planning`, `urdf`, `xacro`, `ros2-control` |
| Description | "ROS 2 Jazzy + MoveIt 2 stack for a 6-DoF arm with a parallel gripper" |
| Social preview | Optional; a render of the robot from RViz2 works well |

## 4. Versioning

- Semantic versioning: `MAJOR.MINOR.PATCH`.
- `0.1.0` is the initial state of this stack.
- Tag releases from `main` once CI is green:

```bash
git tag -a v0.1.0 -m "v0.1.0 — initial public release"
git push origin v0.1.0
gh release create v0.1.0 --generate-notes
```

- Move the `Unreleased` section of `CHANGELOG.md` into the new version block in
  the same commit that bumps `package.xml` versions.

## 5. Continuous integration

`.github/workflows/ci.yml` runs on every push and pull request:

| Job | Blocking | What it does |
| --- | --- | --- |
| `build-and-test` | Yes | Builds all packages with `colcon` on Ubuntu 24.04 / ROS 2 Jazzy, runs `colcon test`, uploads logs. |
| `lint` | No (advisory) | Runs `pre-commit` (whitespace, YAML/XML validity, flake8) over the whole tree. |

Promote the lint job to blocking once `pre-commit run --all-files` is clean:
remove `continue-on-error: true` from the job in the workflow.

## 6. History

The history is intentionally split into reviewable, single-purpose commits
(scaffolding → interfaces → description → MoveIt configuration → bringup →
commanders → CI → documentation) so that a reviewer can follow the stack
bottom-up.

The commits were created with the neutral identity
`project_GMT maintainers <maintainers@users.noreply.github.com>`. To attribute
them to yourself before the first push:

```bash
git config user.name "Your Name"
git config user.email "you@example.com"
git rebase -r --root --exec 'git commit --amend --no-edit --reset-author'
git log --format='%h %an <%ae> %s' | head
```

The same result without a rebase, if you prefer a single author identity baked
into the refs (rewrites all history, then force-push):

```bash
git filter-branch -f --env-filter '
  export GIT_AUTHOR_NAME="Your Name"
  export GIT_AUTHOR_EMAIL="you@example.com"
  export GIT_COMMITTER_NAME="$GIT_AUTHOR_NAME"
  export GIT_COMMITTER_EMAIL="$GIT_AUTHOR_EMAIL"
' -- --all
```

## 7. Offline transfer

If the repository is moved by file transfer rather than by `git push`, ship the
bundle produced next to the working tree:

```bash
git bundle create project_GMT.bundle --all
# on the target machine
git clone project_GMT.bundle project_GMT
```

The bundle contains every ref and the full history in a single file.
