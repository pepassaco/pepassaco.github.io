---
layout:  post
title: "FUTracker: A foosball ball tracker with OpenCV"
author: "Pablo Álvarez"
categories: imaging
tags: [
  imaging,
  foosball
]
image: /assets/img/posts/2018-10-13-FUTracker/2.png

---

The new academic course already began - and, with it, the habit of playing foosball for hours everyday.

One day, I wondered whether it would be possible to *upgrade* our foosball: such a traditional game has remained exactly the same for years, so it would be cool to add it some new features. I first thought about adding some sensors to it, but ended up deciding that using some image recognition software would be the most versatile option. Moreover, I had never worked with any video processing software, so I saw in this project a good opportunity to learn something new.

## The idea

As a first objective, I tried to be able to locate the ball and draw its movement. This is a good starting point, since, once this is working, I will be able to detect the ball's speed and other statistics such as possession, passes, etc.

The script I coded (available in [my GitHub](https://github.com/pepassaco/FUTracker) is dividing into three main parts:

- **Color mask**: we take advantage of the fact that the ball is always the same color (and different from the player's one) for applying this first filter to the image.

- **Morphological transformations**: once we have a binary image, we can use some morphological transformations in order to help identifying the ball. We start with a noisy image and we end up with one where it is possible to identify a night white rounded spot.

- **Detection of shapes, centroid and plot of the data**: we detect the biggest shape, calculate it's centroid and determine that this is the position of the ball. Finally, we plot these results in the original video.

![Fut-2](/assets/img/posts/2018-10-13-FUTracker/1.jpg)

## The code

The full version of the code can be found in my [GitHub](https://github.com/pepassaco/FUTracker). In order to improve the overall performance of the script and show the FPS on the screen, I followed the tutorials from this [website](https://www.pyimagesearch.com/2015/12/28/increasing-raspberry-pi-fps-with-python-and-opencv/). 

First, we must include the necessary libraries:

```python=
from collections import deque
from imutils.video import VideoStream
from imutils.video import FileVideoStream
from imutils.video import FPS
import numpy as np
import argparse
import cv2
import imutils
import time
```
Then, we iniciate the argument parser which allows us to add arguments with options when calling the program from the console.

``` python=
ap = argparse.ArgumentParser()
ap.add_argument("-v", "--video",
	help="path to the (optional) video file")
ap.add_argument("-b", "--buffer", type=int, default=64,
	help="max buffer size")
args = vars(ap.parse_args())
```
Afterwards, we choose the bounds for our color mask, the kernels for the morphological operations (see OpenCV's [documentation](https://www.tutorialspoint.com/opencv/opencv_morphological_operations.htm))and a deque where we will save the last positions of the ball. We also manage the different possible arguments and strar the FPS meter. 

``` python=
colorLower = (0, 140, 170)
colorUpper = (80, 255, 255)

kernel1 = np.ones((1,5),np.uint8)
kernel2 = np.ones((5,5),np.uint8)
kernel3 = np.ones((3,3),np.uint8)


pts = deque(maxlen=args["buffer"])


if not args.get("video", False):
	vs = VideoStream(src=1).start()

else:
	vs = cv2.VideoCapture(args["video"])

    
time.sleep(1.0)

fps = FPS().start()
```
This point is where our loop starts: evey iteration will be processed on a different frame of the video.

The first step here is then grabbing the corresponding frame.

```python=
while True:

	frame = vs.read()
	frame = frame[1] if args.get("video", False) else frame

	if frame is None:
		break
```
Then, we will resize it to reduce the processing needed, blur it to remove noise and apply the color mask.

```python=
frame = imutils.resize(frame, width=500)
	
blurred = cv2.GaussianBlur(frame, (3,3), 0)
hsv = cv2.cvtColor(blurred, cv2.COLOR_BGR2HSV)

mask3 = cv2.inRange(hsv, colorLower, colorUpper)
```
Before trying to detect the ball, we may use some morphological transformations in order to facilitate this process.
```python=
mask2 = cv2.morphologyEx(mask3, cv2.MORPH_OPEN, kernel3)
mask1 = cv2.dilate(mask2, kernel2, iterations=1)
mask = cv2.morphologyEx(mask1, cv2.MORPH_OPEN, kernel3)
mask = cv2.morphologyEx(mask1, cv2.MORPH_CLOSE, kernel3)
```
Once we have a clear binary image without a lot of noise, we can proceed to find the ball. For that, we first find all the possible contours. If we can detect more than one, we choose the one whose dimmensions can match the ones of the ball. Finally, we calculate its centroid, which will become our data about the position of the ball, and draw a rectangle around the ball.

```python=
cnts = cv2.findContours(mask.copy(), cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
cnts = imutils.grab_contours(cnts)
center = None

if len(cnts) > 0:

    c = max(cnts, key=cv2.contourArea)

    x,y,w,h = cv2.boundingRect(c) 

    M = cv2.moments(c)
    center = (int(M["m10"] / M["m00"]), int(M["m01"] / M["m00"]))

        
    if (w > 9 and h > 9 and 33  > w and 33  > h):
        cv2.rectangle(frame,(x,y),(x+w,y+h),(0,255,0),2)
        cv2.circle(frame, center, 5, (0, 0, 255), -1)
```

For drawing the trail of the trajectory, we use the pts deque we defined before. For that, we first append our new sample to the deque, and then choose the thickness of each point based on their position in the list. They get plotted by using a *cv2.line* sentence. For more information on this, refer to this [site](https://www.pyimagesearch.com/2015/09/14/ball-tracking-with-opencv/).
```python=
pts.appendleft(center)

for i in range(1, len(pts)):

    if pts[i - 1] is None or pts[i] is None:
        continue
        
    thickness = int(np.sqrt(args["buffer"] / float(i + 1)) * 2.5)
    cv2.line(frame, pts[i - 1], pts[i], (0, 0, 255), thickness)
 
```
Once we have our image ready to be shown, we update the FPS meter and plot the desired frames

```python=
fps.update()
fps.stop()

info = [
    ("FPS", "{:.2f}".format(fps.fps())),
]


cv2.imshow("Original", frame)
#cv2.imshow("Blur", blurred)
#cv2.imshow("Filtrado", mask3)
#cv2.imshow("Erosionado", mask2)
#cv2.imshow("Dilatado", mask1)
cv2.imshow("Rellenado", mask)
````

Finally, we check for a key pess of the key *q*, which would stop the program.

```python=
key = cv2.waitKey(1) & 0xFF
if key == ord("q"):
		break
```
Befor ending the script, we must handle the exits of the main loop. Since this will only happen when the either the video has finished or the user wants to quit the program, we just have to close the video stream and close the open windows.
```python=
if not args.get("video", False):
    vs.stop()
else:
    vs.release()

cv2.destroyAllWindows()
```
## Results

It works!

![Fut-1](/assets/img/posts/2018-10-13-FUTracker/3.jpg)

Two of the biggest problems of trying to follow a foosball ball are its extreme speed (it can reach tens of Km/h) and the fact that, when one player has the possession, either the foosball player or the bar which joins them together can cover the ball. While this program suffers a bit with high speed shoots due to the low number of maximum FPSs achievable, it responds incredibly well to situations where the ball is partially or even almost completely cover.

If you enjoyed this project, I encourage you to try the examples that you can find on my [GitHub](https://github.com/pepassaco/FUTracker) or try it with your personal foosball.
