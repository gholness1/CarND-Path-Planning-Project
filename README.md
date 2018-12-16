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

## Reflection

Path generation takes a two step approach. The first ensures consistency from each iteration
of the path planner to the next.  The second, produces waypoints along a smooth trajectory
that obey a maximum reference velocity.

1. Consistency across iterations of the path planner is achieved by inclusion of unconsumed waypoints
   from the previous iteration.   This gives a consistency because the previous path is extended instead
   of created from scratch. This also saves a little bit of computation.   In the beginning, when there
   are very few points, we create such consistency by stepping back from the first point if needed.

2. The generation of waypoints along a smooth trajectory is achieved using a spline fit to points
   by looking ahead some distance in the longitudinal direction (Frenet coordinates) along the road.
   The spline, what I call a lookahead spline, is then sampled by generating points that satisfy
   the reference velocity.  This is done by manipulation of the motion equation velocity= distance/time.
   The distance spanned by the spline is approximated as a straight line distance.   Using the Pythagorean
   theorum as an approximator, the distance is computed from the starting point of the spline to the
   lookahead (end point) of the spline.   Given this distance, we know the time for each iteration
   is 0.02 seconds.  Given this, the calculation gives us a number of steps N used to achieve that
   instance.   We have  N= (target_dist/(0.02 * ref_vel/2.24)).  This comes from...

   The conversion of 2.24 is to convert velocity from miles/hour to meters/second. We know the
   time to travel the distance is N-iterations times 0.02 seconds.  We know that the reference
   velocity divided by 2.24 gives us the velocity in meters/second.   This should equal the
   distance marked between the starting and endpoints for the spline.   We solve for N and this
   gives us the number of increments we need to obtain each waypoint's x-component for the spline
   sampling.   Given each x-component, we evaluate the spline to obtain the corresponding 
   y-component.

The aforementioned two steps together gives us the trajectory.  The nice part is that because
we are working in Frenet coordinates, if we have a lane change, the only thing that changes
is the starting (x,y) values for the spline and, therefore, the ending (x,y) values associated
with a different lane.  We simply recalculate what the d-value is in Frenet and call getXY.
Easy peasy smooth and breezy!
