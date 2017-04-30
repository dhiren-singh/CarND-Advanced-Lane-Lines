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

[image1]: ./output/Distortion.png "Undistorted"
[image2]: ./output/Distortion2.png "Actual Image Undistorted"
[image3]: ./output/Threshold.png "Threshold Calculations"
[image4]: ./output/PerspectiveTransform.png "Perspective Transform Points"
[image5]: ./output/PerspectiveTransform1.png "Actual Image Perspective Transform Points"
[image6]: ./output/Histogram.png "Histogram"
[image7]: ./output/LanePoints.png "Lane Points"
[image8]: ./output/LaneArea.png "Lane Area"
[image9]: ./output/LaneAreaOverlay.png "Lane Area Overlay"
[image10]: ./output/FinalOutput.png "Final Output"
[video1]: ./project_video_labeled.mp4 "Project Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cell 3 of the IPython notebook located in "./CarND-Advanced-Lane-Lines.ipynb". 

* Convert the images to grayscale.
* Apply the findChessboardCorners to get all corners.
* Call calibrateCamera method of opencv to get the camera cliberation matrix and coefficients.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Undistort.

Undistort Method is defined in code cell 4. Using this method one of the test image is undistorted.
![alt text][image2]

#### 2. Sobelx

I calculate the sobelx for the image. Apply threshold and return binary. Code in cell 7.

#### 3. S Channel Threshold

I take the s channel of image and apply threshold. Code in cell 8.

#### 4. Color Selection

I mask White & Yellow. Then merge the the masks. Code in cell 9.

#### 5. Combining Thresholds.

Now combine binary of all these different thresholds. Code in cell 10.

On an test image I apply these thresholds:

![alt text][image3]

#### 7. Perspective Transform

Now as next step in the pipeline I take perspective transform. By multiple experiments on the project video I decided to take the value of `src` & `dst` as:

```
src_mat = [
    [230,695],
    [578,458],
    [704,458],
    [1065,695]
]

dst_mat = [
    [220,h],
    [220,0],
    [1060,0],
    [1060,h]
]
```
These points plotted on a test image:

![alt text][image4]

In code cell 15 the code for percpective transform & inverse transform is written.

Test on a test image:

![alt text][image5]

This looks fairly good to proceed ahead.

#### 8. Curvature Calculation

Here in code cell 18 & 19 I define the curvature calculation & distance calculation methods which would be used later to identify the curvature & distance from center of lane.

#### 9. Lane Points

To identify lane pixels apply following steps.

1. Load a thresholded image
3. Define verticle slices.
4. For each vertical slice
	* Create a histogram.
	* Smooth the histogram with a generalized Gaussian shape (method: signal.general_gaussian)
	* Calculate the relative extrema of data (method: argrelextrema)
	* Split extremas to left and right lanes based on their x value
	* If only one extrema is found on each side store them for later processing
	* If two are found, check if they are close (distance < threshold)
	* If they are close store their average for later processing
	* In other cases, ignore the points for this slice

Code in cell 20.

Histogram:

![alt text][image6]

Lane Points on sample image:

![alt text][image7]

#### 10. Lane Lines

Using the Lane Points we fit them in polygon to fit and then fill the polygon. using this curvature & the area is overlay on the image.

![alt text][image8]

Then we do a inverse transform and merge images:

![alt text][image9]

### Using the pipeline on a video.

In order to us the pipeline on a video we had to look into smothening the lanes that we had detected. So started maintaining history of detection & used this while overlaying. Code in cells 26-28


---

### Pipeline (video)

#### 1. Execution on Project Video

Final output:

![alt text][image10]

Here's a [link to my video result](./project_video_labeled.mp4)

#### Summary
In project video the lane area is properly labeled. At all times the Lane is properly visible. Label shows 'Left' & 'Right' Curvature properly. Car position from the lane centre is displayed.

#### 2. Execution on Challenge Video

Here's a [link to my video result](./challenge_labeled_video.mp4)

#### Summary

At all times the Lane Area is properly annotated. At some places the curvature isn't properly detected. Overall more work in smothening is needed.

#### 3. Execution on Harder Challenge Video

Here's a [link to my video result](./harder_challenge_labeled_video.mp4)

#### Summary

n this video of annotation is way away. Tried experimenting with the histogram in order to detect two lines very close as one.

---

### Discussion

#### 1. Challenges

Some of the challenges that I would like to solve are: 

* Identifying the Source & Destination polygon for Perspective Transforms.
* Detecting double hard lines using Histogram.
  
