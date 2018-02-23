# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps. First, I converted the images to grayscale, then I applied gaussian filter followed by canny
edge detector. I applied a Region of Interest mask to the edges where the ROI is a triangle over the bottom half of the image.
Next i passed the output to openCV's probabilitistic hough transform function which returned line segments. Finally, the lines and original image are blended together using weighted_img helper function provided.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by separating the line segments returned from hough transform into 2 buckets using their gradient - negative gradients are for left lane while positive gradients are for right lane. Next, since all line segments in each bucket should have approximately the same gradient, I removed outliers by taking the average gradient and removing line segments with gradient > 1 standard deviation away from the mean. Lastly, since all we need is 1 line for each lane, I used least squares regression to draw a best fit line using all the points that are left over from the line segments.

For the video, in order to remove outliers and to smooth the output, we can make use of the observation that the gradient of the lines should not change significantly from frame to frame. Thus, we keep track of a global running average and if the delta for the current frame is too much, we discard the results as an anomaly and show the last correctly updated frame, otherwise, we update the last correctly updated frame and return it.

If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when the car is driving along a sharp curve. The linear regression model only works when approximating a straight line. The model breaks down when trying to approximate a curved road.

Another shortcoming could be false positives when other traffic lane markings or random streaks on the ground within the Region of Interest gets picked up as line segments, which subsequently throws the line fitting off.

Another shortcoming is the top of the Region of Interest is currently hardcoded to be 320. This is a very important value as going any higher towards the top of the image means going above the horizon, which increases noise tremendously, as was the case for the challenge video.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to only pick up lines that are yellow or white in colour.

Another potential improvement could be to filter out gradients that are definitely out of range, given the normal range of gradients we normally would expect for this task.

Yet another potential improvement would be to dynamically figure out the horizon level in the image so as to set a good point for the top cutoff point for the region of interest. Update this value periodically - e.g. every 30 frames.

One way to figure out the horizon would be to use the same hough transform technique to identify all parallel lines on the ground, extend all lines to the ends of the image. We make use of the knowledge that all parallel lines intersect at vanishing point under perspective projection to arrive at the location of the horizon. Set the top of the ROI to be somewhere below this value.
