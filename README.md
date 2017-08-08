# CarND-Controls-MPC
### Autonomous Driving using Model Predictive Control (MPC)

This project involves writing a C++ program to steer a car around a track in a simulator. The simulator provides a stream of data including - car position, speed, heading and a reference path for the car to follow.  The coordinates are supplied in Global Cartesian A key constraint for the simulation is that actuations will have a latency of 100ms.

#### Introduction

For this project, we attempt to drive the car using Model Predictive Control. The inputs to the model are: state [x, y, ψ, ν] and actuations [δ, a]. MPC uses an optimizer to find the inputs that minimize the cost function.  The elements of the cost function are: 
•	cte: cross track error
•	epsi: error of heading orientation
•	v - speed limit
•	change rate of input

In the MPC controller, we pass the current state as the initial, call the optimization solver which then returns vector of the control inputs. The first control inputs are applied and then the solver is called again based on the new state of the car returned by the simulator.

The hyper-parameters to be tuned are:
•	N - the length of the trajectory
•	dt - each time step

Where T = N * dt will determine the time horizon. 

The choice of these hyper-parameters (along with the right cost model) are critical tuning knobs for the algorithm. Choosing a large T (predicting too far ahead) does not make sense as the track is continuously changing; a large value of dt can result in poor approximation of the control inputs to match the continuous reference trajectory whereas too small value results in higher computation cost. More about this in the following section.

#### Pre-processing

The global coordinates are first converted to vehicle co-ordinates by first adjusting each of the waypoints by subtracting out px and py accordingly such that they originate from the vehicle's position and then applying a standard 2d vector transformation equations:
ptsx[i] = x * cos(-psi) - y * sin(-psi);
ptsy[i] = x * sin(-psi) + y * cos(-psi);

Next the waypoints are approximated to a third-degree polynomial line and the cross-track error calculated by evaluating the polynomial against the position (zero because the origin is now the car position). Similarly, the heading error epsi is calculated from the difference of the car heading (again zero) and the derivative of 3rd degree polynomial trajectory.

#### Setting the cost model

The cost model uses weights and penalises deviations from the trajectory (cte), heading errors, use of actuators and large changes in actuations. Setting the weights low, caused the car to veer off the track especially at high speeds. This was essentially a trial and error exercise to find the weights that performed reasonably for the reference velocity of 70mph.

#### Accounting for Latency

I considered, 2 approaches to handle the effects of 100 ms latency in the acutations.

##### Predicting the car position in the latency interval and using that as the initial state
This approach predict the likely car position after time interval 100ms using the same update equations from the vehicle model and then feeding this new predicted state to the solver. This approach worked fine but for no particular reason, I decided to use and retain the second option described below.

##### Choosing time step as a multiple of the latency:
The time step dt was set to 0.1s, conveniently chosen to be a multiple of the latency which allowed the control variable to be picked up from one step before

A video of the car driving around the track using the above approach can be found at [video](https://youtu.be/xJuJECJRJuM):


---

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
       +  Some Mac users have experienced the following error:
       ```
       Listening to port 4567
       Connected!!!
       mpc(4561,0x7ffff1eed3c0) malloc: *** error for object 0x7f911e007600: incorrect checksum for freed object
       - object was probably modified after being freed.
       *** set a breakpoint in malloc_error_break to debug
       ```
       This error has been resolved by updrading ipopt with
       ```brew upgrade ipopt --with-openblas```
       per this [forum post](https://discussions.udacity.com/t/incorrect-checksum-for-freed-object/313433/19).
  * Linux
    * You will need a version of Ipopt 3.12.1 or higher. The version available through `apt-get` is 3.11.x. If you can get that version to work great but if not there's a script `install_ipopt.sh` that will install Ipopt. You just need to download the source from the Ipopt [releases page](https://www.coin-or.org/download/source/Ipopt/) or the [Github releases](https://github.com/coin-or/Ipopt/releases) page.
    * Then call `install_ipopt.sh` with the source directory as the first argument, ex: `sudo bash install_ipopt.sh Ipopt-3.12.1`. 
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




