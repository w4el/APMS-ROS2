# Autonomous Precision Manipulation System (APMS)

## Overview

[cite_start]This repository contains an end-to-end autonomous robotic pick-and-place system developed for high-precision object manipulation in simulated industrial environments[cite: 7, 82]. [cite_start]The system's core innovation lies in leveraging **Oriented Bounding Boxes (OBB)**, rather than traditional axis-aligned boxes (AABBs) [cite: 84, 127][cite_start], to achieve highly accurate pose and orientation estimation for non-symmetrical and arbitrarily rotated objects[cite: 126, 193]. [cite_start]This capability is critical for complex, high-mix, low-volume manufacturing applications like dynamic **bin picking**, intricate component **assembly**, and precise **quality inspection** where parts may be presented at any angle and successful grasping relies on aligning the gripper with the object's principal axis[cite: 176, 179].

[cite_start]This project integrates state-of-the-art deep learning for high-speed perception with industry-standard robotics middleware and advanced planning frameworks (**ROS 2, Gazebo, MoveIt2**) [cite: 8, 88, 89] [cite_start]to realize a robust, real-time automation solution that serves as an excellent template for sim-to-real deployment[cite: 62]. [cite_start]The entire framework emphasizes **efficiency, modularity, and adaptability**, requirements paramount in modern industrial automation[cite: 123, 175].

## Key Features

* [cite_start]**Real-Time Oriented Bounding Box (OBB) Detection:** Uses a fine-tuned version of **YOLOv8-OBB** to detect objects and their precise rotation in 2D space[cite: 8, 196]. [cite_start]Standard AABBs often lead to wasted area and ambiguous grasp points for elongated objects, making grasp planning unreliable[cite: 179, 193]. [cite_start]This OBB approach provides the object's true minimum bounding area, which is essential for determining the narrowest gripping axis for a successful pinch or power grasp, minimizing the risk of slippage during transport[cite: 194]. [cite_start]The model achieves an Inference Speed of about **25 FPS on an RTX 3060** [cite: 50, 446][cite_start], demonstrating its capability for real-time robotic applications[cite: 476, 746].

* [cite_start]**Automated Synthetic Data Pipeline:** Implements a versatile Python-based **STL-to-Image pipeline** for generating large, perfectly annotated, and diverse synthetic datasets[cite: 8, 27, 87]. [cite_start]Manual OBB annotation is highly labor-intensive and error-prone due to the need for precise angle measurement [cite: 210][cite_start]; this automated pipeline eliminates that bottleneck, enabling infinite, perfectly annotated dataset creation[cite: 65, 888]. [cite_start]It effectively mitigates real-world data scarcity and annotation costs while comprehensively covering rare **edge cases and rotational variance** that are nearly impossible to capture exhaustively through real-world collection[cite: 215].

* [cite_start]**Industry-Standard Robotics Stack:** The system is built entirely within the robust, modular, and asynchronous **ROS 2**, **Gazebo** (high-fidelity 3D physics simulator), and **MoveIt2** (advanced motion planning framework) ecosystem[cite: 8, 241, 245]. [cite_start]This architecture choice ensures **scalability, interoperability, and stability**, handling the complex "hard problems" of robotics—like collision avoidance and trajectory generation—and providing a blueprint for the development of similar autonomous systems[cite: 162, 248].

* [cite_start]**Precision Grasp Planning (OBB-to-6D Pose):** The system translates the detected 2D OBB orientation and position data into an actionable **3D grasp pose** (x, y, z, roll, pitch, yaw) for the robot's end-effector[cite: 379]. This transformation node acts as the crucial link between the perception domain and the action domain. [cite_start]By assuming the object rests on a known plane, the OBB angle is mapped directly to the end-effector's crucial **yaw angle**[cite: 380]. [cite_start]This step is fundamental to the system's success, guaranteeing orientation-aware and collision-free manipulation paths[cite: 372].

---

## Technical Architecture

[cite_start]The system is architected around a modular pipeline orchestrated within the **ROS 2** framework[cite: 282, 305], leveraging its Data Distribution Service (DDS) communication layer for low-latency, reliable data exchange:

1.  **Perception Module (YOLOv8-OBB Node):**

    * [cite_start]**Input Handling:** A dedicated Python ROS 2 node subscribes to raw image data (e.g., `/camera/image_raw`) published by the simulated camera in Gazebo[cite: 22, 350, 436]. [cite_start]The `cv_bridge` library is used for the high-performance conversion of ROS 2 image messages[cite: 437].
    * [cite_start]**Inference Pipeline:** Performs inference using the loaded, trained YOLOv8-OBB model[cite: 352, 438]. [cite_start]The model achieves real-time speeds, contributing minimally to the overall cycle time[cite: 871].
    * [cite_start]**Output Publishing:** Publishes the detection results as custom ROS 2 messages, specifically a dedicated **ObbDetectionArray**[cite: 25, 355, 441]. [cite_start]This message contains the class, confidence, and full OBB parameters[cite: 442].

