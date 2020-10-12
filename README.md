## README/Writeup
![alt text][image6]
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

[image1]: ./output_images/image_undistort.jpg "Undistorted"
[image2]: ./output_images/Road_Transformed.jpg "Road Transformed"
[image3]: ./output_images/binary_combo.jpg "Binary Example"
[image4]: ./output_images/warped_straight_lines.jpg "Warp Example"
[image5]: ./output_images/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/example_output.jpg "Output"
[video]: ./output_images/challenge1.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.


The code for this step is contained in the 4th code cell of the IPython notebook.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objectpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imagespoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

With help of the object and image points I use my function image_undistort in the 5th code cell to undistort every image. The function uses the cv2.calibrateCamera and the cv2.undistort function. Here is an example:

![alt text][image2]

#### 2. Describe how you used color transforms, gradients or other methods to create a thresholded binary image.  

I used a combination of color and gradient thresholds to generate a binary image. In code cell 7 I wrote some helper functions to get for exemple the gradient magnitude or the gradient direction. I combine the diffrent threshols in the threshold_pipeline function in the 8th code cell:
```python
combined_binary[((gradx == 1) & (grady == 1)) | ((mag_binary == 1) & (dir_binary == 1)) | (s_binary == 1)] = 255
```
Here's an example of my output for this step:


![alt text][image3]

#### 3. Describe how you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points basicly by try and error but in the end these were the most usefull:


Source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

On this image you can see that the lanes appeare parallel in birds-perspective:

![alt text][image4]

#### 4. Describe how you identified lane-line pixels and fit their positions with a polynomial?

See code cell 12.
At first I toke a histogram of the bottom half of the image and idenfified the left an right maximum, which is the point were the lane lines start. Those are the start points of my search window for lane pixels. I loop through every window and always recenter the next window on the current mean position. (However, when the input is a video I only search around the last polynomial.) After that I fit my lane lines with a 2nd order polynomial like this :

![alt text][image5]

#### 5. Describe how you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I use 3 points. The center of the image, the left lane line, the right lane line. In code cell 14 I identify the offset by comparing the middle of the image to the middle of the lane line. 
I calculated the radius of curvature by fitting a new polynomials to x,y in world space. After that I defined the y-value where I want the radius of curvature, which is at the button of the image. In the end I calculate the radius whith the polynomina and the y-value.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The whole image pipeline is in cell 17.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

Here is a link to my [video]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

If there is no left/right lane line, because of a strong turn, my pipline does not work. Since I am always searching for an area between two lines. I could improve the pipline by specifying that there has not to be a line, if there are too less pixels or if the curve has a small radius. In that case I could expand the driveable road part to the left/right end of the image. 
