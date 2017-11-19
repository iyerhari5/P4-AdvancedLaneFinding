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
[image8]: ./output_images/color_threshold_1.png
[image9]: ./output_images/color_threshold_2.png
[image10]: ./output_images/color_threshold_3.png
[image11]: ./output_images/lane_finding.png
[image12]: ./output_images/lane_tracking.png
[image13]: ./output_images/project_video_output_debug.gif


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

Another option is to use gradient based detection of the lane pixels. However this method seemed to produce lot of artifacts especially when there are uneven shadows in the image. So I did not pursue this further.

After playing with different color spaces and thresholding parameters, I decided to use the L channels from the HLS color space and the B channel from the LAB color space to get the lane pixels. In order to make the pipeline more robust to partial shadows etc, I use the [CLAHE algorithm] (https://en.wikipedia.org/wiki/Adaptive_histogram_equalization#Contrast_Limited_AHE) to normalize the channel of interest before thresholding. Figure below shows for a sample image the selected channels, the normalized image and the final thresholding for each of those.
![alt text][image8]
![alt text][image9]

The final lane pixel mask is found by combining the information from both the above. Figure below shows a sample image and the final detected lane pixels 

![alt text][image10]

## Curve fitting to localize the lanes

Now that we have the pixels potentially belonging to the lanes, we need a better way to identify the correct lane pixels and also to form a smooth lane boundary. We do this by fitting a second order polynomial to the left and right lanes separately. Initially, the lane pixels are identified using a window search starting from a peak detection in the column wise histogram of the pixel mask. Once the starting location is found, we move the window veritically and search for lane pixels within this window with a margin for movement. If sufficient number of pixels are found, then the x-location of the window is updated to the mean x position of the pixels inside the window. This procedure is show in the figure below. The boxes are colored in green if sufficient number of pixels were found inside. Otherwise the box is colored in cyan. You can notice that when the box is cyan,its x location does not change from the last green window.
![alt text][image11]

The pixels highlighted in red (for the left lane) and in blue (for the right lane) are then used to fit a 2nd order polynomial to get the lane boundaries from top to bottom of the image. The fit is superimposed in yellow in the image above.

Once we have an estimate of the lanes, we can do a much more targeted search for the lane pixels. This is done by searching in the vicinity of the polymonial fit from the previous frame. This procedure is illustrated in the figure below. The green region around the pixels shows the search region and the selected pixels for the curve fit for the current frame are displayed in red and blue as before. The road image used here is the next consecutive frame from the road image used in the previous figure. You can see how this procedure may help to track the lanes efficiently even through turns and ligting changes.
![alt text][image12]

## Making the pipeline robust

The above steps help to identify the lane pixels and to fit a polynomial. There are many cases, when we are not able to find a reliable fit or the fit is affected by spurious lane pixel detections. I introduced some robustness into the system by putting in a few checks before using the fit from the current frame.  The bottom and top ends of the line fit for each of the left and right lanes are first stored. If the current fit produces line endings that are too far from the previous frame, then the new fit is not used. This is done separately for the left and right lanes. This helps to track the lanes better. The tolerance for movement of the line endings depend on the expected movement and was set empirically based on a few trial runs to 50 pixels.   

## Radius of curvature and position determination

The radius of curvature is calculated using the following formula:
```
curverad =  ((1 + (2*fit_cr[0]*y_eval*ym_per_pix + fit_cr[1])**2)**1.5) / np.absolute(2*fit_cr[0])
```

Here fit_cr stores the coefficients of the polynomial fit and y_eval is set to the bottom of the image (as that corresponds to the car position)
The left and right radius of curvature are computed separately and averaged  to report the final radius of curvature of the road.

The position of the car in the lane is calculated using the following formula:
```
offset = shape[1]/2*xm_per_pix - (left_bot_x+right_bot_x)/2
```

Here the first term is the car position which is assumed to be the center of the image along x. The left_bot_x and right_bot_x are the x-interxepts of the line fits evaluated at the bottom of the image (highest y). xm_per_pix is the physical dimension of each pixel in meters.


## Results 

Here are the results on the project video. The top panel shows the bird's eye view of the lanes, segmentation of the lane pixels and the final lane tracking and polynomial fits. The bottom panel shows the segmentation of the lane region overlaid on the original distortion corrected image.

![alt text][image13]

The output from the challenge video is [here](https://github.com/iyerhari5/P4-AdvancedLaneFinding/output_images/challenge_video_output_debug.gif)


## Possible Improvements

* Explore gradient based lane detection to augmnet the color thresholding
* Explore adaptive thresholding stragegies to better handle brightness variations