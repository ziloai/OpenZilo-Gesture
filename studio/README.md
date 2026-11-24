# OpenZilo Gesture Studio

[中文](https://github.com/ziloai/OpenZilo-Gesture/blob/main/studio/README.zh-CN.md)

Studio is a desktop app separate from the Python recognition package. It scans for and connects to OpenZilo rings, plots live six-axis motion data, records gestures, imports JSON and CSV recordings, trains and exports models, tests recognition, and receives or downloads ring audio. Recent firmware starts IMU streaming when Studio subscribes, without requiring a mode switch. Studio enables features according to the capabilities the ring reports on connection.

## Run Studio

Requires Python 3.10 or later, a graphical desktop, and Bluetooth permission to connect a ring.

```bash
python -m pip install openzilo-gesture-studio
openzilo-gesture-studio
```

To run from source, clone the [GitHub repository](https://github.com/ziloai/OpenZilo-Gesture) with Git and run these commands from its root:

```bash
uv sync --locked --all-packages
uv run --locked --all-packages openzilo-gesture-studio
# Alternatively:
uv run --locked --package openzilo-gesture-studio python -m openzilo_gesture_studio

# Launch the legacy interface:
uv run --locked --all-packages openzilo-gesture-studio --ui legacy
# Alternatively:
uv run --locked --all-packages openzilo-gesture-studio-legacy
```

The workspace shares the root `uv.lock` and `.venv/`. Development defaults to Python 3.12; there is no need to activate the environment manually. You can also run `uv run --locked openzilo-gesture-studio` from `studio/`, but default data directories are relative to the current working directory.

The default PySide6 interface uses PySide6 and pyqtgraph, which uv installs. If Qt reports missing system libraries on Debian or Ubuntu, run `sudo apt install libegl1 libgl1 libxkbcommon0 libdbus-1-3`.

Only the legacy interface needs Tkinter. On Linux you may need `sudo apt install python3-tk`; on macOS use a Python with Tk support, or install the matching `python-tk` for Homebrew Python. Tkinter comes from Python or the operating system, not `uv add`. To select an interpreter, run `uv sync --locked --all-packages --python /path/to/python3.12`.

## Interface language

Open **Settings → Language** in the sidebar to choose English, Chinese, or the system default. The change takes effect immediately and persists across restarts, without interrupting the device connection, a recording, or unsaved input. By default, Studio uses Chinese if the system interface language is Chinese and English otherwise.

The legacy Tkinter interface is available only in Chinese.

## Interface reference

Both interfaces use the same data directories. Use this table to find the corresponding controls in the default interface:

| Legacy interface | Default interface |
| --- | --- |
| Device connection area | Device card at the bottom of the sidebar, then the **Device** page |
| Gesture recording and training | **Capture** for recording and datasets; **Train** for settings, training, and export |
| Live six-axis chart | **Live**; **Capture** shows only recorded waveforms |
| Live recognition toggle | **Live** |
| Filtered recording waveform | **Capture** |
| Acceleration diagnostics | **Diagnostics** |
| Ring audio | **Audio** |
| Calibration and firmware | **Device tools** |
| Not available | **About** shows versions, the GitHub repository, and contributors. Studio fetches public information from GitHub only when you open this page, and sends no device or gesture data. |
| Log | Click **Show log** at the bottom of the window to expand the list. You can select and copy multiple rows. |

## Record, train, and export

1. Choose a working directory (default: `gestures/`). Scan for a ring or enter its address directly, then click **Connect**. Studio remembers the last 20 successfully connected devices so you can select them without scanning again. **Clear device history** removes these entries. On macOS, the address is typically a UUID. Studio shows the BLE connection and IMU stream status separately, and allows recording only after fresh IMU samples arrive.
2. Name the gesture, click **Record take**, perform the gesture once, and stop. Rejection calibration requires at least three takes per gesture; aim for five to ten or more. After each take, the waveform shows the same median and low-pass filtering used for training as a solid line, and the raw data as a faint dashed line. You can switch between takes, hide the raw overlay, or select a saved gesture in the dataset to inspect it. By default, takes shorter than 12 samples can be previewed but are not kept. When saving, you can confirm appending to an existing gesture if the name and sample rate match. Disconnecting discards any take still in progress.
3. You can import existing JSON recordings or multiple headerless CSVs. Each CSV contains one take with six columns of raw int16 values: `ax,ay,az,gx,gy,gz`. Studio uses the sample rate selected in the interface for CSV imports. For older JSON files without a sample rate, it assumes 25 Hz.
4. Set the training sample rate to match your data. The low-pass cutoff must be below half the sample rate, and setting the median window to `1` turns median filtering off. Use the impulse/snap preset for snaps and knocks, or the continuous motion preset for movements such as waves. Click **Train all data**. Training and rejection calibration run in the background without blocking the interface. Do not mix recordings with different sample rates in one training run.
5. Export a `*.gesture.json` bundle containing gesture names, HMM parameters, and preprocessing and segmentation settings. If you change the data or training settings, retrain before exporting.
6. To test a saved model, load its bundle, connect a ring streaming at the model's sample rate, and turn on **Live recognition**. Start at rest and return to rest after each gesture. Single-gesture models have no relative confidence; Studio does not display a fixed `0.80`. Older single-gesture models without rejection calibration must be retrained before live recognition can be enabled. Rejection checks are not a probability guarantee. Test for false triggers with negative examples such as ordinary movements and shaking.

### Delete saved gestures

Select one or more items in the dataset and click **Delete selected gestures**. The confirmation lists the names, paths, and take counts. Confirming **permanently deletes the selected JSON files and all takes and training-window annotations they contain. This cannot be undone.** Canceling leaves files and model state unchanged.

- Only the files for the selected rows are affected. Other gestures, unsaved takes, data being captured, and exported models are kept.
- After deletion, Studio refreshes the dataset and saved waveform preview. The current trained model and evaluation results become invalid, so retrain. Separately loaded model bundles are unaffected. Deletion is disabled during training, evaluation, and shutdown.
- Bulk deletion processes each file separately. Failures do not stop the remaining deletions. Studio reports which files were removed, already missing, or could not be deleted. It does not retry or roll back automatically. Resolve any errors before selecting the affected files and retrying. If every deletion fails and the data is unchanged, Studio does not refresh or invalidate the model.

## Ring battery

The device connection area shows the battery percentage and charging status. Studio reads it on connection and refreshes it in the background every 60 seconds. A red **Low battery** warning appears at 20% or less when the ring is not charging. The display shows a dash when disconnected and **Unknown** if the reading fails or is invalid. Missing data is not reported as 0%.

Battery checks do not stop the IMU stream or data collection. Studio defers the check during audio transfers but can still refresh while automatic audio reception is waiting. A failed check logs a warning and retries at the next interval instead of disconnecting. Disconnecting or reconnecting clears the previous reading.

## Live charts and stream recovery

- The live page shows separate autoscaling accelerometer (`ax/ay/az`) and gyroscope (`gx/gy/gz`) charts, with the most recent 10 seconds by default. The horizontal axis is estimated from the sample rate, not device timestamps. Values are raw int16 counts.
- With live recognition on, both charts mark each recognized segment with a purple dashed box and show its gesture name and confidence when available. The boxes align with the sample positions, including segmentation padding but excluding the final rest frames. They do not use the time when the recognition result arrived.
- Pausing the display freezes charts and existing boxes, not sample collection or recording. Resuming returns to the latest data. Boxes scroll with the charts and disappear when they leave the window. Clearing the charts removes their contents and boxes without deleting training data or changing the alignment of later results.
- Missing IMU samples or `DEVICE_BUSY` do not mean BLE disconnected. Studio shows that it is waiting, discards any unfinished take, and keeps completed takes.
- Studio automatically retries the subscription and updates the display after fresh data arrives. You can also retry the IMU stream manually. An acknowledgment alone does not count as recovery.
- If BLE disconnects, Studio backs off between reconnection attempts. Disconnecting or canceling the connection from the interface, or quitting, stops the retries. Protocol errors are still reported.
- **Audio recording and transfers do not pause IMU streaming.** Live and remote recording, history downloads, automatic reception, and saving run independently of the raw stream. The six-axis charts and acceleration display keep updating, even when audio is canceled or saving fails. Calibration, diagnostics, and firmware updates still require exclusive device access. Gesture recording and audio controls remain mutually exclusive. Starting an audio operation stops live recognition; re-enable it when you return to gestures and hold still to rebuild the baseline.

## Acceleration magnitude and direction diagnostics

Connect the ring, wait for fresh IMU samples, and open **Diagnostics**. You do not need a recording or a model.

- The page shows the latest three-axis magnitude, `|a| = sqrt(ax² + ay² + az²)`, in raw counts, g, and m/s², along with converted X/Y/Z components.
- The conversion uses the range reported by the device: `a_g = raw × accel_range_g / 32768` and `a_m/s² = a_g × 9.80665`. If the range is unknown or invalid, Studio shows only raw values and direction instead of guessing ±8g or ±16g.
- Direction is the unit vector `a / |a|`. Studio also shows angles to the sensor's +X/+Y/+Z axes and projections onto the XY, XZ, and YZ planes. A negative-axis component is negative and its angle approaches 180°. Direction is undefined for the zero vector and sensitive to noise near zero. Axis labels follow the firmware; Studio does not remap chip axes.
- The magnitude chart keeps every sample received, not just the last one in each batch. It shows the last 10 seconds and the minimum and maximum within that window, with a limit of 5,000 samples. The horizontal axis is estimated from the sample rate, not device timestamps. Pausing freezes the values, direction, and chart without stopping collection. Clearing them does not affect recording or recognition. Studio clears stale readings when streaming stops, the ring disconnects, or the sample rate or range changes.

**These are accelerometer readings that include gravity (specific force), not gravity-compensated linear acceleration, a direction of motion or velocity, or a world-space heading.** At rest the magnitude is usually about 1g. Changing the ring's orientation and setting it down again changes the direction but leaves the magnitude close to 1g. The axis angles are not Euler angles, and `|a| − 1g` is not three-dimensional gravity compensation. Compensation requires subtracting the gravity vector in the same coordinate system, axis by axis: `a_linear = a − g_body`.

### Magnitude reference bands

Two visualization bands around 1g help compare readings at rest and in motion. They are display guides, not firmware algorithm thresholds.

| Deviation from 1g | Inclusive range | Chart line |
| --- | --- | --- |
| ±3% | 0.97–1.03 g | Green dashed line |
| ±10% | 0.90–1.10 g | Orange dashed line |

First check whether the ring reads close to 1g at rest. Repeat in several orientations, then move or rotate it and see how the magnitude and direction respond. The chart and window maximum show brief peaks. These readings do not go through the training filters, and this view does not change training, recognition, or device parameters.

**Being within a band does not prove that the ring is still, that sensor fusion is valid, or that calibration succeeded.** Slow rotation can also keep the reading near 1g. Stationarity and orientation estimates usually need other information, such as gyroscope readings, variation, and duration. This page does not calculate or replace the device's orientation, gravity-compensated acceleration, or calibration status.

## Ring audio

Connect the ring, stop gesture recording, and choose a save directory (default: `audio/`). These four operations cannot run concurrently:

- **Remote recording:** read the current settings and status. Choose mono or stereo, 8,000–32,000 bit/s, left and right Q8 gain (256 is unity; range 0–1024), and a maximum duration from 20 to 60,000 ms in multiples of 20 ms. The ring stores the recording. **Stop and save** requests an early stop; Studio downloads only after the ring confirms the save, not merely after a stop acknowledgment. If the start request times out, Studio checks status rather than sending it again.
- **Live PCM reception and saving:** Studio subscribes before starting and uses `LIVE_AND_FLASH` to retain a copy on the ring. PyAV decodes in-process without system ffmpeg. Studio saves a WAV only after validating the complete END and does not perform speech recognition. Before starting, you can enable optional monitoring through the computer's default audio output. Monitoring is off by default and cannot be toggled mid-stream. Use headphones to avoid feedback. Playback errors or a backing-up buffer stop playback and log a warning without affecting reception or saving. Playback is cleaned up when reception ends, is canceled, or disconnects. An early stop still requires a complete ending; interrupted or incomplete streams are not saved as complete recordings. Download the ring's copy from history if needed.
- **Automatic reception:** enable save-notification listening, then hold the ring's physical button to record and release it to save. The newer `0x050A` notification triggers a download. Neither listening nor downloading pauses the IMU stream. Stopping reception stops the computer from receiving audio; it does not stop recording initiated by the ring button.
- **Recording history:** stop other audio operations, list the recordings, and download by index. Repeated downloads use new filenames without overwriting existing files or deleting the copy on the ring.

Live monitoring uses sounddevice and PortAudio, not an external player. Linux may require `sudo apt install libportaudio2`. If the library or an output device is unavailable, only playback stops; recording reception continues. Playback does not prove a recording is complete. Test audio quality and latency with the target sound device and ring.

Remote and live recordings use the STOP policy on disconnect. Transfer progress is not recording duration; Studio shows duration only for valid WAV files. For downloads, Studio retains the original `.bin` before converting to WAV. The SDK detects mono or stereo Opus. File decoding requires system ffmpeg: `brew install ffmpeg` on macOS, `sudo apt install ffmpeg` on Ubuntu or Debian, or ffmpeg on `PATH` on Windows. The original file is kept if decoding fails.

Device history remains in `hmm-gesture-studio/devices.json` under your user configuration directory, so previously saved rings are still available. The base directory is `~/Library/Application Support/` on macOS, `%APPDATA%` on Windows, or `$XDG_CONFIG_HOME` (default: `~/.config`) on Linux. Only successful connections are recorded, including the address, name, and last connection time. Nothing is uploaded. An unreadable or unwritable history file does not prevent connecting.

Verify subscription, remote start and stop, hold-and-release recording, live END, downloads, and reconnection on the target ring. Automated tests are not a substitute for hardware testing.

## Local IMU calibration and diagnostics

Open **Device tools** and stop automatic audio reception before calibrating. Calibration and calibration diagnostics take exclusive access to the IMU; while they run, gesture recording and live recognition are unavailable.

- **Six-face accelerometer calibration:** keep the ring attached to the charging case throughout the six measurements. Set it in the indicated position, release it, and click **Continue**. After measuring both directions along each of the three axes, verify with fresh samples on three different axes.
- **Stationary gyroscope calibration:** remove the ring from the charging case, keep it away from magnets, and lay it flat. Wait for three stable windows, collect at least 20 seconds of samples, and verify independently. Full calibration runs the two parts in sequence.
- Studio saves the local configuration atomically only after all checks pass. Canceling or failing collection or verification leaves the previous configuration untouched. Studio does not write calibration parameters to the ring.
- The configuration view shows the bound device, model, exact firmware version, and range. Studio refuses to use a configuration after a firmware or range mismatch. Confirm a configuration reset before recalibrating; Studio does not silently overwrite an incompatible configuration.
- Calibration diagnostics show g, °/s, and which calibration components are actually applied. Stopping the diagnostic restores the raw IMU subscription. **Calibration values never enter gesture training, recording, or recognition.** Existing data and models still use raw integer counts. Readings without applied calibration are not a guarantee of calibrated accuracy.

## Signed firmware updates (OTA)

1. Stop recordings, downloads, calibration, and gesture capture. Keep the ring powered and connected over Bluetooth.
2. Select a signed APP `.ota.zip` and a P-256 PEM public key from an independently trusted source. Do not trust a key bundled with the firmware itself.
3. Run offline signature verification. Check the signed model, board type, allowed starting version, target version, and other metadata before confirming. Test firmware is rejected and raw BIN files are not supported.
4. Studio uses the verified byte snapshot, so replacing the file after confirmation cannot change what's sent. The SDK checks compatibility again, then waits for reboot, reconnection, and full version verification after transfer.
5. While the update runs, other device operations, disconnecting, and normal app exit are blocked until completion or timeout. **100% means the transfer finished, not that the update installed successfully.** Studio does not automatically retry on failure; it ends the current device session. Reconnect manually and check the version before proceeding. Recheck local calibration after a firmware update.

Development tests do not operate or flash real hardware. Verify OTA success, calibration accuracy, and audio behavior on the target ring. To test audio and IMU concurrency, confirm that the IMU stream is updating, then separately start a remote recording and a live PCM recording. Make sure the charts and sample counts keep advancing while recording, stopping, and saving, without an IMU pause or resubscription. Repeat with live monitoring enabled. Simulated tests do not establish real BLE throughput or packet loss.

## Snaps and other short impulses

At 100 Hz, a five-point median filter can remove a real impulse lasting one or two samples as if it were an outlier. A 10 Hz low-pass filter can weaken it further. A smoother waveform is not necessarily better for recognition.

1. Apply the impulse/snap preset. Studio takes the sample rate from the current device or recording. At 100 Hz it uses **no median filter, a second-order 30 Hz low-pass filter, and an eight-frame feature window with four-frame overlap**. The filter starts at the steady state for the first sample, so it does not create an artificial transition from zero.
2. The trigger uses the **norm of the difference between adjacent raw acceleration samples**, with a default threshold of 8,000 raw counts, not g. It keeps 0.12 seconds before and 0.28 seconds after the trigger (41 frames at 100 Hz). A 0.6-second cooldown from the trigger prevents one impulse from firing repeatedly.
3. **Training, offline recognition, and live recognition use the same window rule by default.** Manual recordings can keep the idle periods on either side, but training uses only the samples inside the purple box. Studio rejects a take rather than silently choosing a segment if the impulse is incomplete, there are multiple impulses, or the surrounding data is insufficient. You can also mark the training window manually. Studio never truncates the original recording.
4. Check the preview to see whether the purple box covers the full motion. The solid line within it shows the preprocessed segment used by the model. Retrain after changing the trigger threshold, low-pass cutoff, median window, or manual selection.

### Manual training window

In the filtered waveform view, select a take, enable **Manual selection**, and drag horizontally across the accelerometer or gyroscope chart to include the whole gesture and a little rest before and after it. Both charts show the same box. Drag again to replace it, or restore automatic cropping to remove the override. Each take stores its own box, which must contain at least the number of samples required by the current feature settings (12 by default). Manual selection also works in continuous motion mode.

- For unsaved takes, the window is written when you click **Save / append**. For a saved gesture, editing its window writes to the original file immediately. The annotation remains available after changing takes, refreshing, or restarting.
- Only sample indices are saved as annotations; the complete raw six-axis data is untouched. Training and rejection calibration use the samples inside the manual window, so the take no longer has to pass automatic impulse detection for training.
- **Manual selections do not change the live trigger threshold or window length.** The impulse preset uses only differences between adjacent acceleration samples. Even a large gyroscope change may not trigger it. If no automatic window appears, adjust the trigger threshold or use continuous motion mode, then retrain and test. Match the length and position of the manual window to the live window as closely as possible.
- In holdout evaluation, each training fold uses the manual windows, but testing runs the ordinary recognizer on the full recording, with automatic segmentation in impulse mode. Missed triggers count as failures to recognize; a manual selection is not evidence that automatic detection succeeded.

Continuous motion mode still segments against a baseline, but Studio converts durations to frame counts at the actual sample rate. For example, the five-second maximum is 500 frames at 100 Hz, not the 125 frames used at 25 Hz. Evaluate before sharing one segmentation preset across different kinds of gestures.

## Recognition, rejection, and evaluation

Training fits HMMs on standardized 24-dimensional features, applies a lower bound to state variances, and converts parameters back to the original feature space before export. By default, the GUI also calibrates rejection by holding out individual takes. This requires at least three takes per gesture and checks candidate segments against held-out score floors, segment lengths, and impulse strength in impulse mode. A single-gesture model no longer treats every motion as that gesture, but **positive samples alone do not establish reliable rejection of unknown movements**.

The **v2 export format** carries the new filter initialization, impulse segmentation, and rejection settings. Loading a v1 model retains its original filtering, segmentation, and offline model scoring. **Existing models are not upgraded automatically. Retrain and export them again.**

Click **Holdout evaluation** to run a background check with at least four valid takes per gesture. Each outer fold holds out one take and uses the rest for training and inner rejection calibration. Test data is not used for normalization, thresholds, or model fitting. A single-class evaluation reports only the **positive pass rate**, not overall recognition accuracy or a false-trigger rate. The log shows prediction counts by gesture and the number of takes that were too short. Evaluation does not replace the current model, and does not run automatically during training. You still need independent continuous recordings and negative examples.

```bash
uv run --locked --package openzilo-gesture-studio python -m openzilo_gesture_studio.evaluation --data sample_data
```

An earlier five-fold comparison using the repository's 25 Hz examples, legacy segmentation and filter defaults, and no rejection yielded **19/30 (63.3%)** for the old pipeline and **20/30 (66.7%)** after adding only numerical regularization. The gain was small, and downward and rightward movements remained easy to confuse. That comparison did not measure the impulse preset or rejection. It served as a development check, not an independent test after tuning or an accuracy estimate for real-world use. Record at least five to ten takes per gesture, keep the ring orientation and sample rate consistent, and collect a separate test session. Leave a short rest before and after each take, perform one gesture per take, and inspect the filtered preview for noise, differences between takes, and excessive idle time.

## Train without the GUI

From the repository root, train on the six sample gestures:

```bash
uv run --locked --package openzilo-gesture-studio openzilo-gesture-train --data sample_data --output exports/demo.gesture.json
```

This checks that the pipeline works; it is not an independent accuracy test. Studio does not directly import old `*.pkl` models. Retrain from recordings in JSON to avoid pickle's code-execution risk and to include the preprocessing parameters in the export.

After export, the application using the model only needs the [OpenZilo Gesture recognition package](https://pypi.org/project/openzilo-gesture/), without Studio, Tkinter, or the OpenZilo SDK. Use `uv sync --locked` at the repository root to sync only `openzilo-gesture`, or `uv add openzilo-gesture` from another uv project to install the published package.

```python
from openzilo_gesture import GestureRecognizer

recognizer = GestureRecognizer.load("exports/demo.gesture.json")
result = recognizer.predict(samples, sample_rate_hz=25)  # Raw values, shape (N, 6)
for result in recognizer.feed(batch, sample_rate_hz=25):
    print(result.name, result.confidence)
```

The project is licensed under MPL-2.0. Studio installs `openzilo[pcm]>=0.6.3,<0.7` from PyPI. Training and recognition run on the computer.
