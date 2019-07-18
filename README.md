# CarND-Capstone
This is the project repo for the final project of the Udacity Self-Driving Car Nanodegree: Programming a Real Self-Driving Car. For more information about the project, see the project introduction [here](https://classroom.udacity.com/nanodegrees/nd013/parts/6047fe34-d93c-4f50-8336-b70ef10cb4b2/modules/e1a23b06-329a-4684-a717-ad476f0d8dff/lessons/462c933d-9f24-42d3-8bdc-a08a5fc866e4/concepts/5ab4b122-83e6-436d-850f-9f4d26627fd9).

## Project Overview
This is the Capstone project for the Udacity Self-Driving Car Nanodegree. Using the Robot Operating System (ROS), we created nodes to implement core functionality of the autonomous vehicle system, including traffic light detection and classification, control, and waypoint following! 

The goal of the project is to enable the Carla/vehicle to self driving normally (Make sure Carla/vehicle make the right decisions at traffic lights) in a simulator or test lot provided by Udacity. We formed a team to finish the project.

## Team:Dream Car
Team Members: 
* **Yuan Feng** [github](https://github.com/Felix-yuan2018), [emai](yfinn@163.com) - team leader
* **Wang Meng** [github](https://github.com/damengsir), [email](xiaowangyx1@163.com)
* **Wu Gang** [github](https://github.com/Aitical), [email](w965813422@gmail.com)


## System Architecture
The following is a system architecture diagram showing the ROS nodes and topics used in the project. From this diagram we can see the flow of data and the structure of the code.
![pic referenced from Udacity](imgs/arc.png)
*Note: For this project, the obstacle detection node is not implemented*
Now we will introduce what is involved in this project.


## Traffic light detection and classification

This subsystem reads the world surrounding the vehicle and publishes relevant information to other subsystems. Specifically, this subsystem determines the state of upcoming traffic lights and publishes their status to other subsystems.

Car receives image from the camera, system can detect and classify a traffic light color, if the traffic light is not detected the None is returned. We do this by two models with two steps . First chose the Unet to detect the traffic light position, then we developed an network to identify the colors of traffic lights.

 *Note: Unet is inspired by FCN to do semantic segmentation for medical images, and can use a small amount of data to learn a very robust model for edge extraction, which plays a great role in the field of biomedical image segmentation.*
 
### Traffic light Detection

To get the traffic light detected from a picture we need take semantic segmentation techniques. We used [UNet](https://arxiv.org/pdf/1505.04597.pdf) to segment and build model in Keras. During building the model, we learned a lot from this [repo](https://github.com/zhixuhao/unet) which provided a useful model code and we used a previously trained model and weights then fine tuning on our own labled data and [Bosch](https://hci.iwr.uni-heidelberg.de/node/6132) dataset.

### Traffic light Classifier
To classify the type of the traffic light, we used the knowledge learned in the last term and built a simple but efficiency CNN model with FC Layer to classify the traffic lights. Details of the model showing in the following table:
![pic referenced from Udacity](https://raw.githubusercontent.com/Aitical/CarND-Capstone/master/imgs/cls.png)

**For more detail about the detector can be found in [detector](https://github.com/Felix-yuan2018/CarND-Capstone-Project/tree/master/detector)**

## Waypoint following
In this part, we loaded the waypoints provided by Udacity, then chose waypoints to guide Carla/vehicle to drive on the track, also we need to update the target velocity property of each waypoint based on traffic light data to make sure Carla/vehicle run normally at traffic lights.

### Waypoint Loader Node
This node was implemented by Udacity. It loads a CSV file that contains all the waypoints along the track and publishes them to the topic /base_waypoints. The CSV can easily be swapped out based on the test location (simulator vs real world).

### Waypoint updater
This node subscribes to three topics to get the entire list of waypoints, the vehicle’s current position, and the state of upcoming traffic lights. Once we receive the list of waypoints, we store this result and ignore any future messages as the list of waypoints won’t change. This node publishes a list of waypoints to follow - each waypoint contains a position on the map and a target velocity.

Every time the vehicle changes its position, we plan a new path. First, we find the closest waypoint to the vehicle’s current position and build a list containing the next 100 waypoints. Next, we look at the upcoming traffic lights. If there are any upcoming red lights, we adjust the speed of the waypoints immediately before the traffic light to slow the vehicle to a halt at the red light. The vehicle speed is controlled by the following rules, detail see below:
 - Waypoint updater performs the following at each current pose update
 - Find closest waypoint
   - This is done by first searching for the waypoint with closest 2D Euclidean distance to the current pose among the waypoint list
   - Once the closest waypoint is found it is transformed to vehicle coordinate system in order to determine whether it is ahead of the vehicle and advance one waypoint if found to be behind
   - Searching for the closest waypoint is done by constructing a k-d tree of waypoints at the start with x and y coordinates as dimensions used to partition the point space. 
  - The final list of waypoints is published on the /final_waypoints topic.
 - Calculate trajectory
   - The target speed at the next waypoint is calculated as the expected speed (v) at the next waypoint so that the vehicle reaches 0 speed after traversing the distance (s) from the next waypoint to the traffic light stop line and the largest deceleration (a)
   - Using linear motion equations it can be shown that v = sqrt(2 x a x s)
   - If there is no traffic light stopline, then target speed is set to the maximum
   -  In order to prevent vehicles from crossing the stop line, we need to slow down the car when there are 2 waypoints closest to the stop line.
 - Construct final waypoints
   - Published final waypoints are constructed by extracting the number of look ahead waypoints starting at the calculated next waypoint
   - The speeds at published waypoints are set to the lower of target speed and maximum speed of the particular waypoint

## Control 
This subsystem publishes control commands for the vehicle’s steering, throttle, and brakes based on a list of waypoints to follow. 

### Waypoint Follower Node

This node was given to us by Udacity. It parses the list of waypoints to follow and publishes proposed linear and angular velocities to the /twist_cmd topic

### Drive By Wire (DBW) Node
At this point we have a target linear and angular velocity and must adjust the vehicle’s controls accordingly. In this project we control 3 things: throttle, steering, brakes. As such, we have 3 distinct controllers to interface with the vehicle.

#### Throttle Controller
The throttle controller is a simple PID controller that compares the current velocity with the target velocity and adjusts the throttle accordingly. The throttle gains were tuned using trial and error for allowing reasonable acceleration without oscillation around the set-point.

#### Steering Controller
This controller translates the proposed linear and angular velocities into a steering angle based on the vehicle’s steering ratio and wheelbase length. Steering command is filtered with a low pass filter to avoid fast steering changes.

#### Braking Controller
We simply proportionally brake based on the difference in the vehicle’s current velocity and the proposed velocity. This proportional gain was tuned using trial and error to ensure reasonable stopping distances while at the same time allowing low-speed driving. 

## Native Installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. [Ubuntu downloads can be found here](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity provided virtual machine has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * [ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* [Dataspeed DBW](https://bitbucket.org/DataspeedInc/dbw_mkz_ros)
  * Use this option to install the SDK on a workstation that already has ROS installed: [One Line SDK Install (binary)](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/81e63fcc335d7b64139d7482017d6a97b405e250/ROS_SETUP.md?fileviewer=file-view-default)
* Download the [Udacity Simulator](https://github.com/udacity/CarND-Capstone/releases).

## Docker Installation
[Install Docker](https://docs.docker.com/engine/installation/)

Build the docker container
```bash
docker build . -t capstone
```

Run the docker file
```bash
docker run -p 4567:4567 -v $PWD:/capstone -v /tmp/log:/root/.ros/ --rm -it capstone
```

## Port Forwarding
To set up port forwarding, please refer to the "uWebSocketIO Starter Guide" found in the classroom (see Extended Kalman Filter Project lesson).

## Usage

1. Clone the project repository
```bash
git clone https://github.com/Felix-yuan2018/CarND-Capstone-Project.git
```

2. Install python dependencies
```bash
cd CarND-Capstone
pip install -r requirements.txt
```
3. Make and run styx
```bash
cd ros
catkin_make
source devel/setup.sh
roslaunch launch/styx.launch
```
4. Run the simulator

## Real world testing
1. Download [training bag](https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/traffic_light_bag_file.zip) that was recorded on the Udacity self-driving car.
2. Unzip the file
```bash
unzip traffic_light_bag_file.zip
```
3. Play the bag file
```bash
rosbag play -l traffic_light_bag_file/traffic_light_training.bag
```
4. Launch your project in site mode
```bash
cd CarND-Capstone/ros
roslaunch launch/site.launch
```
