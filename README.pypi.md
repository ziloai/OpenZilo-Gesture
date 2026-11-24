# OpenZilo Gesture

Recognize custom OpenZilo ring gestures in your Python application. Train and export a `*.gesture.json` model with [OpenZilo Gesture Studio](https://pypi.org/project/openzilo-gesture-studio/), then use this package to classify recordings or process a live stream. The recognition package does not require Studio, Qt, or a Bluetooth SDK.

## Install

Requires Python 3.10 or later.

```bash
python -m pip install openzilo-gesture
```

## Recognize a gesture

Pass one complete gesture as an `(N, 6)` array of raw int16 samples. Column order is `ax, ay, az, gx, gy, gz`.

```python
from openzilo_gesture import GestureRecognizer

recognizer = GestureRecognizer.load("demo.gesture.json")
result = recognizer.predict(samples, sample_rate_hz=25)
if result is not None:
    print(result.name, result.confidence)
```

Replace `25` with the actual sample rate, which must match the model's training rate. The recognizer does not resample the input. `predict()` returns `None` if it cannot classify the recording. Confidence is a relative score, not a probability, and is `None` for single-gesture models.

For continuous input, pass batches to `recognizer.feed(batch, sample_rate_hz=25)`. Start with resting samples to establish a motion baseline. Use a separate recognizer for each stream and call `reset()` after reconnecting.

## Documentation

- [English quick start and Python API examples](https://github.com/ziloai/OpenZilo-Gesture/blob/main/README.md)
- [Chinese README](https://github.com/ziloai/OpenZilo-Gesture/blob/main/README.zh-CN.md)
- [API and model format reference (Chinese)](https://github.com/ziloai/OpenZilo-Gesture/blob/main/docs/architecture.md)
