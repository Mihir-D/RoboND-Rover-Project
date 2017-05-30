## Project: Search and Sample Return

---


**The goals / steps of this project are the following:**  

[//]: # (Image References)

[image1]: ./misc/rover_image.jpg
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg 
[video1]: ./output/test_mapping.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  


### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.
`a. For Obstacles:` I observed that whichever pixels were not part of the navigable terrain, are part of obstacles. Also, sample stones were detected as navigable terrain. So for detectin obstacles, I have simply used the negation of the condition used to identify navigable terrain pixels. Refer to function **rock_thresh().

`b. For identifying sample rocks:` The rocks are yellow in color. I found that it is easier to threshold for colors in HSV format. I used the cv2 library. First I converted my image from RGB to BGR. Then I converted it to HSV format and applied thresholding. Refer to function **obstacle_thresh().

![alt text][image1]

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 
1. Defined the source and destination points.
2. Took a perspective transform of the input image.
3. Applied 3 different thresholds to extract 3 b/w images of obstacles, rock samples and navigable terrain.
4. Converted each of the valid pixels of above images to rover centric coordinates.
5. Converted each of these 3 images' rover centric coordinates to real world coordinates with `pix_to_world()`.
6. Updated world_map with red color for obstacle, green for rock sample and blue for navigable terrain.
The video is at [this](./output/test_mapping.mp4) locations

![alt text][image2]
### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.
`1. Perception_step() Updates:`
The required functions as described in `process_image()` function in the Notebook were added. Accordingly, vision_image and world_map is updated. Additionally, distances and angles for Rover's obstacle pixels, rock sample pixels and navigable pixels were added. Obstacle distances are used in `decision.py`
`2. decision_step() Updates:`
The rover keeps to the left side of the wall. If it is very close to the left wall, it turns slightly to right. If it is stuck and doesn't go to 'stop' mode, it turns right. The same loop repeats for each frame.
Detailed description follows below:
In forward mode, following functionalities were added:
The rover should move close to left side of the wall. For this, average angle of navigable terrain to Rover's left, the left side obstacle distances (where obstacle_angle > 0) and the right side obstacle distances (where obstacle_angle < 0) are used.
Next, the rover is preferred to go to left, hence in calculating Rover.steer from mean of navigable angles, 8 degrees are added to each angle so that the average inclination is a bit to left than the actual average navigable angles.
Next, if right side obstacle is closer than left side obstacle and there is no immediate wall on left (checked through nav_left which is average angle of left side navigable pixels) then the rover turns left.
Further if the nav_left is less than certain threshold (i.e. rover is very close to wall), the rover is turned right.
In case the roll and pitch angles are out of certain limit, the rover is stopped and turned and enters into 'stop' mode.
Also, if Rover velocity remains zero for more than one second, then Rover enters 'stuck' mode. In stuck mode, it takes a 4 wheel turn for some time period and then goes back to 'forward' mode.
Sometimes when rover hits a rock, (somehow) it sees enough navigable points through the rock and never goes to 'stop' mode. It used to get stuck there. Also, in some small region in the map the rover doesn't go right even when it is stuck on the left wall.The above condition helps rover come out of these situations.

Note: Some times the rover might cover the same path twice.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.
`Environment details:`
1. Resolution -------------------> 800 * 600
2. Graphics quality -------------> Fantastic
3. FPS o/p ----------------------> 5

**Achievements:

1. The rover does a pretty well job in remaining close to left wall to detect objects. 
2. It detects all the objects it encounters in its path.
2. The rover can navigate more than 95% terrain for different starting positions.
3. For coverage > 90%, I have managed fidelity more than 60%.
**Scope for improvement:

1. In case of very large open area as shown below, it is misguided slightly. In another case as shown below, the rover keeps on hitting the left wall again and again. I think both the issues can be handled well by thresholding the distance of left side obstacle. Also, currently the obstacle pixels contain a wide range as shown below (most of the area which the rover doesn't actually see is mapped red). I should limit the pixels to only close to the rover. This will also improve the wall distance calculation accuracy (as currently there are lot of redundant pixels) and efficiency (higher efficiency if less pixels).
2. I can optimize the if else conditions in decision.py forward mode. Currently, I am doing some unnecessary calculations which can be avoided if proper condition is put before.
3. I can increase the maximum velocity of the rover. Currently, I have not implemented the algorithm to stick close to wall in all cases. Therefore, when I tried increasing velocity to 4, The rover missed some turns which could not be seen through rover camera.
4. I can add a code to pick up the rocks. I am working on it currently.
5. I can add memory to rover so that it does not revisit the same path again.
6. I can save the starting location in the memory and make the rover come back to original position once it has found and picked up all the rocks.
7. I can make rover move smoother putting less harsh conditions on steer and left wall distance whenever possible. For example, if the right wall distance is more than a certain threshold, I can increase the distance from left wall and make the rover move straight for longer distances, thus increasing average speed.

![alt text][image3]


