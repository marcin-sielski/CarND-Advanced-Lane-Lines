## Advanced Lane Finding Project

### Writeup.

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

[image11]: ./camera_cal/calibration4.jpg "Original Image"
[image12]: ./output_images/corners_undistort_calibration4.jpg "Undistorted Image"

[image21]: ./test_images/test1.jpg "Road"
[image22]: ./output_images/undistort_test1.jpg "Road Transformed"

[image3]: ./output_images/transform_combine_undistort_test1.jpg "Binary Example"
[image41]: ./test_images/straight_lines2.jpg "Straight Lines"
[image42]: ./output_images/transform_unwarp_undistort_straight_lines2.jpg "Unwarped Straight Lines"

[image5]: ./output_images/detect_transform_unwarp_undistort_test2.jpg "Detected Lane-Lines"

[image6]: ./curvature_radius_formula.gif
[image7]: ./output_videos/00000001.jpg "Output"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](./writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third code cell of the IPython notebook located in [advanced_lane_finding.ipynb](./advanced_lane_finding.ipynb).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the `corners` in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

| Original Image       | Undistored Image     |
|:--------------------:|:--------------------:|
| ![alt text][image11] | ![alt text][image12] |

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Here is sample test image and it distortion-corrected counterpart:

| Road                 | Road Transformed     |
|:--------------------:|:--------------------:|
| ![alt text][image21] | ![alt text][image22] |

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used following combination of color and gradient thresholds to generate a binary image.

| Transformation                   | Parameters                 |
|---------------------------------:|:---------------------------|
| Grayscale Threshold              | thresh=(205, 255)          |
| Absolute X Sobel Gradient        | kernel=7, thresh=(35, 105) |
| HLS Color Threshold - Hue        | thresh=(20, 45)            |
| HLS Color Threshold - Saturation | thresh=(180, 245)          |

Thresholding steps are defined in `Transform.process()` function in fifth code cell of IPython notebook [advanced_lane_finding.ipynb](./advanced_lane_finding.ipynb).

Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `Transform.unwarp()`, which appears in fifth code cell of IPython notebook [advanced_lane_finding.ipynb](./advanced_lane_finding.ipynb).  The `Transform.unwarp()` function takes as input an image (`img`) and uses following hardcoded source (`src`) and destination (`dst`) points:

```python
    src = np.float32([(230, 690), (545, 480), (735, 480), (1050, 690)])
    dst = np.float32([(230, 720), (230,   0), (1050,  0), (1050, 720)])
```

I verified that my perspective transform was working as expected by visually inspecting test image and its unwarped counterpart and verifying that the lines appear parallel in the unwarped image (see below).

![alt text][image41]
![alt text][image42]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In other to identify lane-line pixels I implemented `Line.find_pixels()` function located in seventh code cell of IPython notebook [advanced_lane_finding.ipynb](./advanced_lane_finding.ipynb). The function calculates histogram of white pixels to identify start position of the lines and then uses sliding window algorithm to identify all the pixels that belongs to the lines. I approximate my lane lines with a 2nd order polynomial as follows:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I implemented `Line.get_curvature_radius()` function located in seventh code cell of IPython notebook [advanced_lane_finding.ipynb](./advanced_lane_finding.ipynb) to calculate radius of curvature. The implementation assumes that unwarped images contains 20m long lines (20m/720px). Distance between the lines according to US regulations is about 3.7m and it is easy to obtain it in pixels from the image. I used following formula:

![alt text][image6]

to calculate radius of curvature.

Position of the vehicle with respect to lane center is calculated in `LaneDetector.process()` function located in eighth code cell of IPython notebook [advanced_lane_finding.ipynb](./advanced_lane_finding.ipynb). This function uses the same assumption as described above.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `LineDetector.process()` function located in eighth code cell of IPython notebook [advanced_lane_finding.ipynb](./advanced_lane_finding.ipynb). Here is an example of my result on a test image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](output_videos/project_video.mp4).

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

During implementation of this project I faced following issues that remain unsolved:

* car bouncing - perspective transformations should be depedant on slope of the car,
* traces of patching the road - road imperfections are treated by the algorithm as a lane lines,
* poor quality frames - the algorithm should restart when no lane lines are detected,
* various lighting conditions - the algorithm should use measurements from light sensor to adjust transformations thresholds parameters.
