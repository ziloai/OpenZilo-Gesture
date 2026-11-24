# OpenZilo Gesture Studio

[中文](https://github.com/ziloai/OpenZilo-Gesture/blob/main/studio/README.zh-CN.md)

Record, train, and test custom gestures from an OpenZilo ring in a desktop app. Studio connects over Bluetooth, shows live six-axis motion data, imports JSON or CSV recordings, and exports `*.gesture.json` models for use with [OpenZilo Gesture](https://pypi.org/project/openzilo-gesture/). It also includes ring audio, diagnostics, and device tools; availability depends on the connected ring.

## Run Studio

Requires Python 3.10 or later and a graphical desktop. To connect a ring, enable Bluetooth and grant the app or terminal permission to use it.

```bash
python -m pip install openzilo-gesture-studio
openzilo-gesture-studio
```

If Qt reports missing system libraries on Debian or Ubuntu, install them with `sudo apt install libegl1 libgl1 libxkbcommon0 libdbus-1-3`.

## Record, train, and export

1. Connect your ring from the device card in the sidebar. Wait for live motion data.
2. On **Capture**, record at least three takes per gesture (five to ten is a better starting point), or import existing recordings.
3. On **Train**, choose a preset that suits the movement and train a model.
4. On **Live**, test recognition. Return to rest between gestures and try everyday movements to check for false triggers.
5. On **Train**, export the model for use in your Python application.

## Interface language

Open **Settings → Language** in the sidebar to choose English, Chinese, or the system default. Changes take effect immediately and persist across restarts. By default, Studio uses Chinese if the system interface language is Chinese and English otherwise.

The legacy Tkinter interface (`openzilo-gesture-studio --ui legacy`) requires Tkinter and is available only in Chinese.

## Documentation

- [English quick start and workflow](https://github.com/ziloai/OpenZilo-Gesture/blob/main/README.md)
