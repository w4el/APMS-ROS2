# YOLOv8-OBB Robotic Simulation & Dataset Automation

**BSc Computer Science Final Year Project — University of Greenwich**  
**Author:** Wael Al Mobarak | **Supervisor:** Dr. Ammarah Raees

---

## Overview

An autonomous robotic system for pick-and-place tasks in simulated industrial environments. The system uses **YOLOv8-OBB** (Oriented Bounding Boxes) to detect objects and estimate their orientation, feeds that data into a **ROS 2 / Gazebo** simulation, and uses **MoveIt2** to plan and execute collision-free grasps with a UR5 robotic arm.

A key contribution is a custom **CAD-to-image synthetic data generation pipeline** that programmatically creates annotated training images to overcome real-world data scarcity for OBB detection.

---

## System Architecture

```
┌─────────────────────┐     ROS 2 topic      ┌──────────────────────┐
│  Perception Module  │ ──/yolov8_obb/detect─▶│  Pick-and-Place Node │
│  (YOLOv8-OBB node)  │                       │  (State Machine)     │
└────────┬────────────┘                       └──────────┬───────────┘
         │ subscribes to                                 │ MoveIt2 API
         │ /camera/image_raw                             ▼
┌────────▼────────────┐                       ┌──────────────────────┐
│  Gazebo Simulation  │◀──────────────────────│  MoveIt2 (MoveGroup) │
│  (UR5 + world)      │   trajectory execute  │  OMPL / RRTConnect   │
└─────────────────────┘                       └──────────────────────┘
```

---

## Modules

### 1. Perception — YOLOv8-OBB ROS 2 Node
- Loads a fine-tuned YOLOv8-OBB `.pt` model via the Ultralytics Python API
- Subscribes to `/camera/image_raw` (640×480 simulated camera on the UR5 end-effector)
- Publishes detections as `custom_interfaces/msg/ObbDetectionArray` on `/yolov8_obb/detections`
- Each detection includes: `class_id`, `confidence`, `center_x`, `center_y`, `width`, `height`, `angle`
- Runs at ~25 FPS on the test hardware

### 2. Synthetic Data Generation Pipeline
- **Input:** 3D CAD models (`.stl`, `.obj`, or `.dae`)
- **Process:** Python scripts using `trimesh` and `PyVista` to:
  - Randomly place objects in virtual scenes with varied position, rotation, and scale
  - Add distractors and diverse backgrounds
  - Render with a virtual camera at varied viewpoints
  - Auto-compute OBB parameters from projected 3D bounding boxes
  - Save annotations in YOLO OBB format (`class_id cx cy w h angle`)
- **Output:** Tens of thousands of images + annotation files, no manual labelling required

### 3. Model Training
- Base: YOLOv8-OBB pre-trained on DOTA / COCO, fine-tuned on synthetic dataset
- Optimizer: AdamW | Scheduler: Cosine annealing | Epochs: 150–250
- Dataset split: train / val / test

### 4. Simulation — ROS 2 + Gazebo
- Robot: **UR5** 6-DOF arm with gripper (defined in URDF/XACRO via `moveit2_obb`)
- Simulated wrist camera publishes to `/camera/image_raw` and `/camera/camera_info`
- Launch files orchestrate Gazebo, `robot_state_publisher`, `ros2_control`, perception node, MoveIt2, and task logic in one command

### 5. Motion Planning — MoveIt2
- Configured via MoveIt Setup Assistant (`kinematics.yaml`, `ompl_planning.yaml`, `controllers.yaml`)
- Planner: **RRTConnect** (OMPL)
- Python API (`moveit_py`) used for Cartesian path planning, scene collision objects, and gripper attach/detach

### 6. Pick-and-Place Task Logic
State machine phases: `WAIT → DETECT → PLAN_APPROACH → GRASP → TRANSPORT → PLACE → HOME`

1. Subscribe to `/yolov8_obb/detections` and select target object
2. Transform 2D OBB → 3D grasp pose (assuming known table plane height)
3. Add target as collision object in MoveIt2 planning scene
4. Execute: pre-grasp approach → lower → close gripper → lift → transport → place → open gripper
5. Remove collision object, return to home

---

## Results

| Metric | Value |
|---|---|
| Pick-and-place success rate | **82%** (100 trials) |
| Inference speed | **~25 FPS** |
| Average cycle time | **35 seconds** |
| Failure — perception errors | 8% |
| Failure — grasp execution | 5% |
| Failure — motion planning | 5% |

---

## Tech Stack

| Component | Technology |
|---|---|
| Object Detection | YOLOv8-OBB (Ultralytics) |
| Middleware | ROS 2 |
| Simulator | Gazebo |
| Motion Planning | MoveIt2 (OMPL / RRTConnect) |
| Robot Model | Universal Robots UR5 (URDF) |
| Data Pipeline | Python, trimesh, PyVista |
| Annotation Format | YOLO OBB (`.txt` per image) |
| Language | Python |

---

## Repository Structure

```
APMS-ROS2/
├── yolov8obb_training/        # YOLOv8-OBB fine-tuning scripts
├── dataset_preparation/       # Synthetic data generation pipeline (CAD-to-image)
│   ├── scene_generator.py
│   ├── renderer.py
│   └── annotator.py
├── moveit2_obb/               # MoveIt2 config package for UR5
│   ├── config/
│   │   ├── kinematics.yaml
│   │   ├── ompl_planning.yaml
│   │   └── controllers.yaml
│   └── launch/
├── perception_node/           # ROS 2 YOLOv8-OBB inference node
├── pick_and_place/            # Task logic state machine node
├── custom_interfaces/         # Custom ROS 2 msg: ObbDetectionArray
└── worlds/                    # Gazebo world files
```

---

## Setup & Running

### Prerequisites
- ROS 2 (Humble or later)
- Gazebo
- MoveIt2
- Python 3.10+
- Ultralytics: `pip install ultralytics`
- trimesh, PyVista: `pip install trimesh pyvista`

### Launch

```bash
# 1. Build the workspace
cd ~/ros2_ws
colcon build --symlink-install
source install/setup.bash

# 2. Launch full simulation (Gazebo + perception + MoveIt2 + task logic)
ros2 launch moveit2_obb sim_pick_and_place.launch.py

# 3. (Optional) Run synthetic data generation
python3 dataset_preparation/scene_generator.py --num-images 10000 --output-dir ./data/synthetic
```

---

## Academic Context

This project was submitted as a Final Year Individual Project (COMP1682) for BSc (Hons) Computer Science at the University of Greenwich, April 2025.
