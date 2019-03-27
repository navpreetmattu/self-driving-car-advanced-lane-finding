## Advanced Lane Finding Project

## Writeup Document

### This is a writeup document to describe the working of pipeline created for advance lane detection with with opencv.

---

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

[image1]: ./test_examples/distorted.jpg "Distorted"
[image2]: ./test_examples/undistort_output.jpg "Undistorted"
[image3]: ./test_examples/test1.jpg "Normal Road"
[image4]: ./test_examples/test1_undistort.jpg "Road Transformed"
[image5]: ./test_examples/straight_lines2.jpg "Colored Image"
[image6]: ./test_examples/thresholding.jpg "Binary Image"
[image7]: ./test_examples/warped.jpg "Warp Example"
[image8]: ./test_examples/polynomial_fit.jpg "Fit Visual"
[image9]: ./test_examples/example_output.jpg "Output"
[video1]: ./project_video_result.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cell no. 5 and 6 of the IPython notebook located in `./P2.ipynb`.  

I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the real world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]
#### <center>Distorted</center>

![alt text][image2]
#### <center>Undistorted</center>

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image3]
#### <center>Distorted</center>

![alt text][image4]
#### <center>Undistorted</center>

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding methods have been created at cells 7 through 9 in `P2.ipynb`).  Here's an example of my output for this step.

![alt text][image5]
#### <center>Colored Image</center>

![alt text][image6]
#### <center>Binary Image after Thresholding</center>

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp_image()`, which appears in code cell no. 10 of the IPython notebook `P2.ipynb`.  The `warp_image()` function takes as inputs an image (`image`). I am initializing source and destination points in the same function. I chose the hardcode the source and destination points in the following manner:

```python
src_points = np.float32(
    [[730, 470], [1130, 670], 
    [270, 670], [560, 470]])
dst_points = np.float32(
    [[image_size[0]-150, 0], 
    [image_size[0]-150, image_size[1]], 
    [150, image_size[1]], 
    [150, 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 730, 470      | 1130, 0       | 
| 1130, 690     | 1130, 720     |
| 200, 690      | 150, 720      |
| 550, 470      | 150, 0        |

I verified that my perspective transform was working as expected by drawing the `src_points` and `dst_points` points onto a test image and its warped counterpart to by checking that the lines appear parallel in the warped image.

![alt text][image7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for identifying lane line pixels and fit positions is from code cells 11 to 14 of the IPython notebook located in `./P2.ipynb`.

I, first, calculated sum of the bottom half values of binary image column-wise to get the base points of the left and right lane. The base points are where the highest peaks are present at both side from the center of image. 
Then, I used sliding windows to capture the full lane lines. For that, I stored all the non zero pixels present in the image. Then, I captured all the points comes under the window region. Iterating and storing the points using the same steps till it reaches the top of the image. Using all the captured points, I fitted a 2nd order polynomial to get full left and right lane line curves.

![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for calculating radius of curvature and position of the vehicle is at code cell 15 of the IPython notebook located in `./P2.ipynb`.

The formula used to radius of curvature is (1+ (2Ay + B)<sup>2</sup>)<sup>1.5</sup>/|2A|. For calculating the position of the vehicle w.r.t. center, (I first calculated the midpoint using left and right lane line value at the bottom of the image i.e. near to the vehicle OR divided the image width in half). I calculated the position of right and left lane line near the vehicle. Subtracted the lanes position from the midpoint, If the right lane distance is more than the left lane distance then vehicle is on the left side of the road otherwise it is on the right side of the road. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cells 16 (in function `draw_mapping()`), 17 (the pipeline function `pipeline()`) and 18 at code cell of the IPython notebook located in `./P2.ipynb`. Here is an example of my result on a test image:

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_result.mp4)

---

### Discussion

The approach used is very straight forward:
* First, I removed any distortions present in the image or video.
* Applied color and gradient threshold to extract the lane lines(at bottom half of the image) from the image. 
* Applied perspective transformation to get bird's eye view of lane lines.
* Calculated sum of the bottom half values of binary image to get the base points of the left and right lane. The base points will be at the highest values at left and right from the centre of image. Then, fitted a 2nd order polynomial to get full lane lines.
* Calculated the radius of curvature and position of the vehicle.
* At the last, fused the end results of lane detection with the original image.

**Shortcomings**:

The pipeline might fail when the curves are very deep i.e. on roads with very sharp curves.

**Future Improvements**

We may able to improve the project if we can dymanically calculate the source points for perspective transform and able to find lane lines in resulting warped image.