# Project Gamma — Gripper Intelligence Platform

> AI-powered grasp planning and object recognition for Franka robot grippers.

## Overview

Project Gamma enhances Franka robot grippers with computer vision and machine learning
capabilities. It provides real-time object detection, grasp pose estimation, and
force-feedback optimisation so robots can handle a wide variety of objects without
manual programming.

## Core Capabilities

- **Object detection** — YOLOv11-based model identifies objects in the workspace
- **Grasp planning** — generates 6-DoF grasp poses using Dex-Net 4.0
- **Force feedback** — adaptive grip force based on object material and weight
- **Bin picking** — randomised object arrangement with collision-aware path planning
- **Learning mode** — record and replay successful grasps to improve over time

## Architecture

![System Diagram](assets/system-diagram.pdf)

| Module | Technology | Description |
|--------|-----------|-------------|
| `gamma-vision` | Python + ONNX Runtime | Object detection + segmentation |
| `gamma-planner` | C++ + libfranka | Grasp pose generation |
| `gamma-controller` | C++ | Real-time force/torque control loop |
| `gamma-learner` | Python + PyTorch | Offline training from recorded grasps |

## Quick Start

```bash
git clone https://gitlab.franka.de/franka-dev/ai/project-gamma.git
cd project-gamma
docker-compose -f docker-compose.gpu.yml up -d
```

Requires an NVIDIA GPU with CUDA 12+. For CPU-only:

```bash
docker-compose up -d
```

## Model Zoo

Pre-trained models are available in `models/`:

- `yolov11n-gripper.onnx` — object detector (78 MB)
- `dexnet4-franka.onnx` — grasp quality CNN (245 MB)
- `material-classifier.onnx` — surface material predictor (34 MB)

Last updated: 2026-07-02 14:55:00 UTC
