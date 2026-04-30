# Python & MuJoCo Simulation Guide

Simulation lets us develop and test gait algorithms without the physical robot. Everything we test in simulation gets validated there before it ever touches real hardware.

---

## Why Simulate First?

- No risk of damaging motors or the frame
- Instant reset, crash the robot in sim, restart in 1 second
- Run 10x faster than real time for quick iteration
- The entire gait control algorithm runs identically in sim and on hardware (by design)

---

## What is MuJoCo?

MuJoCo (Multi-Joint dynamics with Contact) is a physics engine specialized for robotic simulation. It is used by:
- MIT (Cheetah), ETH Zurich (ANYmal), Boston Dynamics, DeepMind

It models:
- **Rigid body dynamics**: how links accelerate under forces
- **Contact physics**: what happens when a foot hits the ground
- **Actuators**: motors that apply torque to joints

---

## Installation Check

Verify MuJoCo is installed:

```python
python -c "import mujoco; print(mujoco.__version__)"
```

If not installed:
```
pip install mujoco numpy
```

---

## Your First Simulation: Pendulum

Let's simulate a simple pendulum before touching the robot model. Create a file `pendulum_test.py`:

```python
import mujoco
import mujoco.viewer
import numpy as np

# MuJoCo model defined in XML (called MJCF)
xml = """
<mujoco>
  <worldbody>
    <light pos="0 0 3"/>
    <geom name="floor" type="plane" size="5 5 0.1" rgba="0.8 0.8 0.8 1"/>
    <body name="pendulum" pos="0 0 1">
      <joint name="hinge" type="hinge" axis="0 1 0"/>
      <geom type="capsule" size="0.05 0.5" fromto="0 0 0 0 0 -1" rgba="0.2 0.5 0.8 1"/>
    </body>
  </worldbody>
  <actuator>
    <motor name="motor" joint="hinge" gear="1"/>
  </actuator>
</mujoco>
"""

model = mujoco.MjModel.from_xml_string(xml)
data  = mujoco.MjData(model)

# Run with a viewer
with mujoco.viewer.launch_passive(model, data) as viewer:
    for _ in range(5000):
        data.ctrl[0] = 0.5  # Apply torque
        mujoco.mj_step(model, data)
        viewer.sync()
```

Run it:
```
python pendulum_test.py
```

A window opens showing the pendulum swinging. You just ran a physics simulation.

---

## Understanding the MJCF Format

The robot model is defined in **MJCF** (MuJoCo XML). The team's model lives at:
`quadruped/src/simulation/models/quadruped.xml`

Key elements:

```xml
<mujoco>
  <worldbody>
    <!-- The robot body and its children -->
    <body name="torso" pos="0 0 0.3">
      <!-- A joint connects this body to its parent -->
      <joint name="root" type="free"/>
      <!-- Geometry: collision shape + visual -->
      <geom type="box" size="0.2 0.1 0.05" rgba="0.3 0.3 0.8 1"/>

      <!-- A child body (leg segment) -->
      <body name="FL_hip" pos="0.2 0.1 0">
        <joint name="FL_hip_ab" type="hinge" axis="1 0 0" range="-0.8 0.8"/>
        <geom type="capsule" size="0.02 0.04"/>
        <!-- ...more children... -->
      </body>
    </body>
  </worldbody>

  <actuator>
    <!-- One actuator per joint we control -->
    <position name="FL_hip_ab_act" joint="FL_hip_ab" kp="100"/>
  </actuator>
</mujoco>
```

---

## The Simulation Architecture

In our codebase, all gait and control code works through a `MotorInterface` abstraction:

```python
# src/control/motor_interface.py  (to be written in Phase 1)

class MotorInterface:
    def send_joint_targets(self, targets: dict) -> None: ...
    def get_joint_states(self) -> dict: ...

class SimMotorInterface(MotorInterface):
    """Wraps MuJoCo, used during simulation."""
    def __init__(self, model, data): ...

class SerialMotorInterface(MotorInterface):
    """Wraps USB Serial to ESP32, used on real hardware."""
    def __init__(self, port: str): ...
```

This means `GaitController` never knows if it's running in simulation or on the robot. You swap the interface at startup.

---

## Running the Robot Simulation

Once the model is built (Phase 2 task), you'll run it with:

```
python src/simulation/run_sim.py
```

For now, verify the sim dependencies work:

```python
import mujoco
import numpy as np

print("MuJoCo:", mujoco.__version__)
print("NumPy:", np.__version__)
print("Simulation environment ready.")
```

---

## Exercise

1. Run the pendulum simulation above, make it work.
2. Change `data.ctrl[0] = 0.5` to `data.ctrl[0] = -2.0`, what happens to the pendulum?
3. Read through `quadruped/src/simulation/` in the main repo and write a brief summary of what each file is for.

---

**Next step:** [04: CAN Bus & Motors](../04_can_bus/can_bus_guide.md)
