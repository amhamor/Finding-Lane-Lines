# **Finding Lane Lines on the Road** 
---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road.
* Reflect on my work in a written report.


[//]: # (Image References)

[image1]: ./test_images_output/solidYellowCurve_grayscale.jpg "solidYellowCurve_grayscale.jpg"
[image2]: ./test_images_output/solidYellowCurve_gaussian_blur.jpg "solidYellowCurve_gaussian_blur.jpg"
[image3]: ./test_images_output/solidYellowCurve_canny_edge_detection.jpg "solidYellowCurve_canny_edge_detection.jpg"
[image4]: ./test_images_output/solidYellowCurve_mask_polygon.jpg "solidYellowCurve_mask_polygon.jpg"
[image5]: ./test_images_output/solidYellowCurve_masked_canny_image.jpg "solidYellowCurve_masked_canny_image.jpg"
[image6]: ./test_images_output/solidYellowCurve_masked_lines_hough_transform.jpg "solidYellowCurve_masked_lines_hough_transform.jpg"
[image7]: ./test_images_output/solidYellowCurve_drawn_lines_image.jpg "solidYellowCurve_drawn_lines_image.jpg"

---

### Reflection

### 1. Pipeline desciption:

My pipeline consists of five steps: 

1. Convert the input image to grayscale. This simplifies the next step (blurring the image) by making the edges of the images blend together depending on how bright or dark each pixel is. ![alt text][image1]
2. Blur the grayscale image using cv2.GaussianBlur(src, ksize, sigmaX). This further blends bright and dark pixels together depending on the kernel size used, allowing for some level of control in managing which edges become detected. ![alt text][image2]
3. Detect edges from the blurred image using cv2.Canny(image, edges, threshold1, threshold2). The number of edges detected can depend on how blurry the blurred image is. The more blurry that the image is, the less difference in pixel values from one pixel to the next there will tend to be. To detect edges using Canny Edge detection, the lower threshold will need to be lower the blurrier the image is to get closer to detecting that edge. ![alt text][image3]
4. Create a black image then draw a filled polygon on this black image. This filled polygon will be used to mask the area that is relevant for detecting lane lines. ![alt text][image4]
5. Use cv2.bitwise_and(src1, src2) to blacken any pixels in the Canny Edge detected image outside of where the filled polygon from Step 4 is located. ![alt text][image5]
6. Use cv2.HoughLinesP(img, rho, theta, threshold, lines, minLineLength, maxLineGap) to detect edges from the masked Canny Edge detected image in Step 5 that meet five criteria:
	1) The perpendicular line to the edge is rho pixels from the origin.
	2) The perpendicular line to the edge is a number of radians from the horizontal axis that is a multiple of theta radians.
	3) The number of pixels that comprise the edge is at least a threshold amount.
	4) The number of consecutive pixels that comprise the edge is at least a minLineLength amount.
	5) The number of non-consecutive pixels between two edges that follow the path of the same line is no more than a maxLineGap amount.

![alt text][image6]
   
For this pipeline, detected edges were constrained only to the threshold and maxLineGap parameters due to being able to use rho and minLineLength each at one pixel and theta at pi/180 radians (or one degree). Having a large maxLineGap increases the tendency to detect lines from one lane line to the other while a small maxLineGap tends to limit the number of lines detected within a dashed lane line. A large threshold limits the number of edges detected in general while a small threshold will detect marks in the road, tires from nearby vehicles, shadows, etc...
7. Draw the masked image with lines detected from a Hough Transform from Step 6 over the original image. ![alt text][image7]

I modified the draw_lines() function in the following ways:

To detect only lane lines, introduced into this step are two constraints:
	
1) For a line to be a left lane line, the slope of the line must be between -0.50 and -1.00.
2) For a line to be a right lane line, the slope of the line must be between 0.35 and 0.85.
   
To center the left and right lines used to represent detected left and right lane lines, the average of all slopes and x1 coordinates for each of the left and right lane lines within each image is used as the slope and x1 value to draw a single line representing each left and right lane line. A single point from a detected line is used to extrapolate to the bottom of the image then to use that point at the bottom of the image as the bottom of the lane line. Using the average slope and average x1 coordinate, the entire lane line to the top of the filled polygon from Step 4 is extrapolated.


### Potential shortcomings with the current pipeline:


There are two definite shortcomings to this pipeline: 

1) One definite shortcoming is that bright gray pavement blends in with yellow lane lines to the point where this pipeline cannot detect the yellow lane line without loosening the constraints too much. These yellow lines can be detected when the HoughLinesP threshold is lowered to 5, but this also causes five to ten non-lane-lines to be detected as well. This causes the drawn lane lines to be significantly inaccurate as the x1-coordinate values from these edges are outliers that significantly deviate the average x1-coordinate value from the x1-coordinate that the actual lane line is located.
2) Another definite shortcoming is that the edges of yellow or white vehicles that move within the filled polygon can become detected as part of a lane line. This can have the same effect occur as in the previous shortcoming.

At intersections, this pipeline does not seem capable of detecting lane lines as most of the lane lines would probably be located outside of the filled polygon. Also, faded lane lines would probably tend to be undetected as this pipeline currently cannot detect the yellow lane line in the part of pavement with sun shining on it within the challenge.mp4 video.

The ability of this pipeline to detect curved lane lines probably depends on the amount of curvature that exists within these curved lane lines. Using the average x1-coordinate and slope values as mentioned before seems would successfully detect slightly curved lane lines to a reasonable degree of accuracy. However, lane lanes curving beyond maybe 10 degrees would probably start showing significant mis-matching between the drawn lane lines and actual lane lines due to the drawn lane lines being contrained to a straight line.


### 3. Pipeline Improvement Suggestions:

This pipeline can be improved by giving it the ability to classify objects. The proportion of total variations of each object learned by this pipeline would determine how well lane lines (and other objects) are detected without using any algorithmic instructions and therefore eliminate the shortcomings mentioned above.

Without the ability to classify objects, this pipeline could be improved by introducing a degree 2 polynomial equation to drawn lane lines. This would allow the detection of curved lane lines. Another possible improvement would be to average the pixel values of the detected lane lines and use a small deviation from that average as a range of allowable values that detect the lane lines. Any pixels that are outside of this range would not be considered a lane line and might eliminate the issue of detecting the yellow lane line in the sunshine within the challenge.mp4 video.
