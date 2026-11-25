# Quadrotor Reinforcement Learning and Control-Output Experiments in Genesis

**PPO-based quadrotor position-control experiments comparing direct rotor actuation with collective-thrust and body-rate (CTBR) policy outputs.**

This repository is part of the [Robust Goal-Directed Aerial Manipulation project](https://github.com/ishita-chaudhary/aerial-manipulation-rl-control). A key question in that research was **what level of control command a reinforcement-learning policy should generate**: individual motor commands or higher-level references that can be tracked by a separate flight controller.

The Genesis experiments investigate this question using goal-conditioned quadrotor hovering and position control, with additional variants for reward design, observation noise, and model/actuator perturbations.

## Policy Interfaces

Two four-dimensional action spaces are implemented:

| Interface | Policy output | How it is applied |
| --- | --- | --- |
| **Direct rotor RPM** | Four normalized rotor-speed commands | Actions are scaled to per-rotor RPMs and applied directly in the simulator. |
| **Collective thrust and body rates (CTBR)** | Collective thrust and desired roll, pitch, and yaw rates | A PID angular-rate controller calculates body moments; a motor mixer converts thrust and moments into rotor-speed commands. |

The CTBR interface leaves **angular-rate tracking and motor allocation** to a lower-level controller, rather than requiring the policy to learn individual rotor actuation. The implementations use a custom Tarot 650 quadrotor URDF located in [`custom urdf/`](custom%20urdf/).

### Policy observations

The policy receives the target-relative position, vehicle orientation, body-frame linear and angular velocities, and its previous action. The orientation representation differs by experiment:

| Observation component | Direct RPM (`drone_rpm`) | CTBR variants |
| --- | ---: | ---: |
| Target-relative position | 3 | 3 |
| Vehicle orientation | 3 (Euler angles) | 4 (quaternion) |
| Body-frame linear velocity | 3 | 3 |
| Body-frame angular velocity | 3 | 3 |
| Previous action | 4 | 4 |
| **Total** | **16** | **17** |

The task supplies a three-dimensional position target. Rewards and termination conditions encourage movement toward the target while discouraging unstable motion and crashes.

## Implemented Experiments

| Directory | Focus |
| --- | --- |
| [`drone_rpm/`](drone_rpm/) | Direct rotor-RPM policy actions, hovering environment, PPO training, and evaluation scripts. |
| [`drone_actual/`](drone_actual/) | CTBR control with a PID angular-rate loop and target-holding reward. |
| [`drone_hover/`](drone_hover/) | CTBR hovering with additional reward terms and action-smoothing infrastructure. |
| [`drone_noise/`](drone_noise/) | Observation-noise experiments and utilities for mass, actuator-efficiency, and action-smoothing perturbations. |
| [`drone_rew/`](drone_rew/) | Reward-design experiments, including a bonus for reaching and holding a position target. |
| [`custom urdf/`](custom%20urdf/) | Tarot 650 model and associated robot-description resources. |

Across the variants, reward terms include progress toward the target, distance-based shaping, target holding/completion, action smoothness, and penalties for crashes or excessive rotational motion. The configurations allow these terms to be included and weighted differently.

## Training and Evaluation

The environments use **Genesis** for parallel physics simulation and **Proximal Policy Optimization (PPO)** through **RSL-RL**. Each experiment includes its own environment, training script, and evaluation script. The training scripts configure the actor–critic networks, PPO hyperparameters, number of parallel environments, and checkpoint saving.

After setting up Genesis and the Python dependencies, the repository's training scripts expect `rsl-rl-lib==2.2.4`:

```bash
pip install rsl-rl-lib==2.2.4 tensorboard
```

For example, to train the CTBR hover environment from the repository root:

```bash
cd drone_hover
python hover_train.py -e genesis-ctbr --num_envs 8192 --max_iterations 301
```

Evaluate an available checkpoint from the same directory:

```bash
python hover_eval.py -e genesis-ctbr --ckpt 300
```

The checkpoint number must match a model saved during training. Equivalent `hover_train.py` and `hover_eval.py` scripts are available in the other experiment directories.

## Related Repositories

- **[Aerial Manipulation — Project Overview](https://github.com/ishita-chaudhary/aerial-manipulation-rl-control):** overall research objective and other experimental implementations.
- **[Isaac Lab quadrotor RL](https://github.com/rocon-rrc/quadcopter_issaclab-rl):** related collaborative quadrotor-learning experiments.
- **[Flightmare RL Training](https://github.com/ishita-chaudhary/flightmare):** quadrotor-policy training in Flightmare.
- **[Flightmare Policy Inference in PX4–Gazebo](https://github.com/ishita-chaudhary/RLQuadcopterControlFlightmare):** a separate sim-to-sim inference pipeline.

The simulation and RL components build on the [Genesis physics engine](https://github.com/Genesis-Embodied-AI/Genesis), [RSL-RL](https://github.com/leggedrobotics/rsl_rl), and example code from [GenesisDroneEnv](https://github.com/KafuuChikai/GenesisDroneEnv). Credit for those upstream projects remains with their original authors.

*Research conducted at the Robotics Research Center, IIIT Hyderabad.*
