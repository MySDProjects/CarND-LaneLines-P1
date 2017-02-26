#**Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of the following steps:
* A color filter. I converted the image to the HLS color space and constructed separate filters for white (a high pass filter on Luminosity) and yellow (Hue between 20 - 50, high saturation, high luminosity). Those filters were combined into a single mask, which was applied to the original image using a bitwise_and. The resulting images contained only white and yellow pixels.

[image1]: ./writeup_images/color_filter.jpg "Color Filter"

* A conversion to grayscale to prepare the image for Canny Edge detection. Edge detection is based on image gradients, so it is necessary to convert the image to a single channel for this to work well.

[image2]: ./writeup_images/gray.jpg "Gray Scale"

* Filtering out the region of interest to reduce computational load, and restrict the image processing to only the region of the image where the lane lines nearest the car should lie. This entailed removing left and right portions of the image, and roughly the top half of the image, corresponding to the sky.

[image3]: ./writeup_images/region.jpg "Region of Interest"

* Smoothing the image using a Gaussian Blur (7 pixels), and applying Canny Edge detection. I set the low threshold for Canny to a difference of 120, and the high to double that, at 240. 

[image4]: ./writeup_images/edges.jpg "Canny Edge Detection"

* Applying the hough transform to the edge image. I experimented with different settings for min line length, max line gap, and thresholds, and settled on (20,20,30) respectively. At this point the image can be annotated with every line segment detected in the image using the weighted_img function provided in the setup code.

[image5]: ./writeup_images/hough.jpg "Hough Lines"

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by:
  * assuming the left lane line would have the steepest negative slope, and the right lane line the steepest positive slope
      * on the challenge video I had issues with noisy vertical line detections, so I threw out any near-to-vertical lines.
  * I iterated over each line, if it had a negative slope it became a candidate left lane line, and I checked whether it was     in the left hand side of the image. I then checked whether its slope was in the range of other candidate left lane           lines, and if so grouped them together. If its slope was steeper than the previous candidate left lane lines, I threw       them away and used this new line as the new basis for grouping left lines.
  * I applied the same logic to the right lane.
  * After iterating over all the lines, I had a set of left and right lane line candidates with very similar slopes.
  * I used a least-squares line fit to find the best fitting line to all the left points, and all the right points.
  * I then found the intersection of these lines (around the center of the image). I also found the intersection of both the     left and right lines with the bottom of the image (y=number of rows).
  * I drew lines from an offset (50 pixels) back from the intersection point on each line to the bottom of the image.

[image5]: ./writeup_images/final.jpg "Final Output"

###2. Identify potential shortcomings with your current pipeline

There are a number of shortcomings with the current pipeline:
* It's tailored to straight lines. The performance around curves looks "jittery".
* It's tuned to particular image resolutions, evidenced by the hard-coded thresholds in the region of interest detection.
* It's tuned using a very narrow range of lighting conditions. I had to adjust the color filter in order to get any results on the challenge video
* I've put thresholds on line slopes that mean vertical lines are not considered. This means that it wouldn't work well should the car change lanes.

###3. Suggest possible improvements to your pipeline
* I added the threshold on vertical lines to deal with issues I was having on the challenge video with vertical lines being detected on the white car. One simple improvement here would be to do frame-to-frame filtering and remove moving parts of the image. Or remove "blobs" of white and yellow so that white and yellow cars do not supply candidate lines.
* Instead of fitting straight lines, I could fit a curved line.
* I could use a Kalman filter approach to smooth the detection of lines frame-to-frame, and remove some of the jitter.

