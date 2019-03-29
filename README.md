# CarND-Path-Planning-Project-P1

Udacity Self-Driving Car Nanodegree - Term 3

Path Planning Project by David Sosa

# Introduction

A path planning algorithm is implemented to drive a car on a highway on a simulator provided by Udacity([here](https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2)). 

The simulator sends car telemetry information (car's position and velocity) and sensor fusion information about the rest of the cars in the highway (Ex. car id, velocity, position). It expects a set of points spaced in time at 0.02 seconds representing the car's trajectory. 

The communication between the simulator and the path planner is done using [WebSocket](https://en.wikipedia.org/wiki/WebSocket). The path planner uses the [uWebSockets](https://github.com/uNetworking/uWebSockets) WebSocket implementation to handle this communication. Udacity provides a seed project to start from on this project ([here](https://github.com/udacity/CarND-Path-Planning-Project)).

# Prerequisites

The project has the following dependencies (from Udacity's seed project):

- cmake >= 3.5
- make >= 4.1
- gcc/g++ >= 5.4
- libuv 1.12.0
- Udacity's simulator.

For instructions on how to install these components on different operating systems, please, visit [Udacity's seed project](https://github.com/udacity/CarND-Path-Planning-Project). This implementation carried in Ubuntu 18.10.

In order to install the necessary libraries, use the [install-ubuntu.sh](./install-ubuntu.sh).

# Compiling and executing the project

In order to build the project there is a `./build.sh` script on the repo root. It will create the `./build` directory and compile de code. This is an example of the output of this script:

Now the path planner is running and listening on port 4567 for messages from the simulator. Next step is to open Udacity's simulator:

![Simulator first screen](images/simulator.png)

# [Rubic](https://review.udacity.com/#!/rubrics/1020/view) points

## Compilation

### The code compiles correctly.
Cheked

## Valid trajectories
## Walkthrough

### Line

### The car is able to drive at least 4.32 miles without incident.
The simulation was run for 10 miles without any incidents as shown in the picture below.

![15 miles](images/15_miles.png)

### The car drives according to the speed limit.
Speed limit violation was not shown.

### Max Acceleration and Jerk are not Exceeded.
Maximum jerk red message was not shown.

### Car does not have collisions.
No collisions in th

### The car stays in its lane, except for the time between changing lanes.
The car stays in its lane most of the time but when it changes lane because of traffic or to return to the center lane.

### The car is able to change lanes
The car is able to change lanes when a slower car is ahead. This is done safely and meeting the requirements of confort given by the jerk and the acceleration limits. 
## Code Analysis

The code consist of three main parts:

- perceptions and prediction phase,
- behaviour phase,
- path planning i.e. safe and and confortable trajectory generation. 

### Perception and Prediction [line 251 to line 290](./src/main.cpp#L51)
Data coming from the sensor fusion modules are used to perceive if there are cars ahead of the ego-vehicle (our vehicle) and in the same lane. 

 In the case, we want to know three aspects of it:

- Is there a car in front of us blocking the traffic.
- Is there a car to the right of us making a lane change not safe.
- Is there a car to the left of us making a lane change not safe.

These questions are answered by calculating the lane each other car is and the position it will be at the end of the last plan trajectory. A car is considered "dangerous" when its distance to our car is less than 30 meters in front or behind us.

### Behavior [line 292 to line 314](./src/main.cpp#L293)
This part decides what to do:
  - If we have a car in front of us, do we change lanes?
  - Do we speed up or slow down?

Based on the prediction of the situation we are in, this code increases the speed, decrease speed, or make a lane change when it is safe. Instead of increasing the speed at this part of the code, a `speed_diff` is created to be used for speed changes when generating the trajectory in the last part of the code. This approach makes the car more responsive acting faster to changing situations like a car in front of it trying to apply breaks to cause a collision.

### Trajectory [line 326 to line 416](./src/main.cpp#L313)
This code does the calculation of the trajectory based on the speed and lane output from the behavior, car coordinates and past path points.

First, the last two points of the previous trajectory are added to the points_x, points_y vectors. If there are less than two points in the trajectory (line 326), then the current position and the previous position (taken from the car's yaw). If there are more than two points in the trajectory (line 338) then, those two last points are added to the trajectory and a reference yaw is calculated for future use.
 
Now we create three points with a distance of 30, 60 and 90 meters along the s coordinated (lines 357 - 367) and push them as well to the points_x, points_y vectors, totalling 5 points. These 5 points will be used to create the spline which then the vehicle will follow 
 
To make the math simpler for the spline calculation, the coordinates are transformed (shift and rotation) to local car coordinates (lines 370 to 377).

In order to ensure more continuity on the trajectory (in addition to adding the last two point of the pass trajectory to the spline adjustment), the previous points are added to the new trajectory (lines 374 to 379). It is important to understand that previous generated points still lie ahead of the car. Previous comes from previously generated and not previous to the car.  

## Spacing the spline points at a desired speed

The rest of the points are calculated by evaluating the spline and transforming the output coordinates to not local coordinates (lines 388 to 407). Worth noticing the change in the velocity of the car from line 393 to 398. The speed change is decided on the behavior part of the code, but it is used in that part to increase/decrease speed on every trajectory points instead of doing it for the complete trajectory.
