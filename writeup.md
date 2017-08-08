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

[image1]: ./examples/undistorted_checker.png "Undistorted"
[image2]: ./output_images/undist.jpg "Undistorted Lane"
[image3]: ./examples/perspective_transform.jpg "Road Transformed"
[image4]: ./output_images/threshold.jpg "Binary Example"
[image5]: ./output_images/warped.jpg "Warp Example"
[image6]: ./output_images/viz.jpg "Fit Visual"
[image7]: ./output_images/viz2.jpg "Fit Visual 2"
[image8]: ./output_images/final.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

#### 1. Camera Calibration and Image Undistortion

The code for this step is contained in the first code cell of the IPython notebook located in "Advanced-Lane-Lines.ipynb". I created a class called Calibrate which I use to perform the calibration, store and reload the calibration.

Within the Calibrate class is a function called `get_points()` which will use the 20 images in the camera_cal directory. I use the function `cv2.findChessboardCorners()` to retrieve the corners for each calibration image, which I append to a list of imgpoints, along with objpoints which will be the (x, y, z) coordinates of the chessboard corners in the world, for each image. I store the resulting lists in a pickle file for later calibration loading.

I use the stored imgpoints and objpoints to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the the `Calibrate` function `undistort_image` which uses the camera calibration and `cv2.undistort()` function and obtained this result: 

![alt text][image1]


#### 2. Image Filtering with Thresholds

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at methods `dir_threshold`, `mag_threshold`, `color_threshold` in the `Thresholder` class from in the 6th code cell of the notebook).

I first applied color filtering, keeping only white or yellow-ish pixels of the image. I then only kept pixels that either match a sufficent magnitude or direction threshold.

For magnitude theshold I chose min and max values of 0.7 and 1.2.

For direction threshold I chose min and max values of 20 and 100.

The following image is an example of the binary image:

![alt text][image4]

#### 3. Image Warping

The code for my perspective transform includes a function called `warp_image()`, which appears in section titled "Perspective Transform" and is in the 3rd code cell.  The `warp_image()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([
    [580, 460],
    [700, 460],
    [1040, 680],
    [260, 680],
])

dst = np.float32([
    [260, 0],
    [1040, 0],
    [1040, 720],
    [260, 720],
])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 260, 0        | 
| 700, 460      | 1040, 0       |
| 1040, 680     | 1040, 720     |
| 260, 680      | 260, 720      |

In the same section is a function called `apply_perspective_transform` which uses `warp_image` along with the `src` and `dst` points to warp an image. I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

#### 4. Lane Detection

The code for lane detection can be found in the `Polyfitter` class in the 9th code cell of the notebook. I first computed the histogram of the picture on its lower half to find the rough position of each lane line - this can be found in the beginning of the `polyfit_sliding` function.

The `polyfit_sliding` function then uses a sliding window vertically to detect the position of the center of each lane line on each part of the image.

![alt text][image6]

Once I had a general idea of where the lines were I use the function `polyfit` to search in a margin around the previous line position and then return the resulting `np.polyfit` for each left and right lane line.

![alt text][image7]

#### 5. Measure Lane Curvature 

I used the computed polylines along with the estimated lane width (~3.7m) to compute the real-world lane curvature. I also measured the position of the car in respect to the lane by computing the difference between the center of the lane and the center of the image.

These computations can be found in the `measure_curvature()` method of the `Polyfitter` class.

#### 6. Displaying Detected Lines

This involved creating a polygon using the curves of each computed polyline and then warping the result back using the reversed src and dst defined in step 3. I implemented this in the `draw` function of the `Polyfitter` class. Here is an example of my resulting output from the project video:

![alt text][image8]

---

### Pipeline (video)

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

Over all I enjoyed the project as a exercise in computer vision, but it made me further appreciate deep learning approaches. 

My implementation is heavily weighted towards the initial project video and will fail on the challenge videos which have much more variation in lane positioning and road details. 

Additionally its tuned specifically to deal with yellow and white lines and different colored lines (for example due to weather or night conditions) will throw off my thresholding. 

I choose a static polygon for my perspective warp and in real world conditions this wouldn't be very effective givin changes in lane dimensions and non-flat/hilly roads.

Finally my implementation doesn't make an attempt to use previously found lane line details in the event that no lane lines are detected. 

  
