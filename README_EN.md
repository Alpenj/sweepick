# SWEEPICK · A low-cost dual-arm mobile robot for clearing floor objects

[한국어](README.md) · English

SWEEPICK picked up and moved small floor objects before cleaning. Our four-person team built it from August to November 2026 in Dongguk University's DAPIER Physical AI program, using a TurtleBot3 Waffle, two SO-101 arms, RGB-D cameras and wrist cameras. This repository contains the arm software, simulation data, ACT training and evaluation, hardware integration, and the ROS 2 handoff between the base and arms. The base's Visual SLAM, Nav2 and docking are in a [teammate's repository](https://github.com/shouttt1320/Dapier_project_visaul_slam).

## Simulation grasp evaluation

| All evaluated positions | Test positions held out from training |
|---|---|
| **132/143 (92.3%)** | **27/31 (87.1%)** |

The 31 Test positions are part of the 143 evaluated positions. Demonstrations from those Test positions were held out from training. We first ran the epoch-30 checkpoint at one validation position during training, then froze it for the full evaluation. We tested 80 grid points (8 × 10) and 63 cell centers (7 × 9), once per position. The policy also completed the task at 17 of 21 Train positions that had no demonstrations.

A run counted as complete when the robot lifted the object at least 30 mm, reached the stow pose and held the raised object in contact for two simulation seconds. We varied object color, background and tabletop textures, and lighting. Object size (40 mm), yaw, physics and camera calibration stayed fixed. [Evaluation details (Korean)](docs/CUSTOM_ACT_V1_KO.md) · [Position-by-position results](docs/evidence/custom_act_v1_20261027_positions.csv) · [Evaluation summary](docs/evidence/custom_act_v1_20261027_summary.json)

<p align="center">
  <img src="docs/media/sweepick-real-run-4x.gif" width="640" alt="SWEEPICK picks up a tissue with both arms and moves it to a box marked with an ArUco tag (4× speed)">
</p>

<p align="center"><sub>Final demonstration (4× speed) · <a href="https://julianjeonresume.netlify.app/assets/video/sweepick-real-full-run.mp4">Original video, 62 seconds</a> · <a href="https://julianjeonresume.netlify.app/sweepick/">Portfolio case study</a></sub></p>

## What we tested

| Area | Result |
|---|---|
| Full run on hardware | On November 1, 2026, the robot reached point A, picked up an object with the dual-arm ACT policy, moved to point B and placed it, then returned to HOME and released torque. It was the first complete run without human intervention. |
| Hardware grasping by object | During the final presentation, the robot picked up a 3 cm cube in 7/10 attempts, a tissue in 8/10 and a round rubber piece in 7/10. We tested ten attempts per object in the workspace without object-specific fine-tuning. |
| Training data | SIM Expert generated 113 successful trajectories. We created five appearances per trajectory for 565 views and split the trajectories into 59 Train, 29 Validation and 25 Test groups. All appearance variants of a trajectory stayed in the same split. |
| Grasp checks and recovery on hardware | We switched from a camera-based check to gripper servo load. The readings were 23–26 ticks while holding an object and 6–8 ticks with an empty gripper. For an empty grasp, the arms retracted automatically before the robot re-approached and restarted ACT. |
| ROS 2 handoff | `/task/ack`, `/task/command` (10 cm or 30° per command), `/sweepick/ready`, a 2 Hz heartbeat, and a base lock while the arms worked. |

At project close we left hardware integration of cleaning, docking and charging, and repeated success-rate measurements under the same setup, for future work. [Hardware integration record (Korean)](docs/PROJECT_DETAIL_KO.md)

## What I worked on

| Area | Role | Work |
|---|---|---|
| Simulation data and domain randomization | Led | Built SIM Expert in RoboTwin and attempted demonstrations at 143 positions. Used the 113 successful trajectories to produce appearance variants and checked alignment across observations and actions. Set up the real workspace in Isaac Sim to try Sim-to-Real transfer. |
| Sim-to-Real hardware integration | Led | Automated grasp demonstrations on the physical robot and converted the recordings into episodes for ACT training. Built the native automatic teacher and DAgger handover. Built and maintained a dashboard for robot control and communication checks that displayed start and stop command responses alongside target and measured joint positions. Applied MuJoCo Jacobian-based damped least-squares inverse kinematics and seventh-order time scaling to the SO-101 arms. Implemented servo-load grasp checks and retry handling. |
| ROS 2 base-to-arm handoff | Led | Designed the handoff topics, motion limits, heartbeat and base lock during arm operation. |
| Custom ACT training and evaluation | Co-developed | Trained ACT through 200 epochs with a teammate. We evaluated the fixed epoch-30 checkpoint in a separate closed-loop test and classified the 11 failures. |
| Visual SLAM, Nav2, docking and Qwen commands | Collaborated | Matched re-approach limits and stop conditions with the teammate responsible for those modules. |
| YOLO detection | Teammate | A teammate owned this work. |

## Hardware-free checks

```bash
git clone https://github.com/Alpenj/sweepick.git
cd sweepick
python3 -m unittest discover -s research/test -v
```

The repository recorded 33 passing tests on Windows with Python 3.14 on November 3, 2026. GitHub Actions runs `scripts/verify-hardware-free`, headless MuJoCo checks and RoboTwin data-contract tests.

> **Hardware execution on `main` (September 4, 2026 safety notice):** Physical motion and camera streaming are disabled in the `main` executor pending local safety integration. The required checks cover connected-endpoint identity binding, an independent device-side watchdog, and camera serial/cryptographic binding. The physical runs used a separate executor from a [review branch in DAPIER](https://github.com/Alpenj/DAPIER/branches/all?query=sweepick), with a single owner, stop conditions and onsite approval. That executor was kept out of `main`.

Read the [hardware safety procedure (Korean)](docs/HARDWARE_SAFETY_KO.md) before hardware calibration or control. Example commands do not authorize hardware operation.

## Repository guide

| Find | Where |
|---|---|
| Data collection, splits, training, evaluation and failure analysis | [CUSTOM ACT V1 (Korean)](docs/CUSTOM_ACT_V1_KO.md) |
| Hardware roles and experiment-specific conditions | [Project details (Korean)](docs/PROJECT_DETAIL_KO.md) |
| Bimanual MuJoCo simulation | [Simulation](sim/mobile_dual_so101/README.md) |
| Python research code and observation provenance | [Research](research/README.md) · [Observation provenance](research/OBSERVATION_PROVENANCE.md) |
| C++ core, ROS 2 nodes and cross-language contracts | [so101_ros2](so101_ros2/README.md) · [Contracts](contracts/README.md) |
| Architecture and decision records | [Architecture](ARCHITECTURE.md) · [Decision records](docs/architecture/) |
| Documentation index | [Docs (Korean)](docs/README.md) |

SWEEPICK moved out of [Alpenj/DAPIER](https://github.com/Alpenj/DAPIER) on November 3, 2026. Earlier project history remains there under [2ARM_ROBOT](https://github.com/Alpenj/DAPIER/commits/main/2ARM_ROBOT).
