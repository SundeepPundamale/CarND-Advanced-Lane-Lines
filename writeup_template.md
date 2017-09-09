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

[image1]: ./examples/undistorded_image.png "Undistorted"
[image2]: ./examples/test2.jpg "Road Transformed"
[image3]: ./examples/thresholded_binary_image.png "Binary Example"
[image4]: ./examples/warped_img_src_dst.png "Warp Example"
[image5]: ./examples/color_fit_lines_result.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[image7]: ./examples/chessboard_corners.png "Chessboard Corners"
[image8]: ./examples/undistorted_test2.png "Original & Undistorted Test Image"
[image9]: ./examples/lane_markers_polyfit.jpg "Lane Markers using Polyfit"
[image10]: ./examples/roc_lane_marking.jpg "Lane Markering and Radius of curvature"


[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in 1st and 2nd code cell of the IPython notebook located in `./advanced_lane_finding.ipynb`  

I start by preparing `object points`, which will be the (x, y, z) coordinates of the chessboard corners in the real world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `object_points` is just a replicated array of coordinates, and `object_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `image_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. Corners were found successfully for 17 images and for 3 images it was unsuccesful. Below are the results i obtained:
![alt text][image7]


I then used the output `object_points` and `image_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image, in this case calibration3.jpg in the camera_cal folder using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

I applied this distortion correction to the test image, in this case test2.jpg from the examples folder, it is a copy of the test2.jpg image test_images folder, using the `cv2.undistort()` function and obtained this result:
![alt text][image8]

The undistort seem to have an impact on the edge of the image such as carhood, car on the oncoming traffic lane which is not visible in the undistorted image and bushes on the edges appear closer

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image thresholding steps in cell 5 in the `advanced_lane_finding.ipynb` Jupyter notebook. I used the HLS color space and combined the S channel and the gradient threshold which is a derivative along the X axis. Below is an example of my output for this step. However, in my project pipeline below i used the S & V channels with the combined X and Y thresholds. 

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in cell 6 of the IPython notebook located in `./advanced_lane_finding.ipynb`  .  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

Comments for reviewer: I saved the images with polylines to the examples folder with the following names: `image_with_polylines.jpg` and `warped_image_with_polylines.jpg`

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To implement the project pipe line i referred the [Udacity Q&A youtube video](https://www.youtube.com/watch?v=vWY8YUayf9Q&list=PLAwxTw4SYaPkz3HerxrHlu1Seq8ZA7-5P&index=4). I implemented the line tracker in a python class named `./tracker.py`. I used the sliding window approach by sliding the window placed around the line centers and follow the lines to the top of the frame. As shown in the below image:

![alt text][image5]

Then I fit my lane lines with a 2nd order polynomial using the `polyfit` function and parameters `res_yvals`,`left_fit` and `right_fit`, res_yvals are the heights of the box centers, the left_fit and right_fit have the centre values of left and right lane as we are sliding through the windows. I then drew the lane markers using the fillPoly method. As a result i see the following output with red and blue lane markers on the road:
![alt text][image9]

It is implemented in code cell 8 from line 62 to 97 in `./advanced_lane_finding.ipynb`. Code cell 8 is used to save the images and code cell 9 is the same code but used for the pipeline with the final ouput as the video file.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I implemented this step in code cell 8 from line 105 to 125 in  `./advanced_lane_finding.ipynb`. The radius of curvature is implemented based on the math mentioned in the following [reference](https://www.intmath.com/applications-differentiation/8-radius-curvature.php). The `curve_fit_cr` calculates the curvature in meters and `curverad` calculates the radius of curvature based on the math in the reference link above.

Comments for reviewer: I fixed the radius of curvature by adjusting the meters to pixel ratio and changing the space from warped image to image

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

 Here is an example of my result on a test image which can be found in the examples folder with name `roc_lane_marking.jpg` :

![alt text][image10]


---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_result.mp4)

Comments for reviewer: 
- To fix the lane lines i tried the following:
    - I tried the L and B color channels but that made the lanes even worse
    - I tried S and B that didn’t help either, it still worsened the lanes
    - I next tried to implement the sanity check. I referred the Udacity class which explains about Sanity check. I am blocked at this point. Fixing the distance alone will not help to rectify lanes, fixing the radius of curvature alone will neither. By combining those two i still can’t think a logic how to take a reference point for the correct horizontal distance and curvature because the lane could either bend left or right and keeping the old reference point might not solve the issue. Could you kindly give me pointers on how to proceed further.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

One of the challenges i faced was collecting data points from the image pipeline. I designed it such that that final output is a video with lane marking, radius of curvature and the vehicle postion. I later used an additional code cell to log my image outputs as i progressed through each step. The pipeline could fail when there is a sharp curve such that lane ahead is in the blind spot. In such situations i believe Computer vision alone cannot help to keep the vehicle in center of the lane and might have to rely on digital maps. The pipeline could also fail when there is construction work and there are overlapping lane lines due to the construction work. The pipeline could be further enhanced to prioritise yellow lane markers over white lane markers to handle such situations
