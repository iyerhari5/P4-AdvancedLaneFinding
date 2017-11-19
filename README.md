# Advanced Lane Finding

The goals of this project are the following:

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
[image3]: ./output_images/calibration_result_road.png
[image4]: ./output_images/perspective_projection_result.png
[image5]: ./output_images/color_space_sample1.png
[image6]: ./output_images/color_space_sample2.png
[image7]: ./output_images/color_space_sample3.png


## Camera calibration and distortion correction

We use a number of calibration images to estimate the camera parameters. The images are from a known checkerboard pattern with black and white
alternating squares. There a total of 70 (10x7) squares in each image. We use the OpenCV function findChessboardCorners() to find the corners
of the checkerboard image. The routine looks for the interior corners in the image. So there are a total of 54(9x6) expected corners. 
The find_calibration_parameters() routine takes the calibration directory as input and proceses each of the calibration images. If all the expected corners are found successfully, they are added to a list storing image points. For each of the detected points, we know the corresponding  world coordinate and this is also added to a list. Finally we use OpenCV function calibrateCamera() to compute the camera distortion parameters.

Figure below shows the calibration images with the detected corners in them. For three of the images, the corners were not detected and they are represented as blank images in the figure below.
![alt text][image1]

Once we have the calibration parameters, we can use the OpenCV function undistort() to remove the distortion from a given image. The figure below shows a sample calibration image before and after distortion correction.
![alt text][image2]

We can now use the distortion parmeters calculated to the road images of interest. The figure below shows a sample road image before and after distortion correction. The effect of the distortion correction can be noticed  mainly if we look at the dashboard near the bottom of the image.
![alt text][image3]

## Bird's eye view

In order to segment the lanes properly, we want to create a "bird's eye" view of the road images. We can achieve this by creating a perspective transform.  We need to first find the transformation matrix for this. This can be done by selecting points from an image and defining the points that we want to map the selected points to. The main goal of the perspective transform is to visualize the lane lines so that they are parallel (at least when on a straight road), rather than leaning inwards towards the vanishing point. I selected the points from an image with a straight portion of the road and then defined the mapping points that would map the straight lines to vertical lines in the transofrmed image. Once we have the points defined, we can use the OpenCV function getPerspectiveTransform() to get the transformation matrix (and the inverse transformation).
The function warpPerspective() can be used to transform an image using the computed transformation matrix. Figure below shows a sample image and the 
transformed bird's eye view of the same image.
![alt text][image4]

## Detection of lane pixels

The next step in the pipeline is to identify pixels belonging to the lanes. In order to see what might be a good way to segment the lane pixels, we first explore a few color spaces to see what might help to identify the lane pixels. Figure below shows a few sample color spaces explored and the individual channels from those color spaces. Here we look at the HSV, LAB and HLS color spaces.
![alt text][image5]
![alt text][image6]
![alt text][image7]
