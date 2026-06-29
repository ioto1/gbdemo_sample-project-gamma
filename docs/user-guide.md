# User Guide — Project Gamma

## Calibration

Before first use, calibrate the camera-to-robot transform:

```bash
python scripts/calibrate.py --robot-ip 172.16.0.101 --checkerboard 7x9
```

This captures 20+ poses and computes the hand-eye calibration matrix.
Results are saved to `config/calibration.yaml`.

## Running Detection + Grasp

### Interactive Mode

```bash
python -m gamma.cli interactive
```

This opens a terminal UI where you can:
1. Place an object in the workspace
2. Press `d` to detect objects
3. Press `g` to plan and execute a grasp
4. Press `q` to quit

### Scripted Mode

```python
from gamma import VisionPipeline, GraspPlanner, RobotController

vision = VisionPipeline(model="yolov11n-gripper")
planner = GraspPlanner(strategy="top_down")
robot = RobotController(ip="172.16.0.101")

# Detect and grasp in a loop
for obj in vision.detect():
    pose = planner.plan(obj.centroid, obj.bounds)
    success = robot.grasp(pose, force_limit=5.0)
    if success:
        robot.move_to("drop_zone")
        robot.release()
```

## Force Feedback Tuning

The adaptive grip controller has three profiles:

| Profile | Force Range | For |
|---------|-------------|-----|
| `delicate` | 0.5–2 N | Electronics, glass |
| `standard` | 2–8 N | Plastic, wood, metal parts |
| `heavy` | 8–25 N | Cast iron, steel blocks |

Set via environment: `export GAMMA_GRIP_PROFILE=delicate`

Or per-grasp in code:

```python
robot.grasp(pose, profile="delicate")
```

## Monitoring

Prometheus metrics are exposed on port **9090**:

```
gamma_grasp_attempts_total{result="success"} 1423
gamma_grasp_attempts_total{result="failure"} 87
gamma_detection_latency_ms{quantile="0.5"} 12
gamma_detection_latency_ms{quantile="0.99"} 45
gamma_grip_force_newtons{profile="standard"} 4.2
```

Grafana dashboard: `dashboards/gamma-overview.json`
