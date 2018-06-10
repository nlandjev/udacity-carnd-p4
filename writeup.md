## Writeup

---

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

[undistorted-image-calibration]: ./output_images/undistorted-image.png "Undistorted Calibration Image"
[undistorted-image-road]: ./output_images/undistorted-image-road.png "Undistorted Road Image"
[binary-threshold]: ./output_images/binary-threshold.png "Binary threshold"
[perspective-transform]: ./output_images/perspective-transform.png "Perspective Transform"
[lane-lines1]: ./output_images/lane-lines1.png "Lane Lines 1"
[lane-lines2]: ./output_images/lane-lines2.png "Lane Lines 2"
[lane-overlay]: ./output_images/lane-overlay.png "Lane Overlay"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  
---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

This is the first step in the lane detection pipeline and is included in both notebooks:
 * `./examples/lane-detection-pipeline.ipynb`
 * `./examples/lane-detection-video.ipynb`
 
I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function and I applied this distortion correction to the test image using the `cv2.undistort()` function. Here is a side-by-side comparison of an original calibration image (left) and its undistorted version (right)

![Undistorted calibration image][undistorted-image-calibration]

### Pipeline (single images)

The exploratory pipeline notebook can be found here: [``lane-detection-pipeline.ipynb``](https://github.com/nlandjev/udacity-carnd-p4/blob/master/notebooks/lane-detection-pipeline.ipynb)

#### 1. Provide an example of a distortion-corrected image.

The first step is applying the distortion-correction calculated in the previous step to the test images. The result of this transformation is presented looks like this (the undistorted image is the one on the right: 

![Undistorted road image][undistorted-image-road]

Undistorted versions of all test images can be found in the notebook.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color (red-channel), saturation, luminosity and gradient (X & Y) thresholds to generate a binary threshold image which isolates the lane lines. In addition, I added a region-of-interest mask in the lower half of the image in order to remove any selected pixel which are definitely not part of the lane lines due to their position in the frame. The specific threshold channels and their respective values were arrived at by trial and error.

The complete threshold pipeline is performed in the function `binary_threshold` (and `_binary_threshold` in the `LaneDetection` class in the `lane-detection-video.ipynb` notebook)

Here's an example of a side-by-side comparison of a test image and its corresponding binary threshold image:

![Binary Threshold Image][binary-threshold]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for the perspective transform can be found in the 'Perspective Transform' section of the notebook. For the source and destination points I found the following values to work best:

```python
src_points = np.float32([[600, 450], [680, 450], [1050, 680], [250, 680]])
dst_points = np.float32([[300, 0], [900, 0], [900, 720], [300, 720]])
```

The source points correspond to a trapezoid that contains the lane lines (I chose to remove the part of the image that includes the front of the car); the destination points form a rectangle in the middle of the frame spanning the complete height of the image.

Here's an example of a test image and it's corresponding perspective transform (with a binary threshold applied beforehand):

![Perspective Transform][perspective-transform]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I then used the techniques described in the lectures to fit second-order polynomials to the found pixels in the threshold image. The code can be found in the functions `find-fitted-lines` and `find-fitted-lines-next-frame`. The first one uses the sliding-window technique in the complete image to find the lines and the second one uses information from previously found lines to narrow down the search area. Here is an example of the outputs of both functions: 

![Lane Lines 1][lane-lines1]
![Lane Lines 2][lane-lines2]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The curvature radius calculation is only included in the video generation notebook and is performed in the `_curve_radius` function in the `LaneDetection` class - it uses the technique presented in the lectures to perform the calculation. It is then drawn in the top part of the video frame.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The calculation and the drawing of the unwarped lane overlay is implemented in the `get_lane_overlay` function and the following code cell. The code uses one of the two functions described in the previous section to calculate the lane area in the warped image, applies the inverse of the perspective transform to find the area in the orginal image and then overlays it on top.

An example of an image and the corresponding lane overlay is presented here:

![Lane overlay][lane-overlay]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).


The implementation of the pipeline applied to a video is implemented in the `lane-detection-video.ipynb` noteboook. It is almost identical to the `lane-detection-pipeline.ipynb` with the addition of a lane curvature and vehicle position added to the output frames.

Here's a link to my video result: [``output_video.mp4``](https://github.com/nlandjev/udacity-carnd-p4/blob/master/output_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?


In addition to the occasional inaccuracies of the lane detection algorithm, which can be improved by improving the binary thresholding function, there are several ways to make the pipeline more robust:

1. The region-of-interest mask makes things easier in the default case where the lane lines are straight in front of the camera but it will probably break if this is no longer the case e.g. during a lane switch

2. The pipeline assumes the road is flat. It might lose the lanes if there are sudden changes in flatness (e.g. on top of a hill) and in sharp curves when there is little visibility (e.g. when there's a hedge on the side of the road)
