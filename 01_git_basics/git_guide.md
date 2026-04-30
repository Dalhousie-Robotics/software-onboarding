# Git Basics: Dal Robotics Workflow

Git tracks every change to code so we can collaborate without overwriting each other. This guide teaches you the workflow our team uses every day.

---

## Key Concepts

| Term | What it means |
|---|---|
| **Repository (repo)** | A folder tracked by Git, containing all code and its history |
| **Commit** | A saved snapshot of your changes with a message explaining what changed |
| **Branch** | An independent line of development, your work doesn't affect others until it's merged |
| **Pull Request (PR)** | A request to merge your branch into the main codebase, with code review |
| **main** | The stable, production-ready branch, you never push here directly |
| **develop** | The integration branch, all features merge here first |

---

## Initial Setup (once)

Clone the repos you'll work in:

```
git clone https://github.com/dal-robotics/quadruped.git
git clone https://github.com/dal-robotics/onboarding.git
cd quadruped
```

---

## Our Daily Workflow

Every piece of work, no matter how small, follows this cycle:

### Step 1: Always start from a fresh `develop`

```
git checkout develop
git pull origin develop
```

This ensures you're starting from the latest code.

### Step 2: Create a branch for your work

```
git checkout -b feature/your-description
```

Follow the naming convention:
- `feature/`: new functionality
- `fix/`: bug fix
- `docs/`: documentation only

Example: `git checkout -b feature/ak40-can-driver`

### Step 3: Make your changes

Work in VS Code. Save files normally.

### Step 4: Stage and commit

See what changed:
```
git status
git diff
```

Stage the files you want to commit:
```
git add src/firmware/lib/ak40_driver/ak40_driver.h
git add src/firmware/lib/ak40_driver/ak40_driver.cpp
```

Or stage everything (use carefully):
```
git add .
```

Commit with a meaningful message:
```
git commit -m "feat(firmware): add AK40-10 CAN driver header and implementation"
```

Commit message rules (from [CONTRIBUTING.md](../../quadruped/CONTRIBUTING.md)):
- Start with a type: `feat`, `fix`, `docs`, `refactor`, `test`
- Add a scope in parentheses: `(firmware)`, `(kinematics)`, `(simulation)`
- Imperative tense, lowercase, no period at end
- Keep it under 72 characters

### Step 5: Push your branch to GitHub

First push:
```
git push -u origin feature/your-description
```

Subsequent pushes on the same branch:
```
git push
```

### Step 6: Open a Pull Request

1. Go to https://github.com/dal-robotics/quadruped
2. GitHub will show a banner: "Compare & pull request", click it.
3. Set **base** to `develop` (not `main`).
4. Fill in the PR template completely.
5. Request review from the Software Lead.

### Step 7: Address review comments

Reviewers will leave comments. For each comment:
- Make the change in your code
- Commit and push, the PR updates automatically
- Reply to the comment with "done" or your reasoning if you disagree

### Step 8: Merge

Once approved and CI is green, the Software Lead will squash-merge your PR into `develop`.

---

## Common Commands Reference

```bash
# See current status
git status

# See what changed (unstaged)
git diff

# See what's staged
git diff --cached

# See commit history
git log --oneline -10

# Undo changes to a file (before staging)
git restore filename.py

# Unstage a file
git restore --staged filename.py

# Pull latest changes
git pull origin develop

# Switch branches
git checkout develop
git checkout feature/my-branch

# See all branches
git branch -a
```

---

## Making a Pull Request (Your First One)

Your first PR is to add yourself to the roster. Here's the exact commands:

```bash
# Clone the onboarding repo (if you haven't already)
git clone https://github.com/dal-robotics/onboarding.git
cd onboarding

# Create your branch
git checkout -b docs/add-your-name-to-roster

# Open roster.md in VS Code and add your row
code roster.md

# Stage and commit
git add roster.md
git commit -m "docs(roster): add Your Name"

# Push
git push -u origin docs/add-your-name-to-roster
```

Then go to GitHub and open the PR. That's it.

---

## What Not to Do

- **Never `git push` directly to `main` or `develop`** because branch protection will block it anyway
- **Never `git commit -m "fix"` or `"update"`**, be descriptive
- **Never commit `.env` files, passwords, or API keys**
- **Never use `git push --force`** unless you are 100% sure what you're doing and have asked the Lead

---

**Next step:** [02: ESP32 & PlatformIO](../02_platformio_esp32/esp32_guide.md)
