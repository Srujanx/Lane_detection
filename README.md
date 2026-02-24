#  Self-Driving Car — Lane Detection Pipeline

> A classical Computer Vision pipeline for lane detection, obstacle avoidance, path planning, and autonomous control. **No neural networks. No training data. Just math.**

---

##  Overview

This project implements a **14-stage classical CV pipeline** that simulates a self-driving car's perception, decision-making, and control system using a realistic dashcam perspective. The system handles real-world edge cases including partial lane occlusion, low visibility, and dynamic obstacles — all without any trainable parameters.

---

##  Pipeline Architecture

```
Camera Frame (640×480 @ 30fps)
        │
        ▼
 [1]  ROI Masking          → Trapezoidal mask removes sky & hood
        │
        ▼
 [2]  Color Enhancement    → HLS (white) + HSV (yellow) binary mask
        │
        ▼
 [3]  Edge Detection       → Gaussian blur (5×5) → Canny (40/130)
        │
        ▼
 [4]  Hough Transform      → Probabilistic HoughLinesP → line segments
        │
        ▼
 [5]  Line Separation      → Position + slope filter → left / right groups
        │
        ▼
 [6]  Polynomial Fit       → Quadratic fit: x = ay² + by + c
        │
        ▼
 [7]  EMA Smoothing        → α-weighted average across frames (α=0.75)
        │
        ▼
 [8]  Extrapolation        → One lane lost? Reconstruct from the other
        │
        ▼
 [9]  Curvature + Offset   → Radius (m) + lateral offset (m)
        │
        ▼
[10]  Obstacle Detection   → Frame diff → contours → TTC
        │
        ▼
[11]  FSM Decision         → 6-state machine → speed target + brake flag
        │
        ▼
[12]  A* Path Planning     → Occupancy grid → avoidance path
        │
        ▼
[13]  PID Control          → Steering angle (°) + throttle [-1, 1]
        │
        ▼
[14]  Visualisation        → Lane overlay + HUD + mosaic
```

---

## Techniques Used

| Stage | Technique | Output |
|---|---|---|
| ROI Masking | Trapezoidal crop | Masked image |
| Color Enhancement | HLS + HSV thresholding | Binary mask |
| Edge Detection | Gaussian blur + Canny | Edge map |
| Hough Transform | Probabilistic HoughLinesP | Line segments |
| Line Separation | Slope-weighted clustering | Left / Right groups |
| Polynomial Fitting | Quadratic fit (degree 2) | Coefficients {a, b, c} |
| EMA Smoothing | Exponential Moving Average | Smoothed coefficients |
| Curvature & Offset | Analytical calculus | Radius (m), Δx (m) |
| Obstacle Detection | Frame differencing + contours | Obstacles + TTC |
| Decision Making | Finite State Machine | State + target speed |
| Path Planning | A* on occupancy grid | Path cells |
| Control | PID Controller | Steer °, throttle |

---

##  Finite State Machine (FSM)

| State | Trigger | Speed |
|---|---|---|
| `CRUISE` | All clear | 100 km/h |
| `DECELERATING` | Obstacle ahead OR sharp curve (R < 400m) | 50 km/h |
| `EMERGENCY_STOP` | Time to Collision (TTC) < 2.5 sec | 0 km/h |
| `LANE_LOST` | Both lanes absent > 8 frames | 18 km/h |
| `LOW_VISIBILITY` | Mean brightness < 60 | 36 km/h |
| `RECOVERING` | Threat resolved (10 clean frames to exit) | 50 km/h |

---

##  Edge Cases Handled

- **Zero Detection** — Returns last known good coefficients from EMA buffer if no lines found
- **Single Lane Visibility** — Extrapolates missing lane by shifting existing curve by standard lane width (3.7 m)
- **Insufficient Data** — Degrades gracefully from quadratic (degree 2) to linear (degree 1) fit
- **Numerical Stability** — Validates against NaN / Inf in polynomial coefficients; ignores corrupt frames
- **Geometric Guards** — Division-by-zero protection for near-zero lane widths
- **Horizontal Noise** — Slope filter rejects shadows and road cracks
- **Low Visibility** — Flags `LOW_VISIBILITY` for dark/blurry frames while maintaining path memory
- **Memory Management** — Validates input dimensions against initialization settings

---

##  Advantages

- **Fully interpretable** — every decision traces back to a mathematical formula; failures are debuggable
- **Zero training data required** — works immediately on any road scene
- **Robust edge-case handling** — EMA smoothing, single-lane extrapolation, and graceful FSM degradation

##  Limitations

- Colour thresholding is sensitive to extreme lighting (sunset glare, heavy rain, snow)
- Oncoming vehicles won't trigger emergency stop until TTC < 2.5 sec
- Frame-differencing obstacle detection misses stationary obstacles present in the very first frame

---

##  Future Improvements

- **Bird's-eye (perspective) transform** before polynomial fitting to remove perspective distortion and improve curvature accuracy on sharp bends
- **Adaptive Canny thresholds via Otsu's method** to handle varying illumination automatically instead of fixed low/high values

---

##  Requirements

```bash
pip install opencv-python numpy matplotlib
```

---

##  Usage

```bash
jupyter notebook CV_Assignment_1_final.ipynb
```

---

##  Project Structure

```
├── CV_Assignment_1_final.ipynb   # Main pipeline notebook
├── README.md                     # This file
```

---

##  License

This project was developed as part of a Computer Vision course assignment.
