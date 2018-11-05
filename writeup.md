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
[image25]: ./output_images/sliding-windows.png "Sliding Windows"
[image26]: ./output_images/lane-warped.png "A new layer with polyfilled lane on warped image"
[image27]: ./output_images/lane-unwarped.png "Unwarped image with lane polyfilled on it"
[image28]: ./output_images/lane-overlay.png "Lane overlaid on the actual image"
[image29]: ./output_images/pixels-to-meters.png "Pixels to meters"
[image30]: ./output_images/text-overlay.png "Text overlay"

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

We'll be using the combined threshold image for lane detection in the subsequent steps.

### 5. Detect lane pixels and fit to find the lane boundary.
#### 5.1 Find the lane pixels, and fit polynomial

We are using sliding window approach for lane detection. Here is the result:  

##### Fitted lines with sliding windows
![alt text][image25]

I have then used the coordinates obtained from sliding windows to polyfill the lane on the warped image.  
![alt text][image26]

The layer is then unwarped back to the real world perspective.  
![alt text][image27]

The unwarped layer is then overlaid on top of the real image to get the final image.
![alt text][image28]

### 6. Determine the curvature of the lane and vehicle position with respect to center.
#### 6.1 Determine lane curvature
Here, I am simply using the formula from the lesson to calculate lane radius based from the position (y-coordinate on the image), and the coefficients of the second-degree curves representing the lanes.

#### 6.2. Determining conversions in x and y from pixels space to meters
Here, we are using the unwarped lane image for the straight road as a reference to map dimensions from pixels to meters.  

![alt text][image29]

**Here are some calculations:**
- Dashed line width (in px): 81
- Standard length of dashed line (in meters): 3.0
- meters per pixel in y dimension:  0.037037037037037035
- Lane width in px: 720.0
- Standard width of lanes (in meters): 3.7
- meters per pixel in x dimension:  0.005138888888888889

Hence,  
- meters/pixel along x-axis: 0.0051  
- meters/pixel along y-axis: 0.0370

#### 6.3 Determining 1. lane curvature in pixels and meters  2. vehicle offset
Here, I have written a few helper functions for calculation of lane curvature and vehicle offset in pixels, and to convert it to meters scale.  

For calculation of offset, I have compared the offset at the bottom of the images between:  
1. The center of the lane obtained from the image used to find end coordinates of a straight road.
2. The mid-point of the detected lane.

I then used the multiplication factor (meters/pixel along x-axis: 0.0051) to find the offset in meters.

#### 7. Warp the detected lane boundaries back onto the original image.
I have done it in step #6

#### 6. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.
Here's how it finally looks like after overlaying the lane and displaying the lane curvature and vehicle offset from the center of the road:  

![alt text][image30]

## PIPELINE
Here's the code for the pipeline, with some comments added to explain each step:  

```python
def process_image(img):
    # Undistorting the image frame
    undistorted = undistort(img)

    # Warping the image based on the four corners obtained beforehand.
    pts = endpoints_for_perspective_transform()
    warped, M, dst = warp(undistorted, pts)

    # Thresholding on the warped image. The function get_thresholds() returns multiple
    # thresholds:
    # 1. Gradient along x-axis
    # 2. Gradient along y-axis
    # 3. Magnitude of the gradients
    # 4. Direction of the gradients
    # 5. Threshold from S-channel of HLS
    # 6. Combined threshold from the above.
    gradx, grady, mag_binary, dir_binary, s_channel_binary, combined = get_thresholds(warped)
    binary_warped = combined
    
    # Curve fitting on the binary, warped image frame.
    # out_img = image with the output of curve fitting
    # left_fit = coefficients of second degree polynomial obtained from curve fitting for the left lane
    # right_fit = coefficients of second degree polynomial obtained from curve fitting for the right lane
    # left_fitx = points on the second degree curve obtained from curve fitting for the left lane
    # right_fitx = points on the second degree curve obtained from curve fitting for the right lane
    out_img, left_fit, right_fit, left_fitx, right_fitx, ploty = fit_polynomial(binary_warped)
    
    # Create a blank layer with the lane polyfilled w.r.t. the warped image.
    layer = np.zeros_like(undistorted)
    coordinates_left = np.vstack((left_fitx.astype(int), ploty.astype(int))).T
    coordinates_right = np.flipud(np.vstack((right_fitx.astype(int), ploty.astype(int))).T)
    coordinates = np.concatenate((coordinates_left, coordinates_right), axis=0)
    cv2.fillPoly(layer, np.array([ coordinates ]), color=(0, 255, 0))
    
    # Unwarping the above image (polyfilled lane) back to real-world perspective.
    unwarped, M, dst = unwarp(layer, pts)

    # Overlay the polyfilled layer on the original image.
    overlay = weighted_img(unwarped, undistorted, 0.95, 0.6, 0)
    
    # Overlay text on the image frame
    text = get_overlay_text(binary_warped, left_fit, right_fit)
    overlay_text(overlay, text)
    
    car_offset_text = find_car_offset_text(img, left_fit, right_fit)
    overlay_text(overlay, car_offset_text, pos=(50, 100))

    # Return the composite image (with the polyfilled image overlaid on original image)
    return overlay
```

