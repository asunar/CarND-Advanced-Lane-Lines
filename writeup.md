## Advanced Lane Finding Project

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

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the IPython notebook located in "./Advanced Lane Lines.ipynb" in `calculate_camera_matrix_dist_coeffs` function

I start by setting the corner counts and then preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

In the notebook, used a sample calibration image to view original and undistorted images side by side (see `show_calibration_image` function)

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

I first call the `calculate_camera_matrix_dist_coeffs` to get the camera matrix and distortion coefficients then call the `undistort` function to correct the image.

In the notebook, used a test image to view original and undistorted images side by side (see `process_test_images` function)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. They are implemented in these functions.
* `mag_thresh` : Magnitude Threshold
* `dir_threshold`: Thresholded absolute value of the gradient direction
* `hls_select`: Apply threshold to S channel

Here's an example of my output for this step.  (note: this is not actually from one of the test images)

In the notebook, used a test image to view undistorted and thresholded images side by side (see `process_test_images` function)

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

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

In the notebook, used a test image to view original and processed images side by side (see `process_test_images` function)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Identified and fit lane line pixels in the `Lanes` class insided the `detect_lanes` function kinda like this:
![Poly][image5]

If there is not a previously detected lane:
    * Take histogram, find peaks, use sliding windows, extract left and right line pixel positions and fit a poly
    * Set current_fit/best_fit properties of the `Lane` object to use in subsequent frames

If there is a previously detected lane:
    * Search in a margin around the previous line position
    * Set current_fit/best_fit properties of the `Lane` object to use in subsequent frames

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Calculated radius of curvature in the `curvature` function. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `draw_lane_line` function 

In the notebook, used a test image to view original and processed images side by side (see `process_test_images` function)

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./processed_simple.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
I spent a quite bit of time trying to debug my calibration code when I was unable to find any corners for the first image.

I guess conceptually I was unaware of the fact that camera calibration was a cumulative process and that skipping images does not significantly hurt the calibration


First thing comes to mind is low visibility conditions. Due to the weather events lanes on the road may or may not be visible. That may make it difficult for the pipeline to detect the lanes lines on the road. Another issue may be other lines on the road that pipeline may mistake for a lane like tram or ground level train tracks.

I think it will be important to calibrate the lane detection in the pipeline with more data. Maybe during a blizzard other sensors on the car may help determine where the road begins/ends.  
