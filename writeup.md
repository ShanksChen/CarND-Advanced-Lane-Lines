# Advanced Lane Finding Project

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

[image1]: ./writeup_pictures/camera-calibration.png "Camera Calibration"
[image2]: ./writeup_pictures/undistorted.png "Undistorted"
[image3]: ./writeup_pictures/unwarped.png "Unwarped"
[image4]: ./writeup_pictures/color-space-list.png "Color Space List"
[image5]: ./writeup_pictures/sobel-magnitude.png "Sobel Magnitude"
[image6]: ./writeup_pictures/sobel-gradient-magnitude.png "Sobel Gradient Magnitude"
[image7]: ./writeup_pictures/sobel-direction-threshold.png "Sobel Direction Threshold"
[image8]: ./writeup_pictures/HLS-s.png "HLS S-Channel"
[image9]: ./writeup_pictures/RGB-r.png "RGB R-Channel"
[image10]: ./writeup_pictures/combine-all1.png "Combine All 1"
[image11]: ./writeup_pictures/combine-all2.png "Combine All 2"
[image12]: ./writeup_pictures/test-list1.png "Test List 1"
[image13]: ./writeup_pictures/test-list2.png "Test List 2"
[image14]: ./writeup_pictures/slid-window.png "Slid Window"
[image15]: ./writeup_pictures/fit-ploy-pre.png "Fit Polynomail Prevous"
[image16]: ./writeup_pictures/draw-lane.png "Draw Lane"
[image17]: ./writeup_pictures/draw-lane-with-description.png "Draw Lane With Description"
[video1]: ./project_video.mp4 "Project Video"
[video2]: ./project_video_output.mp4 "Project Video Output"
[video3]: ./challenge_video.mp4 "Challenge Video"
[video4]: ./challenge_video_output.mp4 "Challenge Video Output"
[video5]: ./harder_challenge_video.mp4 "Harder Challenge Video"
[video6]: ./harder_challenge_video_output.mp4 "Harder Challenge Video Output"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

ALL code for this project is contained in the IPython notebook "Advanced-Lane_lines.ipynb".

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

This function code which appears in the 2nd code cell. 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

And I saved `mtx` and `dist` in the `wide_dist_pickle.p`. Then I can use this from the file.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I use a function called `distortion_image()`, which appears in the 3rd code cell. I get the distortion corrected image like this:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

