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

[cimage1]: ./my_output/camera_cal/calibration5.jpg "calibration output"
[cimage2]: ./camera_cal/calibration5.jpg "calibration input"
[in] ./test_images/test3.jpg "input image"
[edges] ./my_output/edges.jpg "edges image"
[perspective_edges] ./my_output/perspective_edges.jpg "perspective edges image"
[per_lane_pixes] ./my_output/per_lane_pixels.jpg "Each lane after perspective transform"
[output] ./my_output/output.jpg "Final output"

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is inside the function get_calibration. The way it works is the following: We scan all images in camera_cal directory, we search for all corners by using findChessboardCorners, save everything up and leverage calibrateCamera to get all necessary data (camera matrix, distortion coefficients). In order to get undistorted image user'll have to call get_undistorted_image.

Example of distorned and undistorted image: 

![alt text][cimage1]
![alt text][cimage2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][in]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color (to force including yellow and white pixels) one channel of HLS to generate gradients. I've only kept pretty big gradients and filtered out the ones whose angle was outside of a range (pi*0.2, pi*0.3) - since that is mostly where the road should be located.
Example output:

![alt text][edges]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for perspective lies without get_perspective and update_perspective_data functions. Update_perspective_data is called only once to generate transformation.
get_perspective also supports getting inverse image, and I leveraged it later when drewing lanes.

Example of transformed edges image posted above:
![alt text][perspective_edges]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
There is two pieces for that:
1. fit_polynomial is running sliding window approach per every frame. It tried to leverage the fact that edge pixels will be located on a vertical line.
2. search_around_poly is trying to refit edge pixels withint a tiny window of a currently tracked lane.
By leveraging both algorithms we are able to retrack back to lanes we are lost by accident (e.g. something else looks like a lane)
Than we take whatever is better for that frame.

![alt text][per_lane_pixels]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
Function measure_curvature_real.
Here we take all points within our image, which lies on a lane and scale them appropiately to move from pixel coordinate system to a meter coordinate system.
(ym_per_pix = 20.0/720 # meters per pixel in y dimension
 xm_per_pix = 4.0/700 # meters per pixel in x dimension).
After that we refit new set of points with new polynomial and compute curvature using a formula.

print_car_position - adds car position relative to a center to output image.

I also added averaged out both curvature and vehicle position accross few(~20) frames in order to get rid from outliers and get more reliable data.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![alt text][output]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
Edge detection is the first problematic place. There is a lot of tuning done to be able to get reasonable results for more challenging videos, but even that is far away from being robust to different environment, lighting, etc.
Later it's pretty hard to match edge pixels to a line. Perspective transformation help a bit as it limits pixels which take part in matching, but that might filter out too many pixes (as in a case of hard_challenge_video) it will just cut too many valueable pixels.
It also feels that 2rd order polynomial is not enough to match more trickier cases as in hard challenge - so probably need to go deeper here.
