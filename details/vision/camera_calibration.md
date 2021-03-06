---
title: Camera Perspective Warp
layout: page
image: distort.jpg
image_alt: Abstract image of distorted chessboard.
next_url: /details/table.html
next_topic: how the table is put together
---

# Introduction
---

The Raspberry Turk uses a [Raspberry Pi camera module](https://www.raspberrypi.org/products/camera-module/) and [computer vision algorithms](/details/vision.html) to determine which pieces are in which squares. In order to see the board properly, it needs to be calibrated first.

<center>
{% include image name="capture.jpg" alt="A raw image of the chessboard on table, captured from the camera." width="40%" %}
{% include arrow %}
{% include image name="chessboardframe.jpg" alt="Chessboard after warping the perspective of the raw image captured by the camera." width="30%" %}
</center>
{% include caption txt="A) Raw image taken from camera. B) Final image after perspective transform." %}

# Warp Perspective
---

From [`chess_camera.py`](https://github.com/joeymeyer/raspberryturk/blob/master/raspberryturk/embedded/vision/chess_camera.py#L13)
```python
def current_chessboard_frame(self):
    _, frame = self._capture.read()
    h, w = frame.shape[:2]
    M = get_chessboard_perspective_transform()
    bgr_img = cv2.warpPerspective(frame, M, (BOARD_SIZE,BOARD_SIZE))
    img = cv2.cvtColor(bgr_img, cv2.COLOR_BGR2RGB)
    flipped_img = cv2.flip(img, -1)
    return ChessboardFrame(flipped_img)
```

In order to go from A to B, [`cv2.warpPerspective`](http://docs.opencv.org/3.0-last-rst/modules/imgproc/doc/geometric_transformations.html#cv2.warpPerspective) from OpenCV is used.

# Finding the Transformation Matrix
---

There is a small script used to find the correct transformation matrix. The full source can be found [here](https://github.com/joeymeyer/raspberryturk/blob/master/raspberryturk/embedded/vision/chessboard_perspective_transform.py).

Essentially it does the following:

1. Find the inner chessboard corners using [`cv2.findChessboardCorners`](http://docs.opencv.org/3.0-last-rst/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html#cv2.findChessboardCorners)
2. Ensure it found the chessboard and the camera is centered
3. Extrapolate and find the outer four corners of the board
4. Map the actual four corners of the board to the corners of the new image and create the perspective matrix using [`cv2.getPerspectiveTransform`](http://docs.opencv.org/3.0-last-rst/modules/imgproc/doc/geometric_transformations.html#cv2.getPerspectiveTransform)
5. Save the transformation matrix for later use

If the camera moves or the robot is deconstructed and reconstructed the perspective transformation matrix can be easily recalculated.

```bash
$ python -m raspberryturk.embedded.vision.chessboard_perspective_transform
2017-02-11 16:05:41,371 - __main__ - INFO - Begin camera position recalibration...
2017-02-11 16:05:41,371 - __main__ - INFO - Camera position recalibration successful.
```
