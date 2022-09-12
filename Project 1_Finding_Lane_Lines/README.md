# Project 1: Finding Lane Lines on the Road

---


### **Finding Lane Lines on the Road**
This project focused on developing a pipeline using computer vision methods utilized from the OpenCV library for Python. 


[//]: # (Image References)

![Figure1](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/test_images/solidYellowCurve.jpg)

---

## 1.0 Description

This project can be use for still images of 960 x 540 pixels as well as video string. Below is one of the provided images.

![Figure2](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/test_images/solidWhiteCurve.jpg)


### Step 1: Convert RGB (Red, Green, Blue) Image to Grayscale

The provided images are in `RGB colorspace`. 

To prepress image, we can use greyscale. In a greyscale image, every pixel is assigned a value from 0–255 for example to represent its intensity. Put in another way, a greyscale image is a normal image without the information of color.Canny edge image is constructed from the greyscale image. The intensity values in the greyscale will be used to calculate the gradient of the image. Then, these calculated gradient wil be used to determine which pixel belongs to the edges in the image based on a threshold value. The pixels lying on the edges will be assigned value 1 while the others will have value 0.

Grayscale is more computationally efficient but when we believe color provides meaningful signal to a model and colors are fairly similar in their appearance, we should be cautious to use grayscale. If you want to distinguish yellow line form white line, greyscale might not be the best choice. However, the purpose of this project is just for lane detection, so we can still use greyscale. 

#### Some other useful preprocessing technique (optional, form [silverwhere](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/README.md))
Continuing, I achieved a decent result with RGB to Grayscale conversion, but I had also learned about HSV and HSL colorspace when developing a traffic light classifier In the "Intro to Self Driving Cars Nanodegree" to detect specific colors of a traffic light. At this point I am only concerned with detecting a line, not the specific color, so HSV (Hue, Saturation, Value) or HSL (Hue, Saturation, Lightness) colorspace will prove to be the best filter to detect the lines themselves. Utilizing HSL I am better able to detect the lines as "lightness" is best understood as the amount of white in the pixel. HSV provided a better result than grayscale, however, when looking at the results of HSV compared to HSL, I found better detection and proceeded to continue with HSL for accuracy.

![Figure3](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/test_pipeline_images/gray_white_lanes.jpg)     
*Grayscale*

![Figure4](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/test_pipeline_images/hsv_white_lanes.jpg)  
*HSV*

![Figure5](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/test_pipeline_images/hls_white_lanes.jpg)  
*HLS*     

### Step 2: Apply a Gaussian Blur Filter for Smoothing of Lane Lines 

In image processing, a Gaussian blur (also known as Gaussian smoothing) is an image pre-processing technique. It is the result of blurring an image by a Gaussian function (named after mathematician and scientist Carl Friedrich Gauss). The effect is typically to reduce image noise and reduce detail.<p align="center">

![Figure6](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/test_pipeline_images/gaussian_blur.jpg)  
*Gaussian Blur*

*Note that a Canny Filter which we will also use has a 5 x 5 Gaussian Blur built inside, but adding one before is for additional smoothing.

### STEP 3: Apply a Canny Edge Detector  

A Canny Edge Detector is an edge detection operator. This is useful for us as since we have already identified the regions of lightness and smoothed the image in pre-processing, the Canny Edge Detector can detect and edge with a low error rate, which means that the detection should accurately catch as many edges as shown in the image as possible.

![Figure7](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/test_pipeline_images/canny_edge.jpg)  

*Canny Edge Detector Output*

### STEP 4: Create a Masked Image of our Canny Edge Output

The output of the Canny Edge Detector is an image "edges", which is made of dots. (black canvas with dotted outliner) To make it accurate, we can applied a region of interest utilizing a polygon combined with the masked image. The result is a Canny Edge Output image "masked_edges".

![Figure8](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/test_pipeline_images/region_of_interest.jpg) 

*Region of Interest*

The OpenCV implementation requires passing in two parameters in addition to our blurred image, a low and high threshold which determines whether to include a given edge or not. A threshold captures the intensity of change of a given point (you can think of it as a gradient). Any point beyond the high threshold will be included in our resulting image, while points between the threshold values will only be included if they are next to edges beyond our high threshold. Edges that are below our low threshold are discarded. Recommended low:high threshold ratios are 1:3 or 1:2. We use values 50 and 150 respectively for low and high thresholds.<p align="center">

![Figure9](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/test_pipeline_images/masked_canny.jpg)  

*Region of Interest with Masked Canny Detector Output*

#### STEP 5: Perform a Hough Transform 

The next step is to apply the Hough Transform technique to extract lines and color them. A Hough Transform finds best fit lines from all points. This is done by converting our current Cartesian system denoted by axis (x,y) to a parametric one where axes are (m (slope), b (y-intercept).
In parameter space this plane:  

1. lines are represented as points
2. points are presented as lines (since infinite number of lines can pass one point)
3. intersecting lines between two lines in Hough space means a line connecting two points in regular coordinate system 
4. we can find best fit line by find the most intersect section in Hough space
5. the slop of a line in a regular coordinate system is infinite when the line is vertical. We can't have that so we use polar system. In polar coordinates, a given line will now be expressed as (ρ, θ), where line L is reachable by going a distance ρ at angle θ from the origin, thus meeting the perpendicular L; that is ρ = x cos θ + y sin θ.  

![Figure10](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/hough_transform_diagram.jpg)

All straight lines going through a given point will correspond to a sinusoidal curve in the (ρ, θ) plane. Therefore, a set of points on the same straight line in Cartesian space will yield sinusoids that cross at the point (ρ, θ). This naturally means that the problem of detecting points on a line in cartesian space is reduced to finding intersecting sinusoids in Hough space.<p align="center">

![Figure11](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/test_pipeline_images/hough_lines.jpg)  
*Hough Lines*

#### STEP 6: Separate Left and Right Lanes / Draw Lines

The program we have now can detect the edge of the lane. But for some dotted lane, we need to connect them into a longer line. To solve this, we can use the fact that dotted segments have same slope. draw_lines() function can be modified by first calculating the slope of the detected line coordinates. For slopes greater or less than 0.3 I appended the coordinates of the identified left or right lane lines. Slopes between -0.3 and 0.3 were ignored as edges close to zero did not appear as a lane line and often detected edges that were not lanes in certain frames of the video.
  
Now that we know our left and right lanes, we can draw single, solid, red lines that trace the lane line through the entire region of interest. To accomplish this we need to determine the X-coordinates of the bottom of the line and the top of the line to be traced. Y-coordinates were already determined as the y-coordinates of the region of interest, 540 (bottom) and 350 (top) pixels. To draw the lines we used our x and y coordinates with the CV2.line function to draw our solid red lines in red.<p align="center">

![Figure12](https://github.com/silverwhere/Self-Driving-Car-Nanodegree---Udacity/blob/main/Project%201%20-%20Finding%20Lane%20Lines/test_pipeline_images/solidWhiteCurve.jpg)
