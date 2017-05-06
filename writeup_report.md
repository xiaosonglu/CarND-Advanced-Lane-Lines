**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/image1.png "Undistorted"
[image2]: ./output_images/image2.png "Road Transformed"
[image3]: ./output_images/image3.png "Binary Example"
[image4]: ./output_images/image4.png "Warp Example"
[image5]: ./output_images/image5.png "Fit Visual"
[image6]: ./output_images/image6.png "Output"
[video1]: ./out.mp4 "Video"

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cell #1 and #2 of the IPython notebook located in "./P4.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Given the object points and image points based on the above steps, code cell #3 generates an example of a distortion-corrected image of test2.jpg from the test_images folder.
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. Code cell #4 generates an example of the binary image based on test2.jpg for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the 5th code cell of the IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src_img`) and destination (`dst_img`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 94],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 94]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 454      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 454      | 960, 0        |

Below is the warped image by applying a perspective transform to the previous thresholded binary image and using the above source and destination points. It shows that the curved lines are (more or less) parallel in the transformed image.
![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this section is under the 6th code cell of the IPython notebook. I first find peaks in a histogram along all the columns in the lower half of the image. The two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. I use that as a starting point for where to search for the lines. From that point, I use a sliding window, to identify the nonzero pixels in x and y within the window. I use 9 windows and recenter the next window on their mean position from the current window. After I find all the left and right pixel positions for all the left and right sliding windows, I fit a second order polynomial to them to get the left and right line pixel positions. The image below shows the lines with the green shaded area which shows where we searched for the lines.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature of the lane is calculated based on the equation described in [Radius of Curvature](http://www.intmath.com/applications-differentiation/8-radius-curvature.php) and implemented under the 7th code cell of the IPython notebook. The position of the vehicle with respect to center is calculated based on the offset of the lane center from the center of the image (converted from pixels to meters). The lane center is the midpoint at the bottom of the image between the two lines. The offset is implemented inside the 8th code cell of the IPython notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 8th code cell of the IPython notebook.  Here is an example of my result on a test image:

![alt text][image6]

#### 7. Sanity Check with Look-Ahead Filter
I did the sanity check to check if the top and bottom pixel positions of left and right lines are separated by approximately the right distance horizontally. This is implemented in the 10th code cell of the IPython notebook. The Look-Ahead Filter is implemented as follows:

If the distance between the two top lines (right and left) is not close to the distance between the two bottom lines (right and left) and the difference is greater than 150 pixels, the blind search will be run next frame. If not, the blind search will be skipped and only a margin around the previous line position will be searched.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Smoothing might be applied to smooth over the last n frames of video to obtain a cleaner result. Also, the shadows under the trees might cause the failure of the lines detection. In this case, a new method might be needed to re-establish the measurement.
