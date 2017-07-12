
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

[image1]: ./output_images/undistorted/output.png "Undistorted"
[test_image]: ./test_images/straight_lines1.jpg "Test Image"
[undistorted_test_image]: ./output_images/undistorted/straight_lines1.jpg "Undistorted Test Image"
[mag_threshold]: ./output_images/writeup/mag_threshold_output.png
[direction_threshold]: ./output_images/writeup/direction_threshold_output.png
[abs_threshold]: ./output_images/writeup/abs_threshold_output.png
[final_thresholded]: ./output_images/writeup/final_thresholded_output.png
[warped_image]: ./output_images/writeup/warped_image.png
[thres_warp_summary]: ./output_images/writeup/thresholding_warping_summary.png
[sliding_window_plot]: ./output_images/writeup/sliding_window_plot.png
[lane_fit_plot]: ./output_images/writeup/lane_fit_plot.png
[final_detection]: ./output_images/detection/test4.jpg

[video1]: ./project_output_3.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

My submission contains the following files.
* p4.ipnb : IPython notebook used for solving the problem.
* project_output_3.mp4 : Video where the lane markings have been drawn.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first goal of the IPython notebook titled `Goal 1: Compute the camera calibration matrix and distortion coefficients given a set of chessboard images`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

##### Original Image
![alt text][test_image]

##### Undistorted Image
![alt text][undistorted_test_image]

Notice that the hood has been pulled back during un-distortion.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I had used three thresholding 

##### Absolute Sobel Threshold
Kernel size : 3

Min Threshold : 20

Max Threshold : 100

![abs_threshold]

##### Direction Threshold
Kernel size : 15

lower Threshold : 0.7

upper Threshold : 1.3

![direction_threshold]

#### Magnitude Threshold
Kernel size : 5

Min Threshold : 20

Max Threshold : 100

![mag_threshold]


#### Final Threshold
I had merged the above three theshold with the following logic
```python
combined[((gradx == 1) & (grady == 1)) | ((mag_s_binary == 1) & (dir_s_binary == 1))] = 1
```
![final_thresholded]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I chose to map the source and destination points in relation to image size.

The logic for that is

```python
src_points = np.float32(
    [
        [(width/2) - size_top, height*0.65],        
        [(width/2) + size_top+40, height*0.65],        
        [(width/2) + size_bottom + 100, height-50],
        [(width/2) - size_bottom, height-50]
    ])

dst_points = np.float32(
    [
        [(width/2) - size_bottom, height*0.15],        
        [(width/2) + size_bottom, height*0.15],        
        [(width/2) + size_bottom, height],
        [(width/2) - size_bottom, height]
    ])
```

The warping and upwarping image looked like the following
![warped_image]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

If we aggregate all the steps mentioned above, we get a warped binary image.
![thres_warp_summary]

Here we if we plot the top of binary wrapped image with frequency of positive pixels. We can see that the two lanes are very easily detected.
![sliding_window_plot]

Hence we will be using a rolling window technique to figure out pockets of screen area where lanes are present.

Then we will polyfit these pixels to approximately fit a 2nd order curve from the left and right side. End result looks like this
![lane_fit_plot]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Since we now have a 2nd order curve for both left and right side. We can calculate the radii for each curve as mentioned in http://www.intmath.com/applications-differentiation/8-radius-curvature.php

For calculating the position on road, we calculated the deviation of image's middle point with the left and right lane's average x axis position.

For this we assumed that `720` pixels approximate 30 meters on y axis and 700 pixels correspond to 3.7 meters on x axis. These values have been taken from the course material.


This calculation has been mentioned in `Goal 6: Determine the curvature of the lane and vehicle position with respect to center.`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `render_lane()`.  Here is an example of my result on a test image:

![final_detection]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a link to my video result https://www.youtube.com/watch?v=ZtVfJFwAwyE

You can also find it as project_output_3.mp4 at the root of submission.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?


Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
