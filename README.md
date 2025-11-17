# AI-Powered Driver Assistance System (ADAS)

This repository contains an integrated Advanced Driver Assistance System combining driver monitoring (drowsiness and behavioral analysis) and forward-view road perception (traffic sign recognition and lane/turn detection). The project targets Raspberry Pi 5 for edge deployment and supports a desktop test mode.

**Main features**
- Driver drowsiness detection: EAR, MAR, yawning counter, head-down detection
- Traffic sign recognition: GTSRB-trained MobileNetV2 -> TensorFlow Lite for edge inference
- Sharp-turn detection via lane curvature estimation
- Vehicle speed management dashboard: percentage-based speed control
- Integrated Pygame dashboard for overlays and audio alerts

## Files of interest
- `integrated_driver_monitoring.py` – Main integrated application (threads, dashboard, drowsiness + lane)
- `Facerecognition/` – Drowsiness helper scripts and assets (alarm.wav)
- `Road Lane detection/Advanced-Lane-Lines/` – Lane detection pipeline and prepared test video
- `PROJECT_REPORT.md` – Detailed project report and methodology (created)

## Quick start (Raspberry Pi)
1. Clone project to Pi and enter project folder:

```bash
cd ~/Desktop
# copy or clone repo here
cd "M&I PROJECT - Updated"
```

2. Create and activate venv (Python 3.11 recommended):

```bash
python3.11 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

3. Run the integrated system:

```bash
python integrated_driver_monitoring.py
```

## Exit methods
- Press `Q` or `ESC` in the window
- Click the window Close (X) button
- Press `Ctrl+C` in the terminal (signal handler catches it)

## Notes and Raspberry Pi Tips
- All GUI operations are performed in the main thread to avoid Qt threading issues.
- If MediaPipe is used, ensure `numpy<2.0` in that environment; YOLO/PyTorch may require a separate venv due to different numpy requirements.
- For better performance on Raspberry Pi 5, consider increasing GPU memory and using TensorFlow Lite with quantized models.

## Methodology (short)
- Driver monitoring uses MediaPipe (or Dlib) landmarks and computes EAR/MAR for blink and yawn detection.
- Road perception runs a TFLite classifier trained on GTSRB; outputs are fused with lane curvature to detect sharp turns.
- All frames are processed in worker threads; the main thread composes combined frames and handles display + input.

## Flowchart
Use a Markdown renderer that supports Mermaid to visualize the flowchart included in 'graph TD
  %% Initialization and Setup (A to C)
  A[Start System] --> B{Init Hardware}
  B -->|OK| C[Load Models & Calibrate]
  C --> D[Start Parallel Threads]

  %% Parallel Threads Start (D to D1/D2)
  D --> D1[Driver Monitoring Thread]
  D --> D2[Forward View Thread]

  %% Driver Monitoring Thread (D1 to E8)
  subgraph Driver Monitoring
    D1 --> E1[Capture Camera Frame]
    E1 --> E2[MediaPipe Face Mesh]
    E2 --> E3[Compute EAR/MAR/HeadPose]
    E3 --> E4{Driver State Logic}
    E4 -->|NORMAL| E5[Update Counters]
    E4 -->|DROWSY| E6[Play Alarm + Slow Down]
    E4 -->|DRIVER_DOWN/MISSING| E7[Play Alarm + Emergency Stop]
    E5 --> E8[Share Driver Frame]
    E6 --> E8
    E7 --> E8
    style E4 fill:#ffcccc
    style E6 fill:#ffddaa
    style E7 fill:#ff6666
  end

  %% Forward View Thread (D2 to F10)
  subgraph Forward View Processing
    D2 --> F1[Capture Video Frame]
    F1 --> F2[Lane Detection Pipeline]
    F2 --> F3[Perspective Transform + Polynomial Fit]
    F3 --> F4[Compute Lane Curvature]
    F4 --> F5{Sharp Turn?}
    F5 -->|Yes| F6[Show Turn Warning]
    F5 -->|No| F7[Normal Lane Display]
    F6 --> F8[Run TFLite Sign Classifier]
    F7 --> F8
    F8 --> F9[Overlay Driver Monitor]
    F9 --> F10[Share Combined Frame]
    style F5 fill:#aaddff
    style F6 fill:#ffaa66
  end

  %% Main Display Loop and Shutdown (E8/F10 to M)
  E8 --> G[Main Thread Display Loop]
  F10 --> G

  G --> H[cv2.imshow Combined Frame]
  H --> I[Handle Keyboard Input]
  I --> J{Exit Signal?}
  J -->|No| D
  J -->|Yes| K[Cleanup Threads]
  K --> L[cv2.destroyAllWindows]
  L --> M[Shutdown Complete]
  style J fill:#ccffcc'
`.

## License & Contact
(Include your license or institution info here.)

---
Generated: November 16, 2025
