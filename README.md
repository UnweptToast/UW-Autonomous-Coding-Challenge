# Ego-Trajectory & Bird’s-Eye View Mapping

## Overview

This project estimates the trajectory of an ego-vehicle using stereo RGB-D (depth) data, with the traffic light as a fixed world reference. Part A computes the vehicle’s motion in the ground (BEV) frame.

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
   - Extracted the 3-D location for the traffic light using the pixel center `(u, v)`

3. **World Frame Definition & Trajectory Extraction**  
   - Defined world frame as specified:  
       • Origin at the ground point directly beneath the traffic light.  
       • +Z axis upward through the traffic light.  
       • At the first frame (t = 0), the vector from the traffic light to the car aligns with +X.  
   - Translated each extracted 3-D traffic light point by subtracting the first frame’s traffic light coordinate → makes the light be at origin.  
   - Rotated all ground-plane points by an angle so that the initial vector lies along +X. Used 2-D rotation (ignoring Z) from camera frame → world frame.

4. **Plotting & Visualization**  
   - Plotted trajectory in the X–Y plane (ground, ignoring Z), with equal axis scaling so 1 unit in X equals 1 unit in Y.  
   - Ensured the plot is square (same width & height) and origin at (0,0).  
   - Color gradient over time (frame index) to visualize motion.  

---

## Assumptions

- The first frame’s camera-to-traffic-light line gives a valid reference direction (i.e. no misdetection).  
- Depth and XYZ data are accurate enough near the traffic light bounding box.
- The ground is approximately level (so projecting to a “ground plane” by dropping Z is reasonable).  
- Skipping missing frames does not distort the trajectory in a way that affects overall correctness.

---

## Results

- **Trajectory plot**: see `trajectory.png`.
