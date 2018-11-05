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
[image1]: ./camera_cal/calibration1.jpg "Calibration Image"
[image2]: ./output_images/undistorted_chessboard.jpg "Undistorted Chessboard Image"
[image3]: ./test_images/test2.jpg "Test Image"
[image4]: ./output_images/undistorted_image.jpg "Undistorted Image"
[image5]: ./output_images/distorted_undistorted_overlapped.jpg "Overlapped Image"
[image6]: ./output_images/h-channel.jpg "H-Channel"
[image7]: ./output_images/l-channel.jpg "L-Channel"
[image8]: ./output_images/s-channel.jpg "S-Channel"
[image9]: ./output_images/threshold-x.png "Sobel operation on image over x-axis"
[image10]: ./output_images/threshold-y.png "Sobel operation on image over y-axis"
[image11]: ./output_images/threshold-magnitude.png "Magnitude of the gradient"
[image12]: ./output_images/threshold-direction.png "Direction of the gradient"
[image13]: ./output_images/threshold-s-channel.png "Threshold of the s-channel image"
[image14]: ./output_images/threshold-combined.png "Combined threshold"
[image15]: ./output_images/straight_lane_calibration.png "Straight lane calibration"
[image16]: ./output_images/calibration_road_warped.png "Calibration road warped"
[image17]: ./output_images/straight_lane_overlay_on_curved.png "Overlay straight lane on curved road"
[image18]: ./output_images/warped_lane.png "Warped Lane"
[image19]: ./output_images/sobel-x.png "Threshold image from Sobel operation along x-axis on the road image"
[image20]: ./output_images/sobel-y.png "Threshold image from Sobel operation along y-axis on the road image"
[image21]: ./output_images/sobel-magnitude.png "Threshold image from magnitude of the gradient on the road image"
[image22]: ./output_images/sobel-direction.png "Threshold image from direction of the gradient on the road image"
[image23]: ./output_images/s-channel-binary.png "Threshold image from s-channel of the road image"
[image24]: ./output_images/combined-threshold.png "Combined threshold of above threshold images"

### Setup

##### Link to Jupyter Notebook
- ![Jupyter notebook][./Advanced_Lane_Lines.ipynb]
- I have reused much of the boilerplate code from quizzes where available, with some tweaks in places.

### 1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
#### 1.1 Computation of camera calibration matrix and distortion coefficients.

The code comprises of two methods:  
- get_distortion_vars(): This function steps through the list of calibration images, and finds their object points and image points to perform camera calibration.
- undistort(image): This function takes an image, and undistorts it using the calibration attributes returned by the above function.

#### 1.2. Test for image distortion correction on chessboard image
In this section, I have used the functions in code section 1.1 to calibrate and undistort one of the calibration image.  
##### Calibration Image:
![alt text][image1]

##### Undistorted Chessboard Image:
![alt text][image2]

### 2. Apply distortion correction to raw images.
In this section, I have used the functions from Code section 1 to undistort an image of a road image.  

##### Original Image
I have used the image `/test_images/test2.jpg` for the purpose of testing the functionalities below.  
![alt text][image3]

##### Undistorted Image
![alt text][image4]

##### Overlapped Image
The difference between the original and undistorted images isn't quite evident when seen separately. So I have created an overlapped image.  
![alt text][image5]

### 3. Use color transforms, gradients, etc., to create a thresholded binary image.
#### 3.1 HLS and Color Thresholds
I have splitted the image into H, L and S channels to check which one depicts the lane more prominently.  
##### H-Channel
![alt text][image6]
##### L-Channel
![alt text][image7]
##### S-Channel
![alt text][image8]

We see that the lanes are more prominent on the S-channel. In the sections below, we will perform further operations on the S-channel image for lane detection.

#### 3.2 Threshold codes taken from Course resources

##### Sobel operator applied along the x-axis
![alt text][image9]

##### Sobel operator applied along the y-axis
![alt text][image10]

##### Magnitude of the gradient
![alt text][image11]

##### Direction of the gradient
![alt text][image12]

##### Threshold image of the S-channel
![alt text][image13]

##### Combined threshold
![alt text][image14]

In the quizzes, the combined threshold was generated as:  
``` combined[((gradx == 1) & (grady == 1)) | ((mag_binary == 1) & (dir_binary == 1))] = 1 ```

However, I observed that the s-channel provided a pretty good and strong signal about the lanes. So, I have factored that in during the generation of combined threshold.
``` combined[((gradx == 1) & (grady == 1)) | ((mag_binary == 1) & (dir_binary == 1)) | (s_channel_binary == 1)] = 1 ```

### 4. Apply a perspective transform to rectify binary image ("birds-eye view").
#### 4.1 Finding street endpoints from a straight road for perspective transform
I calibrated for straight road using the provide image `straight_lines_2.jpg` and traced lines over the lanes to obtain the endpoints for use in subsequent steps.

| Corner        | Source coordinate                   |
|---------------|-------------------------------------|
| Top Left      | (595, 450)                          |
| Top Right     | (690, 450)                          |
| Bottom Right  | (1115, 720)                         |
| Bottom Left   | (216, 720)                          |

##### Calibration image with lane markers
![alt text][image15]

##### Perspective transformation of the image over the four endpoints (obtained from above)
![alt text][image16]

#### 4.2 Using the four coordinates from above to perform perspective image on other road image
Firstly, I overlapped the corners obtained from straight lane calibration over the test image.  
![alt text][image17]

Then I warped the image along the four coordinates. The red lines here (and below) serve as a reference to the straight lanes obtained from straight lane calibration above. 
![alt text][image18]

I then performed a series of threshold operations on the warped image.

##### Threshold image obtained by applying Sobel operator along x-axis on the road image
![alt text][image19]

##### Threshold image obtained by applying Sobel operator along y-axis on the road image
![alt text][image20]

##### Threshold image from magnitude of the gradient on the road image
![alt text][image21]

##### Threshold image from direction of the gradient on the road image
![alt text][image22]

##### Threshold image from s-channel of the road image
![alt text][image23]

##### Combined threshold of above threshold images
![alt text][image24]
