## Project: Search and Sample Return

---


**The goals / steps of this project are the following:**  

[//]: # (Image References)

[image1]: ./misc/rover_image.jpg
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg 

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  


### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
a. For Obstacles, I observed that whichever pixels were not part of the navigable terrain, are part of obstacles. Also, sample stones were detected as navigable terrain. So for detectin obstacles, I have simply used the negation of the condition used to identify navigable terrain pixels. Refer to function **rock_thresh().

b. For identifying sample rocks: The rocks are yellow in color. I found that it is easier to threshold for colors in HSV format. I used the cv2 library. First I converted my image from RGB to BGR. Then I converted it to HSV format and applied thresholding. Refer to function **obstacle_thresh().
![alt text][image1]

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
1. Defined the source and destination points.
2. Took a perspective transform of the input image.
3. Applied 3 different thresholds to extract 3 b/w images of obstacles, rock samples and navigable terrain.
4. Converted each of the valid pixels of above images to rover centric coordinates.
5. Converted each of these 3 images' rover centric coordinates to real world coordinates with `pix_to_world()`.
6. Updated world_map with red color for obstacle, green for rock sample and blue for navigable terrain.
The video is location at `put in the video location`
![alt text][image2]
### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.
1. Perception_step() Updates:
The required functions as described in `process_image()` function in the Notebook were added. Accordingly, vision_image and world_map is updated. Additionally, distances and angles for Rover's obstacle pixels, rock sample pixels and navigable pixels were added. Obstacle distances are used in `decision.py`
2. decision_step() Updates:
In short, the rover keeps to the left side of the wall. If it is very close to the left wall, it turns slightly to right. If it is stuck and doesn't go to 'stop' mode, it turns right. The same loop repeats for each frame.
Detailed description follows below:
In forward mode, following functionalities were added:
The rover should move close to left side of the wall. For this, average angle of navigable terrain to Rover's left, the left side obstacle distances (where obstacle_angle > 0) and the right side obstacle distances (where obstacle_angle < 0) are used.
Next, the rover is preferred to go to left, hence in calculating Rover.steer from mean of navigable angles, 8 degrees are added to each angle so that the average inclination is a bit to left than the actual average navigable angles.
Next, if right side obstacle is closer than left side obstacle and there is no immediate wall on left (checked through nav_left which is average angle of left side navigable pixels) then the rover turns left.
Further if the nav_left is less than certain threshold (i.e. rover is very close to wall), the rover is turned right.
In case the roll and pitch angles are out of certain limit, the rover is stopped and turned and enters into 'stop' mode.
Also, if Rover velocity remains zero for more than one second, then Rover enters 'stuck' mode. In stuck mode, it takes a 4 wheel turn for some time period and then goes back to 'forward' mode.
Sometimes when rover hits a rock, (somehow) it sees enough navigable points through the rock and never goes to 'stop' mode. It kind of used to hang there. Also, in some small region in the map the rover doesn't go right even when it is stuck on the left wall.The above condition helps rover come out of these situations.

Note: Some times the rover might cover the same path twice.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  



![alt text][image3]


