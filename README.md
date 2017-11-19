# Advanced Lane Finding

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

[image1]: ./output_images/calibration_find_corners.png
[image2]: ./output_images/calibration_result.png
[image3]: ./output_images/Training-Distribution-After.png
[image4]: ./output_images/9-layer-ConvNet-model.png
[image5]: ./output_images/run1.gif
[image6]: ./output_images/run2.gif


## Camera calibration

We use a number of calibration images to estimate the camera parameters. The images are from a known checkerboard pattern with black and white
alternating squares. There a total of 70 (10x7) squares in each image. We use the OpenCV function findChessboardCorners() to find the corners
of the checkerboard image. The routine looks for the interior corners in the image. So there are a total of 54(9x6) expected corners. 
The find_calibration_parameters() routine takes the calibration directory as input and proceses each of the calibration images. If all the expected corners are found successfully, they are added to a list storing image points. For each of the detected points, we know the corresponding  world coordinate and this is also added to a list. Finally we use OpenCV function calibrateCamera() to compute the camera distortion parameters.

Figure below shows the calibration images with the detected corners in them. For three of the images, the corners were not detected and they are represented as blank images in the figure below.
![alt text][image1]

Once we have the calibration parameters, we can use the OpenCV function undistort() to remove the distortion from a given image. The figure below shows a sample calibration image before and after distortion correction.
![alt text][image2]

