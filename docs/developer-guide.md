# Developer Guide — Project Gamma

## Repository Structure

```
project-gamma/
├── gamma/
│   ├── vision/          # Object detection pipeline
│   │   ├── detector.py
│   │   ├── segmenter.py
│   │   └── calibrator.py
│   ├── planner/         # Grasp pose generation
│   │   ├── dexnet.py
│   │   ├── sampler.py
│   │   └── validator.py
│   ├── controller/      # Real-time grip control
│   │   ├── force_loop.cpp
│   │   └── pid_tuner.py
│   └── learner/         # Offline training
│       ├── dataset.py
│       ├── trainer.py
│       └── augment.py
├── models/              # Pre-trained ONNX models
├── config/              # YAML configuration files
├── scripts/             # Utility scripts
├── tests/               # Unit + integration tests
└── docker-compose*.yml  # Deployment configs
```

## Building from Source

```bash
# C++ modules
cd gamma/controller
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)

# Python environment
python -m venv .venv
source .venv/bin/activate
pip install -e ".[dev,gpu]"
```

## Training a Custom Model

### 1. Collect training data

```bash
python scripts/collect_grasps.py \
    --robot-ip 172.16.0.101 \
    --objects config/training_objects.yaml \
    --grasps-per-object 50 \
    --output data/raw/
```

### 2. Preprocess

```bash
python -m gamma.learner.preprocess \
    --input data/raw/ \
    --output data/processed/ \
    --augment rotate,scale,noise
```

### 3. Train

```bash
python -m gamma.learner.train \
    --data data/processed/ \
    --model dexnet4 \
    --epochs 100 \
    --batch-size 32 \
    --output models/dexnet4-custom.onnx
```

### 4. Evaluate

```bash
python -m gamma.learner.evaluate \
    --model models/dexnet4-custom.onnx \
    --test-data data/test/
```

## Testing

```bash
# Unit tests (no hardware)
pytest tests/unit/ -n auto

# Integration tests (needs simulator)
SIM_HEADLESS=true pytest tests/integration/ -n auto

# Benchmark (run weekly)
python scripts/benchmark.py --runs 1000
```

## Contributing

1. Branch from `main`: `git checkout -b feat/your-feature`
2. Write tests in `tests/unit/` for new code
3. Run `pre-commit run --all-files` before pushing
4. Open an MR — CI runs the full test suite automatically

Last updated: 2026-07-02 14:55:00 UTC
