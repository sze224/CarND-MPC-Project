# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

---

Model Predictive Control is another control method widely used in the self driving car world along with the PID controller. Model predictive control, as shown in the name, uses the kinematic model of the system to determine how the system should react and perform. In this case, we are provide with a trajectory that the vehicle should follow. Then we are determining how the actuator should move that can provide us with the optimal cost. In this case, cost can be whatever we defined, which will be discussed below.

## The model

State

The model that I am using in the project contain the state x, y, psi, and v. State x, y represent the vehicle coordinate. State psi represent the orientation of the vehicle. State v represent the speed of the car. 

Actuators

In this model, there are 2 actuators (delta and a). Delta is the actuator that controls the steering angle and a is the actuator that controls the acceleration of the vehicle.

Equations 

The equations used here are the simple kinematic equation derived from the lectures.
<img width="228" alt="screen shot 2017-06-20 at 9 02 29 pm" src="https://user-images.githubusercontent.com/22971963/27366576-e321a926-55fb-11e7-8bd4-725fdef36900.png">


Additionally, we also calculate the cross track error and the error is orientation based on the reference trajectory and the 
current states of the vehicle.

Cost functions

Here are the breakdown of what I used as a cost function:
* cte - how far it is from the desired position
* epsi - how different it is from the desired orientation
* v - how far it is from the reference velocity (which in this case is 50)
* delta - minimize the movement of steering actuator
* a - minimize the acceleration actuator
* d_delta - minimize the sequential actuations of steering
* d_a - minimize the sequentual acceleration actuator

In order to tune the scaling factor of these contribution, I first give cte and epsi a high contribution, as one of the major goal for a self driving car is to follow the trajectory. If the car is off the trajectory, the vehicle might crash into another car or start driving at an dangerous area. Once I have those gain set, I observed what kind of behavior the simulation is giving me. For instance, if I observe that the car is oscilliating a lot, then I will give delta and d_delta a higher contribution. Then using those gain, I will observe the behavior in the simulator. I keep repeating that process until I get to a point where I am content with the performance. When tuning this, I treat safety as a higher priority than passenger comfort, therefore the contribution of cte and epsi is higher than the rest. 

## Timestep Length and Elapsed Duration (N and dt)

Another set of parameters that we can pick is the timestep length and elapsed duration. when picking these number, we need to find a good balance. If we have a small dt, we will have higher precision but we also need higher computation time. If we have a large N, we can predict further into the future but we ignore many of the outside influence to the system. In this case, I picked N to be 10 and dt to be 0.15. This will allow us to predict 1.5s to the future. Since we want the self driving car to be able to react in real time, we want to have values that will give both good precision and computation performance.

## Polynomial Fitting and MPC Preprocessing

I decided to fit a 3rd order polynomial to the way point becasue we know that the road will have curvature to it, therefore we will need a least a 2nd order polynomial function to describe that.

For the Preprocessing step, we have to convert all the waypoints from the global coordinate view into the vehicle coordinate system, since the result is calculated from the vehicle's point of view.

## Model Predictive Control with Latency

In a real car, there is always latency assoicated with the system, meaning that the command doesn't get executed right away. This will cause a problem in which the vehicle might be at a different state when the actuator command is actually being executed. This can cause error and even instability. In this case, we are modeling the latency to the simulator by focusing the thread to sleep for 100ms in order to see the most real performance of the system.

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets 
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.
* Fortran Compiler
  * Mac: `brew install gcc` (might not be required)
  * Linux: `sudo apt-get install gfortran`. Additionall you have also have to install gcc and g++, `sudo apt-get install gcc g++`. Look in [this Dockerfile](https://github.com/udacity/CarND-MPC-Quizzes/blob/master/Dockerfile) for more info.
* [Ipopt](https://projects.coin-or.org/Ipopt)
  * Mac: `brew install ipopt`
  * Linux
    * You will need a version of Ipopt 3.12.1 or higher. The version available through `apt-get` is 3.11.x. If you can get that version to work great but if not there's a script `install_ipopt.sh` that will install Ipopt. You just need to download the source from the Ipopt [releases page](https://www.coin-or.org/download/source/Ipopt/) or the [Github releases](https://github.com/coin-or/Ipopt/releases) page.
    * Then call `install_ipopt.sh` with the source directory as the first argument, ex: `bash install_ipopt.sh Ipopt-3.12.1`. 
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [CppAD](https://www.coin-or.org/CppAD/)
  * Mac: `brew install cppad`
  * Linux `sudo apt-get install cppad` or equivalent.
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions


1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

## Hints!

* You don't have to follow this directory structure, but if you do, your work
  will span all of the .cpp files here. Keep an eye out for TODOs.

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./
