# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_videos_output/last_frame_gray.jpg "Grayscale"
[image2]: ./test_videos_output/last_frame_edges.jpg "Edges"
[image3]: ./test_videos_output/last_frame_medges.jpg "Masked Edges"
[image4]: ./test_videos_output/last_frame_lines.jpg "Hough Lines"
[image5]: ./test_videos_output/last_frame_avgs.jpg "Processed and Averaged"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps:

1) Converted the images to grayscale, and applied a mild gaussian smoothing to reduce the chance of noise being caught
    as legitimate edges
    
    ![alt text][image1]
    
2) Used Canny edge finding to detect edges in the image
    
    ![alt text][image2]
    
3) Applied a region mask to roughly isolate where the lane lines would be (at least, reducing unnecessary processing
    of sky/roadside/other noise in subsequent steps)
    
    ![alt text][image3]
    
4) From the edges remaining after masking, detected Hough lines. Parameters were purposely set to find more lines vs
    trying to isolate only "the best" lines in one operation.
    
    ![alt text][image4]
    
5) From the Hough lines, any lines "too flat" were ruled out as noise/artifacts, and straight vertical lines had to
    be ruled out as well (unfortunately) due to the effect they have on averaging functions. The remainder were
    separated into left vs. right based on slope (since lane lines should converge at a point in the distance).
    Further culling of lines is done by finding the average length of segments, and trusting only those longer than
    average (a crude, but quick way of filtering out short outliers). With this done, the remaining segments in the
    "left" and "right" line containers are averaged respectively to generate an average left and average right lane
    line.
    
    ![alt text][image5]
    
    Modifying the draw_lines() function to handle the averaging and display functionality seems kind of crude
    (and required adding parameters that probably shouldn't be passed to it in a production application... 
    namely a region height parameter and a slope checking parameter), but without changing the template of the
    project this is all I could do.


### 2. Identify potential shortcomings with your current pipeline


As the code currently stands, it cannot handle a couple situations.

1) When lane lines are shaded/faded/disappear even to the human eye. This pipeline has no "memory" of past frames
    and no "sense of heading" that could be used to guess where future "lane line pixels" will be. This would 
    come in handy for the challenge.mp4 video.
    
2) Lane changes would break the left/right assumptions that we made regarding the lane lines, so during the time
    that either 1 or 3 lane lines are visible, drawing Hough lines would be fine, but drawing average left/right
    lane lines would fail.

3) The code may not break per se if the road is sharply curved, but drawing straight "average" lane lines on a
    in-reality curvy road would be very inaccurate and probably dangerous for a driving computer.


### 3. Suggest possible improvements to your pipeline


One improvement needed to this pipeline is an implementation of "past" line marker position. Since the vehicle
cannot turn instantaneously, knowing past "lane line pixels" will add additional points to reference when finding
new "lane line pixels" via the pipeline we implemented here, improving robustness. Without delving into object 
tracking, one such way to make line memory is to pass the (X2,Y2) of the average-left/average-right lines from 
the previous frame, into the processing of the current frame, as a weighted addition to the line averaging process.
Another method could be to use the average lane lines from the last frame as a basis for the region mask in the
current frame - biasing the code to look for lane lines in the same area that they were found 1/30th of a second
ago, which is a decent assumption for a car.

Although I can't imagine how to implement it yet, it would be better to connect line segments end-to-end to
generate a lane line vs. averaging the lane line candidates we find. Something like:

1) Only average/combine line segments that exist in the same space (X1/X2/Y1/Y2 are close)
2) Find segments that are close to co-linear (slope and intercept close) but exist in different parts of
    the image
3) Extend these segments toward each other to draw a polygonal spline approximating the lane line

This would allow us to draw a lane line that follows a curved road, especially dotted lines which the current
pipeline isn't super great at dealing with.



PS. Note to grader - sorry that this is late AND I didn't do the challenge, but right now I really just want to
catch up to the rest of the class. I didn't get a chance to actually start Udactiy until I wrapped up my business's
operations earlier last week. Thank you for your understanding!