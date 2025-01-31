## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

In this project, my goal was to use the techniques and code examples presented in the Udacity course to calculate smooth car lanes that follow the (possibly curved) roads.  

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

[image1]: ./output_images/calibration_test.jpg "Test of Undistortion"
[image2]: ./output_images/camera_org.jpg "Test of Undistortion - Original Image"
[image3]: ./output_images/camera_und.jpg "Test of Undistortion - Undistorted Image"
[image4]: ./output_images/S_Gray_thres.jpg "S Channel & GrayScale Threshold"
[image5]: ./output_images/S_Gray_thres_mask.jpg "S Channel & GrayScale Threshold with ROI mask"
[image6]: ./output_images/B_eye_org.jpg "Warped Imaged - Color"
[image7]: ./output_images/B_eye_bin.jpg "Warped Imaged - with Color preprocessing"
[image8]: ./output_images/hist.jpg "Histogram of the bottom half of the Warped-Binary-Masked Image"
[image9]: ./output_images/window.jpg "Search window approach to find the pixels that represent the car lanes"
[image10]: ./output_images/S_window.jpg "Faster search window"
[image11]: ./output_images/test3.jpg "Final result"
[image12]: ./output_images/final_result.jpg "Final result - with lane details"
[image13]: ./output_images/Delta_fit.jpg "The Change of the left lane's polynomial coefficients as a function of image sequence"

The Following Describes my methodology to find the car lanes. 

# Step 1: Camera Calibration

Using the Camera Calibration Images provided I calculated the Calibration matrix. The detailed steps are:
1. Prepare a matrix of objects points, these provide a framework of the pattern's coordinate space.
2. Prepare the Calibration images, i.e. read the images and convert to grayscale.
3. Find the chessboard corners within each image using the OpenCV function "cv2.findChessboardCorners" and the expected number of corners (for our project 9 X 6)
4. If the coordinates are found store the points (for all images)
5. Then finally calculate the "mtx & dist" parameters that allow us to use cv2.undistort to remove effects of the intrinsic parameters of the lens. 

I've included the image bellow of a test sample of the undistortion of a calibration image.

![alt text][image1]

# Step 2: Load Test Images and Apply undistortion

Using the test images, perform a quick check of the previous Step. The undistortion is apparent around the outer edges.
![alt text][image2]
![alt text][image3]

# Step 3: Color transformations

First, I set up all the possible color transformations, describe in class and utilized in previous exercises. 
Then using the recommended techniques, if tested with color transformations provided the best visual representation of the car lanes. This part was a lot of trial and error to discover which transformation provided the best results.

After much deliberation, I decided to utilize the "S" & "L" channels from HSL color space and the grayscale image to capture both the yellow and white lanes. Even though, I found values that yielded good results, I think better accuracy can be achieved by systematically testing the variety of parameters. The S channel was threshold at a min of 140 and max of 255, L channel was threshold at a min of 215 and max of 255 and the grayscale was threshold at min of 200 and max of 255.  

The last step to the color transformations was to clean up the image by applying a ROI mask, like that of Project 1. 

I've included images of the Binary image after selective thresholding and masking. 

![alt text][image4]
![alt text][image5]

# Step 4: Spatial Transformation

Using the OpenCV functions, cv2.getPerspectiveTransform and cv2.warpPerspective, we were able to calculate a warping Matrix M to apply a 
apply a geometric transformation to get the "bird's eye" view of the car lane ahead. Utilizing the test images, and the images of the straight lanes (labelled test 8 and test 9) I was able to derive the coordinates of the four corners to transform the image to the desired view. The Initial points were tuned by trail and error to get the desired affects.

I have included the images of the of the following step.

![alt text][image6]
![alt text][image7]

# Step 5: Find Lanes

In this stage, for each test image I calculated and displayed the lane using the following steps.
1. Calculate the histogram along the X direction (using the bottom half of the warped image)
2. Then split the image in two sections, left and right, and find the max value which corresponds to the general x location of the car lane (left or right). See Below for an example of the following step.

![alt text][image8]

