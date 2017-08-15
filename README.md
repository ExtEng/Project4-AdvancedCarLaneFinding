## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

In this project, my goal was to use the techniques and code examples presented in the Udacity course to calculate smooth car lanes that folow the (possibly cuves) roads.  

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
[image9]: ./output_images/search_window.jpg "Search window approach to find the pixels that represent the car lanes"


The Following Describes my methadology to find the car lanes. 

# Step 1: Camera Calibration

Using the Camera Calibration Images provided I calculated the Calbration matrix. The detailed steps are:
1. Prepare a matrix of objects points, these provide a framework of the pattern's coordinate space.
2. Prepare the Calibration images, i.e. read the images and convert to grayscale.
3. Find the chessboard corners withtin the each image using the OpenCV function "cv2.findChessboardCorners" and the expected number of corners (for our project 9 X 6)
4. If the cordinates are found store the points (for all images)
5. Then finally calculate the "mtx & dist" paramters that allow us to use cv2.undistort to remove effects of the intrinsic parameters of the lens. 

I've included the image bellow of a test sample of the undistortion of a calibration image.

![alt text][image1]

# Step 2: Load Test Images and Apply undistortion

Using the test images, perform a quick check of the previous Step. The undistortion is apparent around the outter edges.
![alt text][image2]
![alt text][image3]

# Step 3: Color transformations

First, I set up all the possible color trandformations, describe in class  and utilized in previous excersies. 
Then using the recomended techniques, if tested with color transformations provided the best visual representation of the car lanes. This part was alot of trial and error to discover which transformation povided the best results.

After much deliberation, I decided to utilize the "S" & "L" channels from HSL color space and the grayscale image to capture both the yellow and white lanes. Even though, I found values that yielded good results, I think better accuracy can be achieved by systematically testing the variety of parameters. The S channel was threshold at a min of 120 and max of 255, L channel was threshold at a min of 215 and max of 255 and the grayscale was threshold at min of 200 and max of 255.  

The last step to the color transformations was to clean up the image by applying a ROI mask, similar to that of Project 1. 

I've included images of the Binary image after selective thresholding and masking. 

![alt text][image4]
![alt text][image5]

# Step 4: Spatial Transformation

Using the OpenCV functionns, cv2.getPerspectiveTransform and cv2.warpPerspective, we were able to calculate a warping Matrix M to apply a 
apply a geometric transformation to get the "bird's eye" view of the car lane ahead. Utilizing the test images, and the images of the straight lanes (labelled test 8 and test 9) i was able to derrive the coordinates of the four courners to transform the image to the desired view. The Intial points were tuned by trail and error to get the desired affects.

I have included the images of the of the following step.

![alt text][image6]
![alt text][image7]

# Step 5: Find Lanes

In this stage, for each test image I calculated and displayed the lane using the following steps.
1. Calculate the historgram along the X direction (using the bottom half of the warped image)
2. Then split the image in two sections, left and rigth, and find the max value which corresponds to the general x location of the car lane (left or rigth). See Below for an example of the folowing step.

![alt text][image8]

3. Then Using a window search parameter, with a plus and minus bounds of 100 pixel, store all the non zero pixel indices.
4. If the min non zero pixels condition is met then you can ajust the midpoint of the next window by the x coordinate of the mean nonzero value. This shifts the search window left of rigth depending on the curve of the road. See Below for a illustration

![alt text][image9]

5. Using all the nonzero values found for each lane, calculate a 2nd order polonomial to fit these points. With This equation, the lane is represented by a smooth curve or straight line. Once this line is calculated, it can be used to find the lane in the next image, because there will never be a scenario where the lane change so drastically between image frames (assuming a normal acquition frequency). I 

![alt text][image10]