2.  **Planning Module (MoveIt2 Interface):**

    * [cite_start]**Grasp Pose Computation:** This node computes the 3D grasp pose by transforming the 2D OBB information, often via table plane estimation[cite: 37, 379]. [cite_start]The OBB's orientation is crucial for alignment[cite: 296, 380].
    * [cite_start]**Dynamic Scene Update:** The detected object is added as a collision object to the MoveIt2 planning scene[cite: 37, 372].
    * [cite_start]**Trajectory Generation:** MoveIt2 is utilized via OMPL (RRTConnect) to generate collision-free, optimized joint trajectories[cite: 22, 37, 381].

3.  **Control Module (ROS 2/Gazebo):**

    * [cite_start]**Execution & Safety:** Executes the planned trajectories via `ros2_control` (velocity/accel scaling) [cite: 38, 497] [cite_start]and drives the Gazebo joints[cite: 22, 302]. [cite_start]The core logic is orchestrated by a **State Machine** managing phases like Detection $\to$ Pose Computation $\to$ Planning $\to$ Grasp $\to$ Place $\to$ Recovery[cite: 26, 530].
    * [cite_start]**Monitoring:** Monitors Joint States and Errors, with logic to retry on failure[cite: 38].

---

## Synthetic Data Generation Pipeline

[cite_start]The methodology explicitly focuses on synthetic data generation to overcome the challenge of data scarcity and manual annotation costs in OBB detection[cite: 87, 211, 310].

1.  [cite_start]**STL Model Input and Scene Generation:** Starts with the **STL Model** [cite: 28, 313] [cite_start]and involves sampling **Random Poses & Scales** (Euler angles, uniform scale jitter) [cite: 30, 315] [cite_start]for diverse scene composition[cite: 314].

2.  [cite_start]**Aggressive Domain Randomization:** The process uses **Offscreen Rendering (PyVista)** [cite: 30, 319] [cite_start]and incorporates **Domain randomization** for **lighting and BGs**[cite: 30]. [cite_start]This pipeline is designed to be runnable in a standard Python environment, without direct dependence on Blender[cite: 318].

3.  [cite_start]**Automated Annotation (The Core Strength):** The pipeline automates the annotation by projecting **3D Corners to 2D** and using this information to **Compute min-area rectangle for OBB**[cite: 30, 321].

4.  [cite_start]**Dataset Preparation:** The process concludes by exporting **YOLOv8-OBB Labels** (`cls cx cy w h angle`) [cite: 30] [cite_start]and running a script to perform **Train/Val/Test Split** (class & rotation balance)[cite: 30, 328]. [cite_start]This entire workflow is documented as the "Synthetic Data Generation Pipeline"[cite: 32].

---

## Simulation and Performance Metrics

[cite_start]The system's performance metrics demonstrate the efficacy of combining OBB perception with the integrated robotic pipeline[cite: 45, 62].

### Perception Performance (YOLOv8-OBB on Synthetic Test Set)

| **Metric** | **Result** | **Context** |
| :--- | :--- | :--- |
| **Precision** | **99.95** | [cite_start]High accuracy in true positive object identification[cite: 50]. |
| **Recall** | **100** | [cite_start]All target objects were identified by the model[cite: 51]. |
| **mAP@0.5** | **99.5** | [cite_start]Mean Average Precision at 50% IoU[cite: 51, 54]. |
| **mAP@0.5:0.95**| **93.53** | [cite_start]Average precision across multiple, stringent IoU thresholds[cite: 50]. |
| **Inference Speed** | **\~25 FPS** | [cite_start]Achieved on an RTX 3060 [cite: 50][cite_start], suitable for real-time execution[cite: 8, 446]. |

### End-to-End Task Performance

| **Metric** | **Result** | **Context** |
| :--- | :--- | :--- |
| **Success Rate** | **82%** | [cite_start]The overall success rate of the pick-and-place task[cite: 58, 753]. |
| **Average Cycle Time**| **\~35 s** | [cite_start]Time from object search to successful placement[cite: 58, 869]. |

[cite_start]**Failure Analysis:** Failures were categorized to identify system weaknesses: **8% perception** errors, **5% grasp** execution failures, and **5% planning** failures[cite: 60, 754].

---

## Future Development

[cite_start]The project conclusions propose several future avenues for enhancement[cite: 67, 922]:

* [cite_start]**Sim-to-Real Transfer and Physical Deployment:** Transferring the framework to a physical robot arm to validate capabilities in a real-world setting[cite: 68, 924].
* [cite_start]**3D Perception and 6D Pose Integration:** Integrating **RGB-D/LIDAR** for sensor fusion to achieve full **6D pose estimation**[cite: 69, 927].
* [cite_start]**Advanced Planning:** Exploring **Learning-based planners with contact physics** to improve grasp success rates[cite: 69, 933].
* [cite_start]**Adaptation and Recovery:** Implementing sophisticated **Online adaptation and error recovery loops** to enhance system robustness[cite: 69, 938].
