
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

[image1]: ./figures/undistort_output.png "Undistorted"
[image2]: ./figures/undistort_output_test.png "Undistorted Test"
[image3]: ./figures/binary_combo_example.png "Binary Example"
[image4]: ./figures/warped_straight_lines.png "Warp Example"
[image5]: ./figures/color_fit_lines.png "Fit Visual"
[image6]: ./figures/example_output.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "advanced_lane_tracking.ipynb".  

Firstly, the number of inside corners are set to (9,6) in x and y direction according to the image. Then I created an array of "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at the fourth code cell).  Here's an example of my output for this step.

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
The code of computing the perspective transformation is in the 5th code cell of the notebook. The points in `src` are manually identified from the first test image which has straigth lane lines. I tried to obtain four points that can form a rectangle. The points in `dst` are set by trial-and-error in order to make sure that the lane lines are in the image while excluding non-lane line pixels.

```
src = np.float32(
        [[749,489],
        [921,597],
        [382,596],
        [539,490]] )
dst = np.float32(
        [[1000,400,],    
        [1000,630],
        [250,630],
        [250,400]] )

```
This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 749, 489      | 1000, 400        |
| 921, 597      | 1000, 630      |
| 382, 596     | 250,630      |
| 539,490      | 250,400        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
I utilized the histogram method described in the lexture to identify the lane-line pixels in the warped binary image. The code is in the 7th code cell.

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The curvature is computed using the method described in the lecture while the position of the vehicle with respect to the center is computed between the distance of the middle point of the right and left lane base point (at the bottom of the image) and the image center.  This is implemented in the 7th code cell of the notebook.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step inthe 7th code cell.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](https://drive.google.com/file/d/0B8qitkbIZglGUVN1SXc1RnY5Nkk/view?usp=sharing). 

###2. Implementation details of the tracking

The lane tracking is implemented in the 8th code cell of the notebook. For the first frame or any frame that the previous frame fails to detect lane lines, the class `LaneLine` utilizes the method describe above to detect the lane lines. If lane lines are found in the previous frame, the current frame will utilize the previous lane finding results to find lane line pixels in the warped binary images. If both the left and right lanes are correctly found and the current lane curvature is close to the previous lane curvature (the difference is smaller than 0.8), the current lane estimation will be remained. Otherwise, the previous lane fitting results will be used and the current ones are discarded(in `LaneLine::validate_lane_fitting`).
---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

For the `challenge_video`, the method sometimes utilizes the wall base as the left lane. Since in our system the left lane is usually in yellow, I would suggest to give more weight to yellow pixels in order to localize the correct lane lines. In addition, the current implementaion, estimat the right and left lanes separately. In fact, as we know that they are parallel to each other, we can explore to jointly estimate their parameters. Moreover, if one lane is detected with hight confidence (for exmaple, containing many edge pixels while being consistent with the previous estimation), the other one can be inferred when it is cannot estimated using the current method.

For the `harder_challenge_video` , the mehtods will fail when the shawdow is heavy, or bright sun light is on the road. In addition, due to the sharp change of the lane lines, using the previous result generates wrong lane lines. Apart frome the mehtod metioned above, I would also suggest that we can perform some pre-processing of the image to mitigate the effect of the bright sun light (for exmpale using some low-pass filter). Moreover, for these pictures, we might have to consider setting specific parameters for these images. If computational rescoures are avaible, we can have a set of thresholds for different situtations to find the best lane fitting results.
