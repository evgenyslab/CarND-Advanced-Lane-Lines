## Advanced Lane Finding
---

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

[//]: # (Image References)

[image1]: ./output_images/demo_calibration.png "calibration"
[image1b]: ./output_images/demo_calibration_2.png "calibration"
[image2]: ./output_images/demo_processing_00.png "Processing_0"
[image3]: ./output_images/demo_processing_01.png "Processing_1"
[image4]: ./output_images/demo_warpping_0.png "Warping_0"
[image5]: ./output_images/demo_warpping_1.png "Warping_1"
[image6]: ./output_images/demo_process_wrap_0.png "Process_Warping_0"
[image7]: ./output_images/demo_process_wrap_1.png "Process_Warping_1"
[image8]: ./output_images/demo_pipeline_0.png "Process_pipeline_0"
[image9]: ./output_images/demo_pipeline_1.png "Process_pipeline_1"
[video1]: ./output_images/project_video.mp4 "Video"

## Overview of Code
The complete implementation of the code can be found in the iPython notebook Project.ipynb. The notebook is divided into multiple sections for the reader. Each section is labelled and thoroughly commented for process flow.

The notebook is partitioned in three main sections:
1. Function definitions

    These include camera calibration, generation of warping matrix, image warping, distortion application, image processing for binary lane detection, lane detection pipeline, lane class, and detector class.

2. Video Processing Framework

    This function includes the video processing pipeline using VideoFileClip module.

3. Functional demos

    These function implement examples of the functions developed in section 1.0. They parse one image at a time and save the output.

### Camera Calibration
Camera calibration was completed using the openCV camera calibration functions. The calibration process, outlined in section 1.2, consists of reading a directory of images with an input checkerboard size. The function accepts, jpegs, and png format images. The function loads every image in the directory and attempts to locate checker board corners. To improve detection over the default grey-scale images, the FineCheckerboardCorners functions was used with the openCV flags for adaptive thresholding and image normalization. During testing, these proved to detect more checkerboard corners than simply using grey-scale images. Once the camera was calibrated used all the images, the calibration data was stored as a pickle file. The following image demonstrates the removal of image distortion in a calibration image.

![alt text][image1]

![alt text][image1b]

### Image Processing Pipeline
The image processing pipeline was established through experimentation of the Sobel x-gradient and color thresholding using the HSL color space, Section 1.7. The HSL color space proved to be more robust than RGB for segmentation of color space in dark and light conditions. The final processing pipeline consisted of a sobel x-gradient threshold in the range of (20,100) on the L-channel of the image, followed by the union of two thresholds on S-channel at (170,255), and L-channel at (100,255). This captured the bright white lines, yellow lines, and removed false positives from dark shadows. The result of the processing pipeline is presented in the following images.

![alt text][image2]

![alt text][image3]

### Perspective Transform
The perspective transform function was defined in section 1.6, and the transform itself is applied in Section 1.5. The source points were chosen based on an image analysis of the straight section of road using image editor. An example of perspective warping using source and target points is presented below.

![alt text][image4]

![alt text][image5]

### Lane Detection Pipeline
The lane detection pipeline consisted of a Lane class, a detector class, and a detection method. All of the code was contained within Sections 1.8-1.10.

The detection method was based on image convolution wherein the binary masked image was parsed into the detector, and the following process occurred:
1. initial positions of left and right lanes were determined using peak-response from summing the lower half of the image in both left and right halves.
2. Using the previously found centres, convolutional windows of size (50,80) pixels where used to find the maximum convolutional response in a margin near the previously located centres from the bottom of the image to the top.
3. If any convolution failed to find a maximum value, the previous window centre was used.
4. Once all centres were found for the left and right lanes, the windows were used to recover all 'hot' binary pixels in the window regions for each lane.
5. The left/right pixels were then used to fit a polynomial for each lane.
6. A set of x-pixels were calculated using the numpy polynomial fit tool for a set of all pixels in all left/right windows.
7. A check was made on an input lane object, namely, if a lane object was passed into the detector with previously detected lane x-pixels, the method checked the mean disparity between the current x-pixels, and previous x-pixels. If the disparity was below a threshold of 120 pixels, the current x-pixels were smoothed (80-20%) with previously detected x-pixels. If the current disparity was greater than the threshold, the current pixels were rejected, and previous ones used.
8. Once the pixels were filtered and smoothed, they were inversely mapped to the original image, and the output image plotted a polygon region within the lanes.
9. The radius of curvature was calculated using:

R = ((1+(2Ay+B)^2)^(3/2))/|2A|


An example of the output is presented below:

![alt text][image6]

![alt text][image8]

![alt text][image7]

![alt text][image9]

### Complete pipeline with video

A video was produced using the pipeline, and is available [here](/output_images/project_video.mp4).


### Discussion
Overall, the method performed fairly well. It was noted that a smoothing and lane-pixel rejection method was necessary to remove several false-positives encountered during the video. Specifically, these occurred during very light road color, and neighbouring car passing the right lane.

Further investigation should include improve image processing for lane detection, and improved filtering for image tracking.
