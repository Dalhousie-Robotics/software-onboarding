# Dal Robotics — Software Team Onboarding

Welcome to the Software & Simulations team. This guide gets you from zero to making your first real contribution.

**Complete every step in order. Do not skip.**

---

## Before You Start

You need:
- A laptop running **Windows 10 or 11** (64-bit)
- A **GitHub account** — create one at github.com if you don't have one
- About **3–4 hours** for initial setup (mostly downloads)

Once you have those, DM the Software Lead your GitHub username so they can add you to the `dal-robotics` GitHub organization.

---

## Week 1 Checklist

Work through these in order. Each item links to a detailed guide.

### Environment Setup
- [ ] [Install all required software](00_setup/windows_setup.md) (Git, VS Code, Python, PlatformIO, MuJoCo)
- [ ] Verify each tool works (each guide has a verification step)

### Git & GitHub
- [ ] [Complete the Git basics guide](01_git_basics/git_guide.md)
- [ ] Clone the `quadruped` repo: `git clone https://github.com/dal-robotics/quadruped.git`
- [ ] Clone this repo: `git clone https://github.com/dal-robotics/onboarding.git`

### Your First Contribution
- [ ] Add your name to [roster.md](roster.md) following the format already there
- [ ] Commit and open a Pull Request to this repo — follow [the Git guide](01_git_basics/git_guide.md#making-a-pull-request)
- [ ] Get your PR reviewed and merged (the Software Lead reviews onboarding PRs within 24h)

### Deep Dives (self-paced, complete by Week 2 meeting)
- [ ] [ESP32 & PlatformIO guide](02_platformio_esp32/esp32_guide.md) — blink an LED
- [ ] [Python & MuJoCo simulation guide](03_python_simulation/python_mujoco_guide.md) — run the simulation
- [ ] [CAN bus & motor protocol guide](04_can_bus/can_bus_guide.md) — understand how motors talk

---

## Week 2

Attend your first team meeting. Bring:
- Your dev environment working
- Your onboarding PR merged
- One question about anything you didn't understand

At the meeting you'll be assigned your first real issue.

---

## Getting Help

- **Stuck on setup?** Post in the team Discord `#software-help` channel with your error message.
- **Question about the codebase?** Open a GitHub Discussion in the `quadruped` repo.
- **Something in the docs is wrong or unclear?** Open a PR to fix it — documentation improvements count as real contributions.

---

## Guides Index

| Guide | What you'll learn |
|---|---|
| [00 — Windows Setup](00_setup/windows_setup.md) | Install every tool you need |
| [01 — Git Basics](01_git_basics/git_guide.md) | Git workflow, branches, PRs |
| [02 — ESP32 & PlatformIO](02_platformio_esp32/esp32_guide.md) | Write and flash ESP32 firmware |
| [03 — Python & MuJoCo](03_python_simulation/python_mujoco_guide.md) | Run the robot simulation |
| [04 — CAN Bus & Motors](04_can_bus/can_bus_guide.md) | How motors communicate |
