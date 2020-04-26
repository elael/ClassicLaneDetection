## Writeup Template


### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[images_detection]: ./images_detection.png "Images with Lane Highlighted"
[lane_windowed]: ./lane_windowed.png "Lane Windowed"
[bird_view_lanes]: ./bird_view_lanes.png "Bird View"
[thresholding_image_merged]: ./thresholding_image_merged.png "Thresholding Image Merged"
[thresholding_image]: ./thresholding_image.png "Thresholding Image"
[undistorted_image]: ./undistorted_image.png "Undistorted Image"
[undistorted_chessboard]: ./undistorted_chessboard.png "Chessboard undistorted"
[filtered_project_video]: ./output_videos/filtered_project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third code cell of the IPython notebook located in "./LineFindingPipeline.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each <u>successful</u> chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][undistorted_chessboard]

### Pipeline (single images)


#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I apply OpenCV's `cv2.undistort` function to distort correct one of the test images like this one:

![alt text][undistorted_image]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color on HSV and gradient thresholds to generate a binary image, with a simple erode/dilatate cicle with `cv2.morphologyEx(combined_threshold, cv2.MORPH_OPEN, (ksize,ksize))` to reduce noise (thresholding steps from the 5-7th code cell).  Here's an example of my output for this step.

![alt text][thresholding_image]

![alt text][thresholding_image_merged]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `bird_eye()`, which appears in the 9th code cell of the IPython notebook).  The `bird_eye()` function takes as inputs an image (`image`), and use the globally avaible variable `M` as the Matrix transformation. `M` is computed as `M = cv2.getPerspectiveTransform(src, dst)`, where `src` and `dst` are the source and destination points, respectively.  I chose the hardcode the source and destination points based on pixel measurements I did:

```python
src = np.float32((
    (207, 720),
    (591, 453),
    (692, 453),
    (1104, 720)
))
dst = np.float32((
    (320, 720),
    (320, 0),
    (960, 0),
    (960, 720)
))
```


I verified that my perspective transform was working as expected by verifing that the lane lines appear near parallel in the bird-view image of the "straight lines" examples, as below:

![alt text][bird_view_lanes]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In the 11th cell, I create a histogram of the filtered pixel on the bottom half of the image. I selected from this histogram the peaks from the left- and right-half to use as a starting `x` value to search for he lines. Using this `x` value and a marging for each lane, I created a sequence of "windows" to detect the whole line extend. 

Every pixel inside the "windows" of each line were used to fit a 2nd order polynomial, with a inverse square root weighting:

![alt text][lane_windowed]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the curvature at the bottom of the image on cell 13, using a radius of curvature (roc) equation especific to 2nd order polynomials. Conversion from pixels to meters was based on dashed line standard size and expected lane width.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 16th code cel in the function `image_processing_pipeline`.  Here is an example of my result on the test images:

![alt text][images_detection]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [filtered_project_video]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The main issue is robustness. It does not perfom well on the challenge and it is even funny to see how bad it performs on the hard challenge. There are many extra variables and details to consider to make it work better. Some important points to improve on:

* It does not detect lane pixel with sufficient robustness, some histogram balancing (like CLAHE) might help. Also exploring a combination of different color spaces tresholds, or even a data-driven non-linear color filtering. Same consideration for the gradient filtering.
* Improve on temporal coherence considerations. Using a near optimal Kalman-like filter could improve predictions when no lines can be detected, also using the detected lines to help not only the pixel selection for a given line, but also the pixel filtering (maybe using color information) could give more consistent results.
* Context aware "bird view" (perspective projection) would be important in the case of going up/down a hill.
* Some edge cases considerations, e.g. when the curve is to steep and it end on the side of the image.
* Many, many more...