First, I need the warped image which is used to get binary image. The code for my perspective transform includes a function called `unwarp()`, which appears in lines 1 through 8 in the 5th code cell of the IPython notebook.  The `unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
offset = 150
src = np.float32([[180, 700], [1100, 700], [574, 460], [706, 460]])
dst = np.float32([[offset, img_size[1]], [img_size[0] - offset, img_size[1]], [offset, 0], [img_size[0] - offset, 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 180, 700      | 150, 720      | 
| 1100, 700     | 1130, 720     |
| 574, 460      | 150, 0        |
| 706, 460      | 1130, 0       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. The warped image like this:

![alt text][image3]

Then I show the four kind of color space in the 7th code cell. There were RGB, HSV, LAB and HLS. The list of these color space shows below:

![alt text][image4]

After comparison I decided to use HLS S-Channel Threshol(function in the 16th code cell) and RGB R-Channel Threshold(function in the 18th code cell) to implement the color threshold.

![alt text][image8]
![alt text][image9]

Only color space threshold is not enough, I also used sobel magnitude(function in the 8th code cell), sobel gradient magnitude(function in the 10th code cell)and sobel direction threshold(function in the 13th code cell). Combine these to create a binary image. 

![alt text][image5]
![alt text][image6]
![alt text][image7]

I used a combination of above five method to generate a binary image(function in the 21th code cell). Here's an example of my output for this step.

![alt text][image10]

I defined a function called `find_lane()`, which in the 22th code cell, implement what I have done before. The input is the image and the output is the binary image. Here's an example output of this function:

![alt text][image11]

In the 24th code cell, I used the `find_lane()` to print all test image in the test_images folder. The result shows below:

![alt text][image12]
![alt text][image13]

#### 3. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used sliding window to identified lane-line pixels and fit a 2nd order polynomial(function in the 25th code cell). The function calls `slid_window()`. First of this function computed the histogram of the bottom half of the image and found the `x` base position of left and right line. Then identifies ten windows from which to identify lane pixels. After that, I got the `left_fit`, `right_fit`, `left_lane_inds`, `right_lane_inds` and `out_img` result.

I used the first four parameters to fit lane line with 2nd order polynomial.

The `out_img` is used for visualize the result. The result shows below:

![alt text][image14]

In the 27th code cell, I defined `fit_poly_prev()` function. This function calculate the new `left_fit`, `right_fit`, `left_lane_inds` and `right_lane_inds` by previous `left_fit` and `right_fit`. If the difference in fit coefficients between last and new fits is reasonable or last fits existed, even the pixels between new left fit and right fit were in the range(960 +/- 100), I used this function to calculate the fits. It's more effective because the function dp not use the sliding window. The example shows below:

![alt text][image15]

#### 4. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in 29th code cell. The camera image has 720 relevant pixels in the y-dimension, and 920 relevant pixels in the x-dimension. Therefore, to convert from pixels to real-world meter measurements, I use:

```python
# Define conversions in x and y from pixels space to meters
ym_per_pix = 30 / 720  # meters per pixel in y dimension
xm_per_pix = 3.7 / 920  # meters per pixel in x dimension
```

In order to calculate the position of the vehicle with respect to center, I got the `left_curverad` and `right_curverad` values within the newfrom the `measure_curvature_real()` function. The car position is the difference between these intercept points and the image midpoint (assuming that the camera is mounted at the center of the vehicle). First I calculated the `car_position` value which equals to half of the maximum x value(x equals to the width of the image). Then I calculated the value of x corresponding to the maximum value of the y-axis for left and right lane line. When I got `left_fit_x` and `right_fit_x`, the `lane_center_position` value can be calculated. At last, `center_dist`(means the position of the vehicle with respect to center) was equal to `car_position` minus `lane_center_position`. But the unit of result is pixels. Multiply it by the `xm_per_pix` to get the value in meters. 

```python
# Defint Car position
car_position = binary_img.shape[1] / 2
left_fit_x = left_fit[0] * binary_img.shape[0] ** 2 + left_fit[1] * binary_img.shape[0] + left_fit[2]
right_fit_x = right_fit[0] * binary_img.shape[0] ** 2 + right_fit[1] * binary_img.shape[0] + right_fit[2]
lane_center_position = (right_fit_x + left_fit_x) / 2
center_dist = (car_position - lane_center_position) * xm_per_pix

```

#### 5. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In the 31th code cell, I defined `draw_lane()` function to visual the polygon is generated based on plots of the left and right fits, warped back to the perspective of the original image using the inverse perspective matrix Minv and overlaid onto the original image. The image below is an example of the results:

![alt text][image16]

In the 33th code cell, I added the radius of curvature of the lane and the position of the vehicle with respect to center by the `description()` function.  The image below is an example of the results:

![alt text][image17]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my project video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In the challenge project, it can be found that the algorithm for identifying lane lines is not robust enough. It is more evident in harder challenge. I used the methods highlighted in the course in my algorithm and the two kinds of color space. Even then, only the lane recognition in the project video can be completed. When the lane class is not used, problems occur even in the lane recognition of the project video. These problems occur mainly when the intensity of the lightness is particularly strong or when the shadow of the roadside trees suddenly appears. 
When using the HLS color space, since the S-Channel is not affected by the lightness change. Therefore, the S-Channel was chosen, but the results show that using only the S-Channel does not completely solve this problem.
Perhaps through other color spaces, or other methods can better solve the problem when the lightness is particularly large.
This issue requires more in-depth exploration.

The use of line class is still at an early stage and there is still much room for improvement.

Below are the result of three videos:

[Project video](./project_video_output.mp4)


[Challenge video](./challenge_video_output.mp4)


[Harder Challenge video](./harder_challenge_video_output.mp4)
