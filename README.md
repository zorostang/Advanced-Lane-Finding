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

## Criteria
#### Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.
The first cell in the Jupyter notebook contain the camera calibration code. I started by looking for the 9 x 6 chessboard corners in each of the images. 3 of the images did not contain all 9 x 6 corners so those 3 images were skipped. For the other images, the points of the detected corners were saved and used to compute the camera calibration and distortion coefficients.
The matrix and distortion coeffients computed with `cv2.calibrateCamera()` and are saved and used in the `cv2.undistort()` function on the test image. An example of distortion corrected calibration image is as follows:
[![Distortion Corrected]](output_images/undistorted_chessboard.jpg)
####Provide an example of a distortion-corrected image.
An example of a distortion corrected image is below. More distortion corrected images can be viewed in the `output_images` directory
[![Distortion Corrected]](output_images/test1.jpg)
####Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.
I used a combination of color and gradient thresholding. I defined two functions to detect yellow lines and white lines named, `find_yellow_lines()` and `find_white_lines()`, respectively. The goal was to first use color and gradient thresholding on yellow and white lines seperatly. These functions make use of opencv's sobelx operator for gradient thresholding. I also experimented with different color spaces and color channels. I found that the Y channel in YUV worked well with white lines and the the V channel in YUV worked well with the yellow lines.

I combined the binary results in the `find_lines()` function. I used `mag_thresh()` to create a larger surface area of detected pixels in order to make line detection more robust. An example of a binary thresholded image is below. Note, I was not concerned with how these functions affected areas of the image outside the lanes because later on the pipeline we will perform a persepective transform on the image.
[![binary thresh]](output_images/binary_straight_lines1.jpg)
####Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
I defined a function named `get_perspective_transform_matrix()` to get the perspective transform matrix that will be used on the rest of the images in the pipeline. I chose hardcoded source and destination points similar to ones given in the example writeup. I used the camera calibration matrix from the initial camera calibration step.

The function `warp()` takes the perspective transform matrix to perform the perspective transform. I verified that my perspective transform works by looking at a test image and verifiying that the transformed lane lines appear pretty close to parallel. For both straight and curved lane lines.
[![perspective transform]](output_images/perspective_transform.jpg)
####Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
I defined one functions called `find_lane_lines()` which performs a blind search for the lane lines. It uses the sliding window method. My window size is 100 pixels. 

I defined another function called `find_lane_lines_from_previous()` which performs a targeted search for the lane lines given the previous left and right fitted polynomials from the results of the blind search.
[![lane lines]](output_images/discovered_lane_lines.jpg)
####Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
I calculated the the radius of curvature at the end of the `find_lane_lines_from_previous()` function. If I had more time I would refactor this section of code outside of this function.

I calculted the vehicle's position with respect to center in the `Line()` class' method named `__calc_vehicle_offset()`. This method assumes a lane width of 3.7 meters. First I calculate the distance of the center of the car from the left lane line and convert that value to meters. Then I subtract that result from half the lane width. A positive result means the vehicle is left from center.

####Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
[![lane]](output_images/detected_lane.jpg)
####Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!)
Please see the video titled `project_video_out.mp4`

#### Reflections
The biggest challenges in this project were as follows:
* Time spent tuning the color and gradient thresholding algorithms
* Implementing the sliding window search (Thank you Udacity for the starter code)

I decided to exagerate the detected lines in with my use of  gradient magnitude line detection. My pipeline might detect outliers more easily in this case and therefore it would be a good idea to better identify and remove outliers. The most simple thing I could try first is reducing the margin in the search window. Right now, it's set at 100px. I could probably tighten it to around 50.

My pipeline seems to completly fail on the challenge video! I never tested a frame from that video during the development of my gradient and color thresholding tests. I'm very surprised. Perhaps I picked thresholds that were too specific for my test images. I believe the cloudy weather and grayish pavement must be greatly affecting all my thresholding chioces. I can somewhat confirm this because my pipeline correctly detects the lane in the beginning of the harder challenge video (which has sunny weather and darker pavement).

I think bigger areas for my pipeline are in how filter and store data frame by frame. For example, I could implement sanity checks against the calculated curves and offsets. I could smooth my measurment by taking the mean of the past n meansurements, etc.
