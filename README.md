## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


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
[image2]: ./examples/undistort_output2.png "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image3.1]: ./examples/binary_combo_example2.jpg "Binary Example"
[image3.2]: ./examples/binary_combo_example2.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[poly2]: ./examples/polyy2.png "Output"
[curv]: ./examples/curv.png "Output"
[video1]: ./out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / REA

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.


The code for this step is contained in the code cell [2] of the IPython notebook "Find_lanes.ipynb"

There we define a function call that start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here we assume the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time all chessboard corners are detected in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  We then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.

On the cell [3] we define a function that does distortion correction to an image using the `cv2.undistort()` function.

In Cell [4] We calibrate the camera and do distortion correction of a sample image, yielding the following results:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the afore defined functions we can remove camera distortions from our dashboard images.
We do this in cell [5]
The result is :

![alt text][image2]



#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in cell [6].  The `warp()` function takes as inputs an image without camera distortions (`undist`) and does the homography transformation to a top view image where the straight lines of lane are parallel.
For this task we hardcode the source and destination points in the following manner:

```python
src = np.float32([[580,460],
                  [705,460],
                  [1046,680],
                  [270,680]])
pts = src.astype(np.int32)
pts = pts.reshape((-1,1,2))


dst = np.float32([[340+0,100],
                  [340+600,100],
                  [340+600,700],
                  [340+0,700]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 580, 460      | 340, 100       |
| 705, 460      | 1040,100      |
| 1046, 680     | 1040,700      |
| 270, 680      | 340, 700       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]



#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

First we will color code an RGB image with different image segmentation techniques.
To do that we define the function `color_code()` in cell [8] .

We will create a color coded image based on the following process:
The red channel is defined by thresholding the input image  using the L channel of HSV colorspace, and the B channel LAB colorspace. The threshold is hardcoded to be 200 and 150 for L and B respectively.

The green channel is based on thresholding the sobel gradients on the X direction.
We  only keep the gradients of the sobel filter in the x direction where their scaled magnitude is between [30 ,100].

The blue channel is based on a combination of the previously computed sobel in the x direction and select
only the pixel in which the and the saturation channel of HSV colorspace is between [30,255]


These are some results on color coding our distortion corrected image and our warped image:

![alt text][image3]


To compute the  final binary image of a warped image we defined the function `binary_image()` in cell [10]. This function takes a distortion corrected color image as input, color code and warp it using the afore mentioned functions. It then repeat the process by first warping the input image and then color coding it.  By combining the two color coded images we then derive a very robust binary image.

![alt text][image3.1]



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

By dividing the image in smaller region of interest and implement a sliding window analysis, we can find peaks in histograms that are more likely pixel belonging the the lane boundaries.

The function `Find_lanes()` defined in cell [12] is used to find probable pixel of the left and right lanes.

The cell [13] shows the achieved results when using a binary wrapped image as input.


![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.


To fit a second order polynomial that could describe the possible lane. We implemented a Line class.
The class will take care of tracking and fitting the line parameters.

The class also computes as binary mask that is used to define region of interested in which we will look for lanes on possible next frames :

![alt text][image3.2]


The line class also computed the curvature curves:
Assuming that the boundary of lane can bes described by

![alt text][poly2]

The radius of the curve described by that function is
![alt text][curv]
​​


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in  cell [19].
It uses the function `carview` define in cell [17] to warp to top viw back to the dashboard view.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?


This project is very sensitive to fine tunning of our lane segmentation. I do not believe a hand-feature engineering approach like this should be the at core of such task. Although it works, and when well tuned give great results, we can leverage the power of machine learning to learn better ways to detect lanes and have even better results. It is frustrating to hand tune in the algorithm to fit the specification on the rubric when a more robust approach could be done if we were not obliged to do a gradient and color threshold method as stated.
