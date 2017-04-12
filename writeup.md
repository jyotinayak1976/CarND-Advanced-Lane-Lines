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

------------------------------------------------------------

## 1. Camera Calibration
--------------------------------

Please refer to the code in advanced_Lane_finding_test_1. I have used OpenCV supplied findChessboardCorners and drawChessboardCorners to calibrate the camera. This is being done by identifying the corners of series of pictures of a chessboard taken from different angles.

The locations of the chessboard corners were used as input to the OpenCV function calibrateCamera to compute the camera calibration matrix and distortion coefficients. I saved them as a pickel file in camera_cal for future use.

## 2. Distortion Correction

The camera calibration matrix and distortion coefficients are used OpenCV function undistort to remove distortions. Please refer to the code in advanced_Lane_finding_test_1. The undistorted images are stored in test_images library as undist_test*.

## 3. Perspective Transform

The goal of this step is to transform the undistorted image to a "birds eye view" of the road which focuses only on the lane lines and displays them in such a way that they appear to be relatively parallel to eachother (as opposed to the converging lines you would normally see). To achieve the perspective transformation I first applied the OpenCV functions `getPerspectiveTransform` and `warpPerspective` which take a matrix of four source points on the undistorted image and remaps them to four destination points on the warped image. The source and destination points were selected manually by visualizing the locations of the lane lines on a series of test images. You can find the code in advanced_Lane_finding_test_1, advanced_Lane_finding_test_2. I have used advanced_Lane_finding_test_1 for checking and testing independently different images. 

## 4. Convert the warped image to different color spaces and create binary thresholded images which highlight only the lane lines and ignore everything else. 
I found that the following color channels and thresholds did a good job of identifying the lane lines in the provided test images:

The L and S Channel from the HLS color space, with a min threshold of 50 and a max threshold of 255, did a fairly good job of identifying both the white and yellow lane lines. I had initially used only S channel in advanced_Lane_finding_test_1. But that did not give me a good result. The whitelines were completely ignored. Hence I applied both in advanced_Lane_finding_test_2 for a single image only.

I also applied SOBEL edge dection , with a min of 0 and max of 255, in both x and y direction to find the lanes.

I chose to create a combined binary threshold based on the three above mentioned binary thresholds, to create one combination thresholded image which does a great job of highlighting almost all of the white and yellow lane lines.

## 5. Fit a polynomial to the lane lines, calculating vehicle position and radius of curvature

I completely followed the procedure mentioned in the Udacity tutorial. I am briefly explaing the processes here.

Identifying peaks in a histogram of the image to determine location of lane lines.
Identifying all non zero pixels around histogram peaks using the numpy function `numpy.nonzero()`.
Fitting a polynomial to each lane using the numpy function `numpy.polyfit()`.

After fitting the polynomials I was able to calculate the position of the vehicle with respect to center with the following calculations:
Calculated the average of the x intercepts from each of the two polynomials `position = (rightx_int+leftx_int)/2`
Calculated the distance from center by taking the absolute value of the vehicle position minus the halfway point along the horizontal axis `distance_from_center = abs(image_width/2 - position)`
If the horizontal position of the car was greater than `image_width/2` than the car was considered to be left of center, otherwise right of center.
Finally, the distance from center was converted from pixels to meters by multiplying the number of pixels by `3.7/700`.

The code is present in advanced_Lane_finding_test_2 for a single image only.

## 6. Visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

The code and figure is in the last 2 steps of advanced_Lane_finding_test_2. The overall process is:

Plot the polynomials on to the warped image.
Fill the space between the polynomials to highlight the lane that the car is in .
Use another perspective trasformation to unwarp the image from birds eye back to its original perspective.
Print the distance from center and radius of curvature on to the final annotated image.

## 7 Video processing 

After establishing a pipeline to process still images, the final step was to expand the pipeline to process videos frame-by-frame, to simulate what it would be like to process an image stream in real time on an actual vehicle. 

The video pipeline first checks whether or not the lane was detected in the previous frame. If it was, then it only checks for lane pixels in close proximity to the polynomial calculated in the previous frame. This way, the pipeline does not need to scan the entire image, and the pixels detected have a high confidence of belonging to the lane line because they are based on the location of the lane in the previous frame. 

If at any time, the pipeline fails to detect lane pixels based on the the previous frame, it will go back in to blind search mode and scan the entire binary image for nonzero pixels to represent the lanes.

In order to make the output smooth I have taken the average of the coefficients of the polynomials for each lane line over a span of 10 frames. The code can be found at Adavanced Lane Finding - Application. The ouput video names are mentioned in the code.


