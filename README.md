# Gary Holness
# CarND-Path-Planning-Project
Self-Driving Car Engineer Nanodegree Program
   
### Simulator.
I downloaded and used Term3 Simulator which contains the Path Planning Project from the [releases tab (https://github.com/udacity/self-driving-car-sim/releases/tag/T3_v1.2).


### Baseline Code
I started with the Path Planning project repository by forking it and making subsequent changes to
my own version of the repository.  The Path Planning Project repository from which I began
is [https://github.com/udacity/CarND-Path-Planning-Project]

### Approach
I watched the Q&A video provided with the class, followed along and implemented the imcremental changes
as described.  These were

1. Have the car drive straight over a 50-step trajectory

2. Using Frenet coordinates, have the car drive along a series of line segments along the
   middle of the the center lane.  This was an improvement but had high jerk because there
   were discontinuities between line segments.

3. Implement sampling of points along a lookahead spline. This provided low jerk trajectories.
   Moreover the samples were done such that the waypoints were spaced apart to remain consistent
   with a reference velocity of 49.5 mph.

4. Implemented a simple lane change to complement the spline lookahead that leaves the car fixed driving
   in the left lane after the lane change from the center lane to the left lane.  This was done
   in response to being too close to the car ahead in the center lane.

5. Implemented the driving logic structured as a finite state machine.  A key component of this
   was to implement a sensor fusion checking routine that can determine if a car is too close in
   a parameterized lane.  This is used for the preparation for lane change in addition to
   changing back to a target lane for implementation of a passing maneuver.


### Goals
In this project the goal was to safely navigate around a virtual highway with other traffic that is driving +-10 MPH of the 50 MPH speed limit. The simulation environment provides the car's localization and sensor fusion data, there is also a sparse map list of waypoints around the highway. 

The car should try to go as close as possible to the 50 MPH speed limit, which means passing slower traffic when possible, note that other cars will try to change lanes too. The car should avoid hitting other cars at all cost as well as driving inside of the marked road lanes at all times, unless going from one lane to another. The car should be able to make one complete loop around the 6946m highway. Since the car is trying to go 50 MPH, it should take a little over 5 minutes to complete 1 loop. Also the car should not experience total acceleration over 10 m/s^2 and jerk that is greater than 10 m/s^3.

## Results

The structure of the decisionmaking within my
path planner takes form as a Finite State Machine
that keeps safe distance, tests whether it is
safe to change lanes to the left or right, and
executes the left or right lane change.

I have implemented two version of the Path Planner

1) Target Lane Return version
   This version returns back to the target lane (lane 1)
   after passing a slower vehicles.  It is accomplished 
   by comparing the target lane to the current lane.
   This version implements the passing maneuver.
 
2) Lane Change Version
   This version changes lanes to get past slower moving
   vehicles, but does not return to the target lane.
 
 
   The code is setup using preprocessor MACROS to select 
   the Target Lane Return version of the path planner.
 
   To select the Lane Change Version, locate the
   preprocessor macro "DO_TARGET_LANE_RETURN" and set it
   to  "#define DO_TARGET_LANE_RETURN 0" and recompile.
 
   To select the Target Lane Return version, make sure
   the preprocessor macro "DO_TARGET_LANE_RETURN" is set
   to "#define DO_TARGET_LANE_RETURN 1" and recompile.


The Finite State Machine 

The FSM, on each iteration, maintains the current state and
determines the next state.  The next state is set by modifying
the state variable before the iteration has completed.

states:
  - STATE_KEEP_LANE:  maintain safe distance to cars in your
                      lane and accelerate or decelerate as
                      needed. Compare target lane with current
                      lane to determine if switchback
                      needed to complete pass maneuver.

  - STATE_PREP_LC_LEFT:  check lane to left if car too close for
                         pontential lane change

  - STATE_PREP_LC_RIGHT: check lane to right if car too close for
                         potential lane change

  - STATE_LC_LEFT:  perform the lane change, record new lane
                    for eventual switchback to complete pass
                    maneuver

  - STATE_LC_RIGHT: perform the lane change, record new lane
                    for eventual switchback to complete pass
                    maneuver

I recorded two videos, one for the Target Lane Return version and
the other for the Lane Change Version.

### The Target Lane Return Version 
[![Target Lane Return Version](https://img.youtube.com/vi/kxHEH9LyKrg/0.jpg)](https://youtu.be/kxHEH9LyKrg)



### The Lane Change Version 
[![Lane Change Version](https://img.youtube.com/vi/41iD3uzLPiE/0.jpg)](https://youtu.be/41iD3uzLPiE)
