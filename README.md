

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

[image1]: ./results/camera_cal/image_udist.png "Undistorted"
[image2]: ./results/output_images/road_transformed.png "Road Transformed"
[image3]: ./results/output_images/binary_image.png  "Binary Example"
[image4]: ./results/output_images/warped_straight_lines.png "Warp Example"
[image5]: ./results/output_images/color_fit_lines.png "Fit Visual"
[image6]: ./results/output_images/example_output.png "Output"
[image7]: ./results/output_images/marker_warp.png "Output"
[video1]: ./results/output_videos/project_video.mp4 "Video"

## Steps followed

### Camera Calibration

#### 1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.

The code for this step is contained in the src/CameraCalibration.py.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Apply a distortion correction to raw images.

To demonstrate this step, I used cv2.undistort function to apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Binary Image: Convert the image to HLS space, threshold L (intensity) and S (color) and combine these to create a binary image

I used a combination of color and gradient thresholds to generate a binary image. The function `black_binary_image` (6th cell) in `src/LaneFinding.ipnyb` takes in the image and threshold and returns the binary image.  Here's an example of my output for this step.  

![alt text][image3]

#### 3. Apply a perspective transform to rectify binary image ("birds-eye view").

The code for my perspective transform includes a function called `perspective_transform()`, which appears in lines 1 through 8 in the file `src/LaneFinding.ipnyb` (example, in the pth code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src_vertices`) and destination (`dst_vertices`).  I choose to hardcode the source and destination points in the following manner:

```python
src_vertices_roi = np.float32([(575, 460), (190 ,image_shape[0]), (1125, image_shape[0]), (710, 460)])
offset = 190
dst_vertices_roi = np.float32([[offset, 0], [offset, image_shape[0]], 
                                     [image_shape[1]-offset, image_shape[0]], 
                                     [image_shape[1]-offset, 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 575, 460      | 190, 0        | 
| 190, 720      | 190, 720      |
| 1125, 720     | 1090, 720      |
| 710, 460      | 1090, 0        |

I verified that my perspective transform was working as expected by drawing the `src_vertices` and `dst_vertices` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
![alt text][image7]
![alt text][image4]

#### 4. Detect lane pixels and fit to find the lane boundary.

Then I used 2D convolutions with a sliding windows from left to right of the image to detected the line. Height of convolution is 1/9 of the image height (too small leads to give way to more noise, too large leads to approximation error of the curve). So the loop runs 9 times to get all the edge points in the left and right part of the image. The code for this section is in cell 49 `find_window_centroids` The points are the passed to cv2.polyfit function to fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Determine the curvature of the lane and vehicle position with respect to center.

I modified the Line class to calculate and store the radius as well as vehicle position. The code is in cell 52 in `line_from_pixels` function. There are a bunch of sanity check functions in the the Line class to check if the lines detected are parallel, similar to lines detected in previous frames and are of justifible width. 

#### 6. Warp the detected lane boundaries back onto the original image.

I implemented this step in 53rd cellin the function `fill_pixels_road()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Here is the link to the video. All 3 video's can be found in result/output_videos directory.

Here's a [link to my video result][video1]

---

### Discussion

#### 1. Techniques used to make pipeline more robust
Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
