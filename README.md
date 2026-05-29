# Hand Gesture Mars-Rover Control

A ROS node that uses a webcam and MediaPipe to detect hand gestures in real time and translate them into rover movement commands published on `/cmd_vel`.

## Dependencies

- Python 3
- [ROS](http://wiki.ros.org/ROS/Installation) (Noetic or compatible)
- OpenCV (`cv2`)
- MediaPipe (`mediapipe`)
- `std_msgs`, `geometry_msgs` ROS packages

Install Python dependencies:

```bash
pip install opencv-python mediapipe
```

## How It Works

The node captures webcam frames, detects hand landmarks via MediaPipe, and measures distances between fingertip landmarks to classify gestures. Each gesture maps to a `Twist` command sent to the robot.

### Gesture Map

| Gesture | Description | Robot Action |
|---|---|---|
| Peace sign ✌️ | Index & middle fingers spread | Move forward |
| Closed fist ✊ | All fingers together | Reverse |
| Glued/flat hand 🤚 | Fingers close together | Stop |
| Thumb gesture (left) 👍 | Ring & pinky spread, others close | Turn left |
| Open hand 🖐️ | All fingers spread | Turn right |

Gesture classification is based on Euclidean distances between these MediaPipe landmarks:

- `4` — Thumb tip
- `8` — Index finger tip
- `12` — Middle finger tip
- `16` — Ring finger tip
- `20` — Pinky tip

## ROS Topics

| Topic | Type | Description |
|---|---|---|
| `/cmd_vel` | `geometry_msgs/Twist` | Velocity commands for the rover |
| `/hand_detected` | `std_msgs/String` | Published when a hand is detected, includes fingertip distance |
| `/gesture_info` | `std_msgs/String` | Human-readable label of the current gesture |

## Usage

1. Make sure your ROS master is running:

```bash
roscore
```

2. Run the node:

```bash
python hand_gesture_control.py
```

3. A window labeled `Handtracker` will open showing the webcam feed with hand landmarks overlaid. Press `q` to quit.

## Notes

- The node uses the first available webcam (`/dev/video0`). Change the index in `cv2.VideoCapture(0)` to use a different camera.
- Distance thresholds (`0.1`, `0.05`) are normalized to the image frame and may need tuning depending on your camera and lighting conditions.
- Linear velocity is set to `±0.2 m/s` and angular velocity to `±0.2 rad/s`. Adjust these in the `Twist` assignments to match your robot's specs.
