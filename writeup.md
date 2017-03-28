# Finding Lane Lines on the Road

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

[//]: # (Image References)

[image1]: ./test_images_output/1_greyscale.jpg "Grayscale"
[image2]: ./test_images_output/2_blur.jpg
[image3]: ./test_images_output/3_canny.jpg
[image4]: ./test_images_output/4_region-of-interest.jpg
[image5]: ./test_images_output/5_lane-segments-all.jpg
[image6]: ./test_images_output/6_lane-segments-classified.jpg
[image7]: ./test_images_output/7_lane-lines.jpg
[image8]: ./test_images_output/8_original-with-lane-lines.jpg


---

## Reflection

### Pipeline description

My pipeline consists of the following steps:
- convert to grey scale
- apply gaussian blur
- canny transformation
- clip all but relevant lane region
- determine hough lines as lane segments
- classify lane segments into left and right
- extrapolate lane segments to lane line
- average lane line using sliding window

First, I create a grey scale copy of the original image.

![Grayscale][image1]


Second, I apply gaussian blur to the grey scale image to
smooth pixel gradients a bit.

![Blur][image2]

Then I apply the canny transformation to find edges.

![Canny][image3]

Then I clip all but the possible region for lane markings in
driving direction.

![Clip][image4]

Then I use the the hough algorithm to determine line segments
in the canny edges.

![Hough][image5]

Then using the line segments I derive a single line for left and right
lane. I first separate the lines segments into left and right lane
by using the segment slope as a classifier. Lines that are not relevant
to be part of a lane marking due to slope e.g. horizontal lanes are
excluded (function lines_split_left_right).

![Left Right Segments][image6]

Then for each lane, I determine the upper boundary of the lane
which is the lowest y value in all line segments (function lines_y_top).
Then I use weighted averaging of slope and top x position
using the individual segment lengths as weight (long lane marking
segments are more relevant). Using the average slope and top lane
position I derive the single lane line (function lines_segment_to_lane).

![Lanes][image7]

Then I use a sliding window for left and right lane line to average
the lane line to draw (function lane_window_average).
This helps with false positive lane segments.

Finally I combine the lanes with the original image.

![Lanes][image8]


### Short comings

Potential shortcomings are for example:
- A) hard coded region of interest
- B) false positive lane markings
- C) sharp curves
- D) fast steering movements

#### A) hard coded region of interest:
If the camera position or aspect ratio changes either relevant lane
markings are not detected or irrelevant lane markings of other lanes /
other edges are detected as false positives.

#### B) false positive lane markings:
Edges in other objects than lane markings could be falsely detected
as lane markings e.g. scratches in the road, structures in trees,
fences, etc.

#### C) sharp curves:
In curves with a low curve radius the classification of
left and right lane marking could fail resulting in seeing both lane
markings as left or right lane.

#### D) fast steering movements
Due to using a sliding window for lane line averaging
fast steering movements and thus lateral movements of lane lines
are detected with a delay.


### Possible improvements

A major improvement would be to apply a color filter using
opencv's inRange function for white and yellow color. This would
reduce the amount of false positive lane marking detection and
help to reduce the size of the sliding window for averaging lane lines.
This color filter could be applied to the original image
in order to produce a mask that is used on the canny result in order
to reduce the canny output to actual lane markings.

Another improvement should remove the hard coded area of interest
for lane markings but instead cluster all found hough lines
(after applying color filter on canny output) in a more
elaborate manner. Find clusters of hough lines that together form
a straight or curved line. Then identify the immediate left and right lane
in those clusters by looking at the lower (highest y value) of those
lane lines / curves.
