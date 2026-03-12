# Configuring a Newly Added Submodule Locally After Merge

If a repository has been updated to include a new Git submodule (for example, after merging a pull request that adds or updates a demo as a submodule), follow these steps to clone and initialize the submodule locally:

## 1. Clone the Main Repository

If you haven't already cloned the repository, clone it using:

```bash
git clone https://github.com/ReptilianEye/SUU-Kube-MCP.git
cd SUU-Kube-MCP
```

If you already have a local copy, pull the latest changes:

```bash
git pull origin main
```

_Replace `main` with your branch if different._

## 2. Initialize and Update Submodules

Run the following command to initialize and fetch all submodules:

```bash
git submodule update --init --recursive
```

- `--init` initializes your local `.git/config` with the submodules' config entries.
- `--recursive` initializes and updates any nested submodules (submodules inside submodules).

## 3. Cloning with Submodules (Alternative)

If you're cloning the repo for the first time and want to get submodules immediately, use:

```bash
git clone --recurse-submodules https://github.com/ReptilianEye/SUU-Kube-MCP.git
```

## 4. Checking Submodule Status

To see the submodule status, run:

```bash
git submodule status
```

## 5. Pulling Submodule Updates Later

When the parent repository or submodules are updated in the future, run:

```bash
git pull
git submodule update --init --recursive
```

Or, to update to the latest submodule commits on their remote branches:

```bash
git submodule update --remote --merge
```

## 6. Troubleshooting

If you encounter issues with submodules, you can reinitialize them:

```bash
git submodule sync --recursive
git submodule update --init --recursive
```

## References

- [Git Tools - Submodules (Pro Git book)](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
- [Official Git Submodule Documentation](https://git-scm.com/docs/git-submodule)

---

**Summary:**
- Always initialize and update submodules after pulling changes that add or modify them.
- Use `git submodule update --init --recursive` after clone or pull.
- Use `git clone --recurse-submodules` for a full clone-and-init in one step.
