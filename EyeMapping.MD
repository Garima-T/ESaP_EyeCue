# Eye and Iris Tracking Notes

This document explains two key methods used in eye-tracking with facial landmarks:

1. **Eye Aspect Ratio (EAR)** – used for blink detection.  
2. **Iris Center Estimation** – used for gaze tracking.

---

## 1. Eye Aspect Ratio (EAR) for Blink Detection

The **Eye Aspect Ratio (EAR)** is a measure of how open or closed the eye is, based on facial landmarks.  
It works by comparing the vertical height of the eye to its horizontal width.

### How It Works
- Select **6 landmarks** around the eye (corners and top/bottom points).  
- Compute two **vertical distances** (between the upper and lower eyelids).  
- Compute one **horizontal distance** (between the left and right corners of the eye).  
- The EAR is the ratio of the average vertical distance to the horizontal distance.  

### Intuition
- When the **eye is open**, the vertical distances are relatively large compared to the horizontal width → EAR is **high**.  
- When the **eye closes**, the vertical distances shrink while the horizontal stays similar → EAR becomes **low**.  
- If the EAR falls below a chosen threshold (commonly around **0.21**) for a few frames, it is counted as a **blink**.  

### Example Code
```python
def eye_aspect_ratio(landmarks, eye_indices):
    p1, p2, p3, p4, p5, p6 = [landmarks[i] for i in eye_indices]
    vertical1 = ((p2.x - p6.x) ** 2 + (p2.y - p6.y) ** 2) ** 0.5
    vertical2 = ((p3.x - p5.x) ** 2 + (p3.y - p5.y) ** 2) ** 0.5
    horizontal = ((p1.x - p4.x) ** 2 + (p1.y - p4.y) ** 2) ** 0.5
    return (vertical1 + vertical2) / (2.0 * horizontal)

## 2. Iris Center Estimation

The **Iris Center Estimation** method finds the center of the iris by averaging the positions of iris landmarks.  
This helps estimate **eye gaze direction**.

### How It Works
- Each iris is represented by **4 landmarks**:  
  - **Left iris** → indices `468–471`  
  - **Right iris** → indices `473–476`  
- Compute the **average x-coordinate** and **average y-coordinate** of these landmarks.  
- Mediapipe provides coordinates in a **normalized range [0, 1]**.  
- Multiply the normalized values by the **image width and height** to get pixel coordinates.  
- The resulting (x, y) pixel location is the **iris center**, which can be drawn on the image.

### Intuition
- The iris moves inside the eye socket as a person looks in different directions.  
- By averaging the landmark positions, we get a **stable center point** that represents where the person is looking.  
- Mapping it to pixel coordinates allows overlays, such as drawing a dot on the iris.

### Example Code
```python
# Compute normalized iris centers
left_iris_x = sum([lm[i].x for i in LEFT_IRIS]) / len(LEFT_IRIS)
left_iris_y = sum([lm[i].y for i in LEFT_IRIS]) / len(LEFT_IRIS)

right_iris_x = sum([lm[i].x for i in RIGHT_IRIS]) / len(RIGHT_IRIS)
right_iris_y = sum([lm[i].y for i in RIGHT_IRIS]) / len(RIGHT_IRIS)

# Convert to pixel coordinates
left_center = (int(left_iris_x * w), int(left_iris_y * h))
right_center = (int(right_iris_x * w), int(right_iris_y * h))

# Example visualization
cv2.circle(image, left_center, radius=2, color=(0,255,0), thickness=-1)
cv2.circle(image, right_center, radius=2, color=(0,255,0), thickness=-1)

