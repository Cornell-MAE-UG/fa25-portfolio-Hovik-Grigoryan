---
layout: project
title: Mechatronics Final Project - Autonomous Robot
order: 1
description: Robot 
technologies: [C, Fusion 360, Arduino, 3d printing]
image: /assets/images/3780robot.JPG
---

For my MAE3780 - Mechatronics course in Cornell, me and two teammates created a robot from skratch as the final project for the course. The robot's task was to collect cubes within its perimiter. The robot competed with other robots in a square field with black borders, where the robots would start at opposite ends of the field, then try to collect as many small wooden cubes as possible within a minute. The robot with more cubes is the winner. The robot was fully autonomous, and its movement relied on the color sensor reading, without any external commands. 

<img src="{{ 'assets/images/field.PNG' | relative_url }}" alt="Render 1" width="550">

Then robot had two motors which were driving the two wheels, a color sensor which was used to detect the black borders as well as a color change, which would indicate that it is near the center of the field. The code ran on an Arduino icrocontroller. An important part of the competition was the fact that no internal Ardino commands were allowed for use, only code written with C. 
For the cube collection, our robot had large wheels and 3d-printed arms which were guiding the cubes under the robot and storing them there. The robot also had a small cardboard box attached to it on the side, in order to collect even more cubes.

<img src="{{ 'assets/images/handles.PNG' | relative_url }}" alt="Render 1" width="250">
<img src="{{ 'assets/images/robot2.JPG' | relative_url }}" alt="Render 1" width="300">

The movement strategy for the robot was the following:
- Drive forward untill you detect a color change
- If color change occurs, and the new color is black (meaning the robot hit the border of the field), turn right by 180 degrees and drive forward
- If color change occurs, and the new color is not black (meaning that the robot crossed the half-field line), turn right by 45 degrees and drive forward

Within the team, my personal contributions included the following:
- Building the body of the robot and wiring all the components
- Writing the part of the code responsible for controlling the motors
- Creating CADs for the arms of the robot and 3d-printing them
- Participating in all the stages of planning of the robot