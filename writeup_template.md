## Project: Search and Sample Return
**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook).
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands.
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./misc/rover_image.jpg
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg
[isorock]: ./output/isolated_rock.jpg
[hsvrock]: ./output/rock_hsv.jpg
[newthresh]: ./output/3chan_threshed.jpg
[results]: ./output/results.png

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

I modified the `color_thresh()` function such that opposed to returning a binary image it returned a ![3 channel RGB imagine][newthresh] in which the red channel contained the location of the obstacles, the green channel contained the location/detection of rocks, and the blue channel contained the location of navigable terrain.

To detect obstacles I took the logical not of the mask which contained the position of navigable terrain.

To detect rock samples I took the image being analyzed and converted it from RGB to ![HSV][hsvrock] using OpenCV as it is apparently easier to detect a specific color in HSV. I then utilized upper and lower HSV values as a filter to create a mask containing the yellow images. I determined this upper and lower HSV values by analyzing the image of a rock in hsv. this mask was then ![applied to the original image][isorock] and the location of the yellow rocks assigned to the green channel of the output image.


#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result.

I followed the provided hints/instructions to fill out `process_image()`. I did not define the source and destination pairs in the function as they were previously defined in the jupyter notebook. the perspective transform and color thresholding was then applied to the recorded image. As I mentioned previously, the color thresholding function was modified to output a 3 channel image containing the locations of navigable terrain, obstacles, and rocks. The pixel locations for each channel of the color thresholded image were extracted, converted to rover coordinates and subsequently converted to world coordinates using the function `pix_to_world()`. To match the world map coordinates, the x and y pixel values had to be swapped when assigned to the worldmap. The world coordinates for the obstacles were assigned to the first channel of the worldmap, the rock coordinates the second coordinate, and the navigable terrain to the third channel. Every time a pixel on the worldmap was identified as either an obstacle, rock, or navigable terrain the appropriate channel for that pixel was incremented by 1.

The movie applied to my test dataset can be found in the output folder. See the movie 'test_mapping.mp4'


### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

the `perception_step()` was filled in to match the provided instructions/hints in the comments. It closely resembled that of the `process_image()` function in the jupyter notebook. Source and destination pairs were assigned enabling the perspective transform to be performed. This warped image was then color thresholded returning a 3 channel thresholded image containing the pixel locations of the obstacles in the first channel, the location of the rocks in the second channel, and the location of navigable terrain in the third channel.
These pixel locations were then assigned to corresponding channels of the Rover.vision_image attribute. the pixel locations of the obstacles, rocks, and navigable terrain were then converted to rover centric coordinates and those rover centric coordinates converted to world coordinates using the pix_to_world function. these world positions were then assigned to the appropriate Rover.worldmap channel with the x and y arrays switched to match the provided ground truth worldmap. The rover centric navigable terrain was then converted to polar coordinates and assigned to the appropriate attribute of the Rover class.

The `decision_step()` function was reviewed and utilized as provided to determine if modifications would be made. After observing the behavior of the rover, it was determined that the logic of `decision_step()` were adequate if certain parameters controlling that logic were modified in the Rover object. Specifically, the Rover.stop_forward attribute was increased and the Rover.go_forward attribute was decreased to create a tighter range of available forward space which would initiate acceleration or deceleration, essentially making the rovers forward momentum dependent on a tighter range of values which were more "realistic" in making decisions to go forward or stop. This modification was found to be sufficient to meet the 40% environment mapping at 60% fidelity.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

Simulator Settings:
    - Resolution: 1280 x 1024
    - Graphics Quality: Good
    - FPS during automation: 25-30 FPS

![Simulation Results][results]

As mentioned previously the rover's self.stop_forward was increased and self.go_forward was decreased creating a tighter range of potential navigable terrain which dictates if the rover will accelerate or decelerate, allowing it to coast more freely in the vicinity of walls without getting too close.

There is still a control issue in large open spaces that are circular, it has the ability to get stuck. I would potentially modify the convert to polar coordinates function such that it clips all potential navigable terrain beyond a certain distance away from the rover's camera as the perspective transform becomes more imprecise further away from the rovers camera. fidelity mapping could also be increased by adding logic to check the pitch and roll values of the rover before
assigning the interpreted image to the world map. The decision logic on what vector to navigate to could also be modified to include integral control thus taking into account the previous navigation angles dampening the observed behavior of the robot wiggling left and right when traveling through a wide open space.
