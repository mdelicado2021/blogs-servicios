# Marker Visual Localization with AprilTags and Odometry

This project implements a complete solution for the Marker Visual Localization exercise from RoboticsAcademy.  
The robot estimates its global pose using AprilTag detections, fuses this information with odometry, and navigates autonomously to approach a visual beacon.

## System Overview

The system performs the following tasks:

- Detection of AprilTags from the onboard camera.
- Estimation of the robot’s relative pose with respect to detected tags.
- Transformation of visual measurements into the global (world) frame.
- Fusion of visual localization with odometry to maintain pose estimation when tags are not visible.
- Autonomous navigation based on perception.

## External Interfaces and Libraries

The implementation relies on the following components:

- HAL: Provides access to the robot camera, odometry, and motor commands.
- WebGUI: Displays camera images and the estimated robot pose.
- Frequency: Ensures a stable control loop frequency.
- OpenCV: Used for camera modeling and pose estimation.
- pyapriltags: Detects AprilTags in the image.
- YAML: Loads the known global positions of the tags.

## Navigation Parameters

Several parameters define the robot’s motion behavior:

- Linear and angular control gains for smooth movement.
- Maximum linear and angular velocities for safety.
- A target distance at which the robot stops in front of the beacon.

These parameters allow tuning the responsiveness and speed of the robot.

## Exploration Strategy

When no visual marker is detected, the robot performs a spiral-like exploration movement.  
This behavior increases the probability of detecting a beacon while ensuring continuous motion.

## AprilTag Detection

The system is configured to detect tags from the `tag36h11` family, which is the one specified in the exercise.  
Each detection provides image-space corner coordinates and an identifier.

## World Map of Tags

The global positions and orientations of all tags are loaded from a configuration file.  
This information allows the robot to convert relative measurements into absolute world coordinates.

## Camera Model and Pose Estimation

A pinhole camera model is used to relate image points to 3D space.  
Using the detected tag corners, the system estimates the pose of each tag relative to the camera with a PnP algorithm.

## Coordinate Transformations

The relative pose between the camera and the tag is inverted and projected onto the robot’s 2D plane.  
Using the known tag pose, the robot’s global position and orientation are computed.

## Visual Confidence and Tag Weighting

When multiple tags are detected, each contributes to the final pose estimate with a weight based on:

- Distance to the tag.
- Quality of the observed orientation.

This weighted fusion improves robustness and reduces the impact of poor detections.

## Temporal Filtering

A temporal smoothing filter is applied to the visual pose estimates.  
This reduces noise and prevents sudden jumps in position or orientation.

## Odometry Fusion

Odometry is used to propagate the robot’s pose when no visual information is available.  
When tags are detected, the odometric estimate is corrected according to the confidence of the visual measurement, reducing long-term drift.

## Reactive Navigation Logic

### When a Tag Is Visible

- The robot aligns itself with the beacon.
- Linear velocity increases when the robot is well aligned.
- The robot stops at a predefined distance from the tag.

### When No Tag Is Visible

- The robot switches to exploration mode.
- A constant spiral motion is applied until a tag is detected.

## Visualization and Control Loop

- The estimated robot pose is displayed in real time.
- Detected tags are highlighted in the camera image.
- The control loop runs at a fixed frequency to ensure stable behavior.

## Final Summary

This solution satisfies all the requirements of the Marker Visual Localization exercise by combining:

- Vision-based localization using AprilTags.
- Odometry-based pose estimation for robustness.
- Confidence-aware sensor fusion.
- Autonomous exploration and perception-driven navigation.

The result is a stable and reliable localization and navigation system aligned with the exercise specifications.

## Demonstration Video

A demonstration video of the robot executing this solution can be found at:

- [*Final behaviour*](https://youtu.be/fba9qxtgA2I)
