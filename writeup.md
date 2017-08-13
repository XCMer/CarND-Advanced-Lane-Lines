**Advanced Lane Finding Project**

The jupyter notebook "Advanced Pipeline" has been divided into sections. These sections roughly correspond to the sections in this writeup. All code references can be found under the same heading in the jupyter notebook.

## 1. Camera Calibration

Let's look at the images that we have for camera calibration:

![](output_images/distorted_images.jpg)

Using cv2's `findChessboardCorners` and `calibrateCamera` (mapping image points to object points), we get the camera matrix and distortion coefficient.

## 2. Distortion Correction

By using cv2's `undistort` function and the coefficients obtained so far, the undistorted chessboards now look like:

![](output_images/undistorted_images.jpg)

## 3. Color/gradient threshold

This is the place where we apply the Sobel graident(s) and color thresholding.

### 3.a Sobel gradient

We apply both the Sobel x and y gradient, and only keep pixels appearing in both of them. A few examples of this transformation are below:

![](output_images/sobel1.jpg)
![](output_images/sobel2.jpg)
![](output_images/sobel3.jpg)

### 3.b Color thresholding

I started by trying HLS thresholding. While trying to improve the results, I switched to HSV. The transformation looks like the following:

![](output_images/hsv1.jpg)
![](output_images/hsv2.jpg)

### 3.c Combined thresholding

The combined thresholding is done as follows. If a pixel appears in either the sobel, or in the color thresholding, then it shows up in the final output. This is how the transformation looks like:

![](output_images/combined1.jpg)
![](output_images/combined2.jpg)

## 4. Perspective transform

This step was done manually. I wrote a couple of helper functions that allowed me to visualize my results. I observed that since the top two points are near to each other, they were very sensitive to the pixel values. Even a single pixel difference showed up significantly in the final warped image.

The source and destination points are captured inside of the `perspective_before_points` and the `perspective_after_points` functions. This is how the points look:

![](output_images/perspective_points.jpg)

And a sample transform looks like:

![](output_images/perspective1.jpg)

## 5. Lane detection

After the thresholding is applied, and the transformation done, we get on to lane detection.

### 5.a Preparing the image for lane detection

This is where we run steps 1 through 4 mentioned above. Apart from that, we remove pixels 0 to 100 to the left, and 1200 onwards to the right. This is to just remove more noise.

These are the images we get up till this step:

![](output_images/pipeline_stage1a.jpg)
![](output_images/pipeline_stage1b.jpg)
![](output_images/pipeline_stage1c.jpg)

### 5.b Find lane bases in histogram

We take the bottom half of the image (so that a possible curvature towards the top doesn't influence the base detection), and draw out a histogram. The lanes would correspond to the spikes seen in the histogram. We take the center of the image, and pick the x positions that has the maximum value on either sides. These two peaks are assumed to be the two lanes.

![](output_images/straight_lines2_hist.jpg)
![](output_images/problematic2_hist.jpg)

The vertical lines below mark the detected bases.

![](output_images/straight_lines2_base.jpg)
![](output_images/problematic4_base.jpg)

### 5.c Find lane boxes and fit line

Starting from detected bases at the bottom, we do a targeted search with a window of width 200. The height of the window is chosen such that we get 9 vertical windows. At each iteration, we move the window towards the mean of white pixels, i.e., towards the concentration of pixels of the lane. This helps the window move with the curvature of the lane.

After that, we only select the white pixels within these windows, and fit a 2nd degree polynomial through it. This gives us our lane line.

![](output_images/straight_lines1_lane.jpg)
![](output_images/problematic3_lane.jpg)

## 6. Measuring curvature

We measure the curvature using differentiation. I'm assuming 30 metres per 720 pixels along the y axis, and 3.7 metres per 700 pixels along the x axis. For the curved image, `test1.jpg`, we get a curvature of about 1km for the left lane, and 2km for the right. The original curvature as per the project was about 1km. So these numbers look right.

For the distance from the center, we know the center of the image. We now calculate the central x positions of both the lanes. We then subtract the two. The magnitude, multiplied by xm_per_pix, gives the deviation from center. If it's negative, then it's to the left, else to the right.

## 7. Drawing the lanes

We first draw the lane on the warped image, and then apply an inverse perspective transform by just reversing the source and destination points. Lane detection applied to all of the test images look like:

![](output_images/test_output.jpg)

## Final video

The `project_video_output.mp4` file shows the pipeline being applied to the project video.

## Discussion

### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline can likely fail in:
1. Extreme curvature like in the challenge video
2. Surrounding vehicles or interfering with the detection

The main problem I faced was with tuning the various thresholds, especially making it robust against different lighting conditions. I'm yet to figure out how to make the pipeline work under different weather conditions. Would that still require manual adjustment? Can it be automated?