3. Then Using a window search parameter, with a plus and minus bounds of 100 pixel, store all the non zero pixel indices.
4. If the min non zero pixels condition is met then you can adjust the midpoint of the next window by the x coordinate of the mean nonzero value. This shifts the search window left of right depending on the curve of the road. See Below for an illustration

![alt text][image9]

5. Using all the nonzero values found for each lane, calculate a 2nd order polynomial to fit these points. With This equation, the lane is represented by a smooth curve or straight line. Once this line is calculated, it can be used to find the lane in the next image, because there will never be a scenario where the lane change so drastically between image frames (assuming a normal acquisition frequency). I 

![alt text][image10]

6. Then we can apply the lanes and create a filled polygon that will map onto our original image, using OpenCV libraries and the fittest lines extracted from the polyfit regression algorithm. Then using the Inverse “warp” matrix, I applied green car lane onto the original image. See Below for the final result of the lane finding.

![alt text][image11]

7. After finding the lanes, as measure of accuracy and a variable that can be used to reject measurement errors, I calculated the radius of curvature. Using the formulas presented in class and the pixel-to-world calibration factors [y- 30 meters to 720 pixels; x - 3.7 meters to 450 pixels], I could calculate the average radius of curvature. The calibration factors were found to heavily dependent of the selection 4 quadrant points that determined the matrix used to warp the image to/and from the "bird's eye" view. The number was found to be reasonable in the order of magnitude, but inaccuracies are apparent. When comparing the left and right radius of curvature, assuming very accurate measurement and a curved lane [radius of curvature --> infinity for straight lines, the difference in the value should be the width of the lane (3.7 meters), which was not the case in my measurements. 

8. The Second part was to calculate the deviation of the car's center to the lane center, Hint hint, useful for path planning or PID steering control. Since the most accurate part of the polynomial fit for each lane is closest to the car, I utilized the two points that represent the base of each fitted line [where y = 719 px] to calculate the midpoint of the lane. Then assuming the car's camera is mounted directly center of the car calculated the difference. See image below for the result. 

![alt text][image12]


# Step 6: Video Pipeline

Using the Function process_pipeline, I was able to change parse the threshold minimum value for the S channel and the warp perspective  matrix (requires a different one for the challenge videos, from my first attempts). 

In the Pipeline the methodology stays the same as previous described. Except, the window approach is used on the first image and only when a lane is not found. Otherwise, using the Line() class, the previous polynomial (that was stored) is used to specify the next search window. This class is also used to store other important values such as flags [ "lane".detected] or values to be used for computations, such as the last n iterations of polynomial coefficients, which are used to perform a rolling average (low pass filter). From the result video, thee lane tracking does a good job. There is a small segment that the end of the lane deviates a little. This can be seen in the output of the delta of the polynomial coefficients shown below, the delta changes dramatically. As a note of future work, after calculating the bounds of expected polynomials for car lanes, these values can be used for false lane rejected 

![alt text][image13]

The final video for submission is labelled "Output_project_video.mp4"

# Discussion

I think overall the pipeline performed well, but there are many areas of improvement. First the color/image preprocessing while yielding good results is still susceptible to error, from road lines (not lanes) that could be interpreted as lanes, and depending on lighting conditions, the performance could degrade severely. An Example, is the white horizontal strips (perpendicular to car lanes) placed on roads when drivers are required to slow down. 

The 4 Quadrant points used to warp the image, was a little tricky to get perfect, while I arrived at a reasonable result, I think this could have been improved, which also, makes me wonder that if I mask away the far away lane, would I yield better results. Essentially the further part of the lane is stretched and warped (which has to decrease its real world accuracy), but during the polynomial regression fit , those pixels hold the same weight. Also depending on the road how its shaped, curved hill side versus highway, the warped perspective will in all likely hood fail to get the proper transformation.

The radius of curvature, was a challenge to perfect. Mainly due to the y pixel to world ratio. While x was reasonable to get, the y was only able to estimate based on either (lane stripe length -  3.048m) or the approximated total length. Ideally, if these values could be measured accurately, false lane rejections during lane transitions would be a stich using the radius of curvatures.


