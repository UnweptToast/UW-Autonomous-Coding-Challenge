# Ego-Trajectory & Bird’s-Eye View Mapping

## Overview

This project estimates the trajectory of an ego-vehicle using stereo RGB-D (depth) data, with the traffic light as a fixed world reference. Part A (expected) computes the vehicle’s motion in the ground (BEV) frame. Part B (extra credit, if implemented) overlays other objects (e.g. barrels, golf cart, pedestrians) in the Bird’s-Eye View.

---

## Method

1. **Bounding Box & Pixel Center**  
   - Used provided `bboxes_light.csv` to obtain each frame’s bounding box for the traffic light.  
   - Computed the pixel center of the bounding box:  
     \[
       u = \frac{x_{\min} + x_{\max}}{2}, \quad v = \frac{y_{\min} + y_{\max}}{2}
     \]

2. **Depth → 3D Camera Coordinates**  
   - For each frame, loaded the `.npz` file containing `points` of shape `(H, W, 3)`, which gives the 3-D coordinates \((X, Y, Z)\) for each pixel in the **camera frame** (units in meters).  
   - Extracted the 3-D location for the traffic light using the pixel center `(u, v)`. To reduce noise:
     - Averaged (median) over a small patch around `(u, v)` (e.g. 3×3 or 5×5) to avoid `NaN`/`inf` values.
     - Skipped frames or interpolated when insufficient valid data.

3. **World Frame Definition & Trajectory Extraction**  
   - Defined world frame as specified:  
       • Origin at the ground point directly beneath the traffic light.  
       • +Z axis upward through the traffic light.  
       • At the first frame (t = 0), the vector from the traffic light to the car aligns with +X.  
       • Y is left (so a right-handed frame with +X forward, +Y left, +Z up).
   - Translated each extracted 3-D traffic light point by subtracting the first frame’s traffic light coordinate → makes the light be at origin.  
   - Rotated all ground-plane points by an angle so that the initial vector lies along +X. Used 2-D rotation (ignoring Z) from camera frame → world frame.

4. **Plotting & Visualization**  
   - Plotted trajectory in the X–Y plane (ground, ignoring Z), with equal axis scaling so 1 unit in X equals 1 unit in Y.  
   - Ensured the plot is square (same width & height) and origin at (0,0).  
   - Optional: color gradient over time (frame index) to visualize motion.  
   - Optional: animated BEV video (if implemented) with moving marker or path.

5. **Extras (if implemented)**  
   - Detected and tracked additional objects (e.g., barrels, golf cart, pedestrians) using bounding boxes or color/template-matching in the RGB frames.  
   - Extracted their 3D positions similarly (patch around pixel centers) and plotted them in BEV alongside the ego trajectory.  
   - Rendered color or style differences to distinguish static vs dynamic objects.

---

## Assumptions

- The traffic light is static / fixed in the world (it does not move).  
- The first frame’s camera-to-traffic-light line gives a valid reference direction (i.e. no occlusion or misdetection).  
- Depth and XYZ data are accurate enough near the traffic light bounding box to allow a clean extraction; errors/noise and invalid values are filtered.  
- The ground is approximately level (so projecting to a “ground plane” by dropping Z is reasonable).  
- Interpolation / skipping missing frames does not distort the trajectory in a way that affects overall correctness.

---

## Results

| Metric | Value / Observation |
|---|---|
| Total displacement | ~ \<your total forward distance\> meters |
| Lateral drift | ~ \<how far left/right you moved\> meters |
| Number of valid frames used | \<number\> / \<total frames\> |
| Frames skipped due to invalid depth | \<number\> |
| Smoothness (optional) | (If applied) raw points showed jitter; applying median / interpolation reduced it by \<qualitative or quantitative measure\>. |

- **Trajectory plot**: see `trajectory.png`. The path starts at (0,0), moves mostly forward (+X) with small lateral deviations (+/−Y).  
- **Optional video**: see `trajectory.mp4` (if generated). Shows motion of ego‐vehicle over time, color-coded by frame index.
