# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consists of 6 steps. 

1. Grayscaling the input image
2. Applying the Gaussian blur filter on the previously grayscaled image
3. Taking the resulting blurred and grayscaled image and applying the Canny algorithm
4. Defining and applying a mask to mask out the image regions that are not of interest for lane detection
5. Moving the resulting image into Hough space so that a set of lines can be found
6. Drawing the found lines onto the original input image

In order to draw a single line on the left and right lanes, I modified the draw_lines() function. This took quite some experimenting.
I pursued the idea of gathering the lines per lane. The code iterates over all lines present in the (masked) hough line set. Per line, the center point gets calculated.
This center point is used to determine whether the line is part of the left or the right lane. If the point of a line is on the right half of the image, this line is part of the right lane.
If not, it must be part of the left lane. I collect this information for all lines and store the coordinates of the center point(s) in one of two arrays, centers_r(ight) or centers_l(eft).<br/>
At this point I have to sets of points and, per set, I have to find a line that best fits through all these points. Sounds like linear regression to me. With function_from_centers() I - per set of points - use numpy for that. There I split up a point set into a x- and a y- coordinate set. These are handed over to numpy's polyfit, its results (namely slope and shift of a well fitting linear function) are returned. I wasn't aware of this numpy functionality and looked it up on Stackoverflow to be honest. <br/>
Now I had the parameters necessary to draw an infinite line. But cv2.line() required a start and an end point. I set these coordinates in such a way that the lines would be drawn from the image's bottom to roughtly 2/3 of the image's height. I chose the 2/3 ratio through experimenting honestly. With this, I got two coordinates per linear function: a start point and an end point. These then were handed over to cv2.line() in order for actual lines to be drawn.

### 2. Identify potential shortcomings with your current pipeline

The lanes are expected to be exactly within a mask's field of vision. Wider lanes this way might not be well detectable. The draw_lines() function works well only on straight roads. With a curve, the "line center left means line belongs to left lane" approach breaks apart, just as the approach of trying to draw a linear function. Also, the behavior on images without any lanes at all makes the model look helpless. By this I mean, there is no error handling no real robustness to be found. <br/>
Another drawback is the pipeline relying on a lot of hardcoded number values. This is especially the case when transforming into hough space for line detection.

### 3. Suggest possible improvements to your pipeline

The hardcoded number values and the mask's boundaries should be variable for better pipeline performance on wider image ranges. The draw_lines() function should incorporate line drawing for higher degree functions. This would, together with getting rid of the "line center left means line belongs to left lane" approach, make the pipeline more robust. A way for the  pipeline to react to non-lane-containing images should be added. This is critical for real world application, as completely random line drawings otherwise could ... well cause a crash.