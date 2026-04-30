# Windows Setup Guide

Install every tool your development environment needs. Follow steps in order.

---

## 1. Git for Windows

Git is the version control system that tracks all code changes.

1. Download from https://git-scm.com/download/win, choose the 64-bit installer.
2. Run the installer. When asked about default editor, choose **VS Code** (you'll install that next).
3. For line endings, choose **"Checkout Windows-style, commit Unix-style line endings"**.
4. Accept defaults for everything else.

**Verify:**
```
git --version
```
Expected output: `git version 2.x.x.windows.x`

---

## 2. Visual Studio Code

VS Code is the team's primary editor for both Python and ESP32 firmware.

1. Download from https://code.visualstudio.com/
2. Run the installer. Check **"Add to PATH"** during install.
3. Open VS Code and install these extensions (Ctrl+Shift+X → search and install):
   - **PlatformIO IDE** (by PlatformIO)
   - **Python** (by Microsoft)
   - **Pylance** (by Microsoft)
   - **GitLens** (by GitKraken), optional but very helpful

**Verify:**
```
code --version
```

---

## 3. Python 3.11

Python is used for high-level robot control and simulation.

1. Download Python 3.11 from https://www.python.org/downloads/
   - Choose the Windows installer (64-bit).
2. **Critical:** On the first installer screen, check **"Add python.exe to PATH"**.
3. Click "Install Now".

**Verify:**
```
python --version
pip --version
```
Expected: `Python 3.11.x`

---

## 4. PlatformIO (via VS Code)

PlatformIO is already installed via the VS Code extension (Step 2). Now verify it works.

1. In VS Code, click the PlatformIO icon on the left sidebar (alien head icon).
2. Wait for it to finish installing its core (first launch downloads ~100 MB).
3. You should see "PIO Home" open.

**Verify via terminal:**
```
pio --version
```
If `pio` is not found, add the PlatformIO bin to your PATH:
`C:\Users\<YourName>\.platformio\penv\Scripts`

---

## 5. Install Python Dependencies

### For high-level control:
```
pip install numpy pyserial
```

### For simulation (MuJoCo):
```
pip install mujoco numpy
```

MuJoCo will download ~50 MB. This is the simulation engine the team uses.

**Verify MuJoCo:**
```python
python -c "import mujoco; print(mujoco.__version__)"
```
Expected: `3.x.x`

### For development tools:
```
pip install ruff pytest
```

`ruff` is the code linter/formatter. `pytest` runs tests.

---

## 6. Configure Git Identity

Tell Git who you are. Run these with your real name and the email on your GitHub account:

```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.autocrlf true
```

---

## 7. Authenticate GitHub

1. Go to https://github.com/settings/tokens
2. Generate a new token (classic): check **repo** and **workflow** scopes. Set expiry to 1 year.
3. Copy the token, you will not see it again.
4. When Git asks for a password, paste this token.

Or, use the **GitHub CLI** for easier auth:
```
winget install GitHub.cli
gh auth login
```
Follow the prompts (choose HTTPS, authenticate via browser).

---

## 8. Final Verification

Run this checklist:

```
git --version          # git version 2.x
python --version       # Python 3.11.x
pip --version          # pip 2x.x
pio --version          # PlatformIO Core 6.x
python -c "import mujoco; print('MuJoCo OK')"
python -c "import numpy; print('NumPy OK')"
python -c "import serial; print('PySerial OK')"
ruff --version
pytest --version
```

All should print a version or "OK". Post in `#software-help` if anything fails.

---

**Next step:** [01: Git Basics](../01_git_basics/git_guide.md)
