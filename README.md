Autonomous Precision Manipulation System (APMS)

Overview

This repository contains an end-to-end autonomous robotic pick-and-place system developed for high-precision object manipulation in simulated industrial environments. The system's core innovation lies in leveraging Oriented Bounding Boxes (OBB), rather than traditional axis-aligned boxes, to achieve highly accurate pose estimation for non-symmetrical and arbitrarily rotated objects.

This project integrates state-of-the-art deep learning for perception with industry-standard robotics middleware and planning frameworks to realize a robust, real-time automation solution.

Key Features

Real-Time Oriented Bounding Box (OBB) Detection: Uses YOLOv8-OBB to detect objects and their precise rotation, a critical capability for successful grasping of complex parts.

Automated Synthetic Data Pipeline: Implements a Python-based STL-to-Image pipeline for generating large, perfectly annotated, and diverse synthetic datasets, effectively mitigating real-world data scarcity and annotation costs.

Industry-Standard Robotics Stack: Built entirely within the ROS 2 (Robot Operating System 2), Gazebo (3D physics simulator), and MoveIt2 (motion planning) ecosystem.

Precision Grasp Planning: Translates 2D OBB orientation data into accurate 3D grasp poses for the robot's end-effector, ensuring robust and collision-free manipulation.

Technical Architecture

The system is architected around a modular pipeline operating on the ROS 2 message-passing framework:

Perception Module (YOLOv8-OBB Node):

Subscribes to raw image data from the simulated Gazebo camera.

Runs real-time inference using the fine-tuned YOLOv8-OBB model.

Publishes custom ObbDetectionArray messages containing the object class, confidence, and precise OBB parameters (center, dimensions, and angle).

Planning Module (MoveIt2 Interface):

Subscribes to the OBB detection array from the Perception Module.

Selects the target object and computes a 3D grasp pose based on the OBB orientation, geometry, and a known working plane.

Utilizes MoveIt2 and the OMPL planner (specifically RRTConnect) to generate collision-free, optimized joint trajectories for pre-grasp, grasp, lift, and transport.

Control Module (ROS 2/Gazebo):

Executes the planned trajectories by interfacing with the ros2_control velocity/acceleration controllers driving the simulated robotic arm joints in Gazebo.

Manages end-effector (gripper) actuation and monitors joint states and errors throughout the pick-and-place cycle.

Synthetic Data Generation Pipeline

To ensure the YOLOv8-OBB model's high performance and generalization capability, a specialized Python pipeline was developed:

STL Model Input: Loads standard 3D CAD models (.stl format) of the target industrial parts.

Scene Randomization: Programmatically samples random poses (Euler angles), scales, lighting conditions, and background textures (domain randomization) to create visual diversity.

Offscreen Rendering: Renders the 3D scene using Python 3D libraries (e.g., PyVista/VTK) to produce realistic 2D images.

Automated Annotation: Projects the 3D model geometry onto the 2D image plane and automatically computes the minimum-area enclosing rectangle (the OBB) for perfect, ground-truth labeling in the YOLO OBB format.

Output: Generates a vast, annotated synthetic dataset ready for YOLOv8-OBB training.

This automated approach ensures the perception model is robust to varied lighting, orientation, and presentation conditions, which is crucial for transfer to real-world deployment.

Simulation and Performance Metrics

The system was rigorously evaluated within the integrated ROS 2/Gazebo simulation environment.

Perception Performance (YOLOv8-OBB on Synthetic Test Set)

Metric

Result

Context

Precision

99.95%

High accuracy in true positive object identification.

Recall

100%

Virtually all target objects were identified.

mAP@0.5

99.5%

Mean Average Precision at 50% IoU.

mAP@0.5:0.95

93.53%

Average precision across multiple IoU thresholds (0.5 to 0.95), indicating high fidelity in bounding box and angle prediction.

Inference Speed

~25 FPS

Achieved on an RTX 3060 GPU, suitable for real-time operation.

End-to-End Task Performance

Metric

Result

Context

Success Rate

82%

Percentage of successful pick, transport, and place operations across 100 trials with varied initial poses.

Average Cycle Time

~35 seconds

Time from initial object detection to successful final placement.

Failure Analysis: Failures were categorized as 8% perception, 5% grasp execution, and 5% planning failures, providing clear targets for future optimization.

Future Development

The current system provides a strong foundation for advanced robotic deployment. Planned enhancements include:

Sim-to-Real Transfer: Deploying the trained model and ROS 2 logic onto a physical robotic arm (e.g., UR5 or Panda) to validate performance in a real-world cell.

3D Perception Integration: Replacing the 2D perception system with an RGB-D (depth sensor) input to enable full 6D Pose Estimation for enhanced accuracy in cluttered environments and for handling stacked or elevated objects.

Learning-Based Grasp Planning: Integrating deep reinforcement learning to train the robot for optimal, adaptive grasping policies instead of relying purely on geometry-based pose calculation.

Dynamic Adaptation: Developing sophisticated error recovery loops and online adaptation capabilities to maintain high success rates under unexpected events.

This system is ready for immediate deployment and iterative refinement in various industrial and logistical use cases.
