# LANE-GUARDIAN

A camera-based, software-only system that detects, classifies, and tracks vehicles in traffic video and measures how lane indiscipline affects saturation flow at signalized intersections in Indian cities. No physical sensors needed, just a video.

> Final year major project, Dept. of CSE, GAT, 2025-26.

## Why this project

Indian intersections have high vehicle density, a mixed fleet (cars, autos, two-wheelers, buses, trucks), and weak lane discipline. All of this reduces the saturation flow rate, but it is hard to measure without expensive road sensors. LANE-GUARDIAN does it from ordinary camera footage.

## What it does

- Detects and classifies vehicles with **YOLOv10n** (NMS-free)
- Tracks each vehicle with a persistent ID using **ByteTrack**
- Rebuilds lane geometry from road markings using **RANSAC vanishing-point estimation** and an **inverse perspective transform**, cross-checked against the paths vehicles actually take, so faded or unmarked roads still work
- Lets you add or correct lanes manually, and compensates for camera shake on pole-mounted cameras
- Flags violations with a rule-based engine:
  - Lane Straddling
  - Unsafe Lane Change
  - Illegal Lane Change (crossing a solid line)
  - Wrong-Way driving
  - Overspeeding
  - Unauthorized Parking
- Converts counts to **PCU** (car 1.0, auto 0.8, two-wheeler 0.5, bus/truck 3.0) and computes lane-wise flow, occupancy, mean speed, and an indicative **Level of Service**
- **Review & Fines** step: an operator goes through each violation with its evidence photo, approves it, and adjusts a suggested fine (starting amounts follow the Motor Vehicles (Amendment) Act, 2019 schedule). Decisions are saved into the Excel report.

## Outputs

- Annotated video (boxes, IDs, lane lines, violation flags)
- Excel report: Summary, Violations, Lane Analysis, Vehicle Classes, Tracks, Lane Changes, Calibration
- HTML dashboard
- Evidence photo for every violation

## Pipeline

```
Video -> YOLOv10n detection -> ByteTrack tracking
      -> Lane calibration (vanishing point + IPM + curve fit)
      -> Violation engine -> PCU / flow / LOS
      -> Video + Excel + HTML dashboard -> Review & Fines
```

## Results

**Detector** (custom dataset, 1000+ annotated Indian traffic images, 5 classes: car, two-wheeler, auto-rickshaw, bus, truck)

| Metric | Value |
|---|---|
| mAP@0.50 | 87.4% |
| Car | 90.4% |
| Auto-rickshaw | 89.5% |

**Full pipeline** on a 99-second mixed-traffic clip

| Metric | Value |
|---|---|
| Tracks | 90 |
| Confirmed vehicles | 65 |
| Total flow | 72.8 PCU |
| Violations logged | 30 |
| Most common violation | Lane Straddling (22) |

Also tested on 30-60 second clips from the Electronic City Skywalk, a highway road in China, a road from the IRUVD dataset, and other roads in Electronic City and near Nice Road.

## Tech stack

Python, Ultralytics (YOLOv10), ByteTrack, OpenCV, NumPy, pandas, openpyxl, Gradio, ffmpeg

## Getting started

The app is a single script, built to run on Google Colab or locally.

## How to use
1. **Video**: upload a traffic video
2. **Lanes**: run auto-calibration, or draw lanes by hand and merge them with the automatic result. Calibrations can be saved and reused for the same camera
3. **Analyse**: set the speed limit, parking time, straddle sensitivity and which violations to check, then start
4. **Review & Fines**: go through each violation, approve it, adjust the fine, and export the final report

## Limitations
- Speed and distance are estimates and depend on calibration quality
- Flow numbers are scaled from the clip to an hourly rate, so they are indicative, not a full-hour count
- Pedestrians and low-light/night footage are not covered
- Suggested fines are only a starting point and need a person's approval

