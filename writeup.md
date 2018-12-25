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
[video1]: ./project_video_output.mp4 "Project Video Output"
[video1]: ./challenge_video.mp4 "Challenge Video"
[video1]: ./challenge_video_output.mp4 "Challenge Video Output"
[video1]: ./harder_challenge_video.mp4 "Harder Challenge Video"
[video1]: ./harder_challenge_video_output.mp4 "Harder Challenge Video Output"

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

I used sliding window to identified lane-line pixels(function in the )

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 4. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 5. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
