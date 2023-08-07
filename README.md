## Introduction 

This project is intended to demonstrate the integration of robot fleet management system which uses openRMF midleware framework with Turtlebot3 using Qualcomm Robotics platform RB5. It demonstrates the tremendous features of openRMF and its usage in robotics platform with navigation stack. 

 

## Prerequisites 

- A Linux host system with Ubuntu 20.04. 
- Install Android Platform tools (ADB, Fastboot)  
- Download and install the SDK Manager for RB5 
- Flash the RB5 firmware image on to the RB5 
- Setup the Network on RB5. 
- Installed python3.6 on RB5 
- Installed python3.8 on host system 
- Installed ros-galectic-desktop-full on host system 
- Installed docker on RB5 
- Installed Turtlebot3-galactic packages on host system 

 

## Installing Dependencies 

### Install non-ROS prerequisite packages on host system 

```sh 
sudo apt update && sudo apt install \ 
  git wget qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools \ 
  python3-rosdep \ 
  python3-vcstool \ 
  python3-colcon-common-extensions \ 
  # maven default-jdk   # Uncomment to install dependencies for message generation 
```

### Install dependent ROS2-galactic packages on host system 

- Install required packages 
```sh
$ sudo apt install ros-galactic-nav2-util ros-galactic-nav2-bringup ros-galactic-rviz2 
```
- Install vcstool 
```sh
$ sudo apt install python3-vcstool 
```
- Install colcon 
```sh
$ sudo apt install python3-colcon-common-extensions 
``` 
- Install Cartographer (To create map of your environment) 
```sh
$ sudo apt install ros-galactic-cartographer 
$ sudo apt install ros-galactic-cartographer-ros 
```
- Install Navigation2 
```sh
$ sudo apt install ros-galactic-navigation2 
$ sudo apt install ros-galactic-nav2-bringup 
```
- Install TurtleBot3 packages. 
```sh 
$ sudo apt install ros-galactic-dynamixel-sdk 
$ sudo apt install ros-galactic-turtlebot3-msgs 
$ sudo apt install ros-galactic-turtlebot3 
```
 
`NOTE: Add below line in the end of ~/.bashrc file, to avoid the environment variables exports for all the time while opening terminal. Make sure that you reopen the shell once the file is edited.` 
 
Add below lines in ~/.bashrc 
```sh
# APPEND AT THE END OF ~/.bashrc
export TURTLEBOT3_MODEL=burger 
export ROS_DOMAIN_ID=30 
source /opt/ros/galactic/setup.bash
``` 
Or run commands given below to update the ~/.bashrc 
```sh
$ echo export TURTLEBOT3_MODEL=burger >> ~/.bashrc 
$ echo export ROS_DOMAIN_ID=30 >> ~/.bashrc 
$ echo source /opt/ros/galactic/setup.bash >> ~/.bashrc 
```

## Setup Workspace 

### Setup Server (Host System) 

Start a new ROS 2 workspace, and pull in the necessary repositories, 
```sh
$ mkdir -p ~/ff_ros2_ws/src 
$ cd ~/ff_ros2_ws/src 
$ git clone https://github.com/open-rmf/free_fleet -b main 
$ git clone https://github.com/open-rmf/rmf_internal_msgs -b main 
```
 
Install all the dependencies through rosdep, 
```sh
$ cd ~/ff_ros2_ws 
$ rosdep install --from-paths src --ignore-src --rosdistro galactic -yr 
```
Source ROS 2 and build, 
```sh
$ cd ~/ff_ros2_ws 
# Run below command, if sourcing of galactic has not taken care in ~/.bashrc 
$ source /opt/ros/galactic/setup.bash 
$ colcon build 
$ source ~/ff_ros2_w/install/setup.bash 
```

### Setting Up Galacting & Free Fleet Client on RB5 (Qualcomm RB5) 

To enter RB5 shell open new terminal on host system and run below command, 

Access the RB5 Using SSH 
```sh
$ ssh root@<IP_ADDRESS> 
```
or Using ADB through host 

```sh
$ adb shell 
```

Clone the project by using below command  
```sh
$ git clone <GIT_LINK> 
```
Change directory where project is cloned and and make sure the Dockerfile is available in the root of the project: 
```sh
$ cd <PROJECT_ROOT_DIR> 
```
Assuming the docker installation has been already fulfilled as its mentioned in the prerequisites, Build the docker file by using the command below, 
```sh
$ docker build –t <DOCKER_IMAGE_NAME>:<VERSION_NO> .   
```
Run docker image 
```sh
$ docker run –it <DOCKER_IMAGE_NAME/DOCKER_IMAGE_ID> --network=host --device=/dev/ttyACM0 --device=/dev/ttyUSB0 bash 
```
 
```
--device=/dev/ttyACM0 : Sharing the OpenCR controller device to the container. 

--device=/dev/ttyUSB0 : Sharing the 2D lidar sensor with the container. 
```
 

`NOTE: Assuming the ROS2 Galactic environment has been initialized at both end using ~/.bashrc, as instructed to add the initialization commands in ~/.bashrc of both Host & RB5 device.` 

 

## Map Generation  
`(Optional if you do not have generated map (.pgm & .yaml))` 

Follow below steps, if you want to generate your own map 

To enter RB5 shell open new terminal on host system and run below command or access using ‘adb shell’ if device is connected with USB. 
```sh
$ ssh root@<IP_ADDRESS> 
```
Make sure the device & host system is in the same network. 

 

After entering in RB5 shell run bring up command, Source the workspace and launch bring up command 
```sh
sh4.4$ ros2 launch turtlebot3_bringup robot.launch.py 
```

Open a new terminal from host system source environment and launch the SLAM node. The Cartographer is used as a default SLAM method.  
```sh
$ ros2 launch turtlebot3_cartographer cartographer.launch.py 
```
 

Once the SLAM node is successfully up and running, TurtleBot3 will be exploring an unknown area and gives the visualization of the same on the RViz window.  

 

To explore your area completely open RB5 shell and source workspace and launch teleop node. 
```sh
sh4.4 ros2 run turtlebot3_teleop teleop_keyboard 
```

- Keyboard shortcuts w, a, x, s, d is used to move the bot and to every corner of the map.
  - w and x: for linear velocity 

  - a and d: Angular velocity 

  - s: stop  

 

 

After analyzing complete map on host system save the map using below command 
```sh
$ ros2 run nav2_map_server map_saver -f ~/map  
```

Image stored in asset/maps/ is the sample map generated. Black portion is a walls or obstacles, white portion is free space. 


### Free_Fleet client

Free_Fleet client launch command on RB5, please do not run the below command, as it is given only for understanding purpose. 
```sh
#JUST FOR INFORMATION, DO NOT RUN THIS COMMAND, PLEASE FOLLOW THE EXECUTION INSTRUCTION BELOW.
sh4.4 ros2 launch ff_examples_ros2 turtlebot3_world_ff.launch.xml 
```
 

### Free_Fleet server

Free_Fleet Server launch command on Host, please do not run the below command, as it is given only for understanding purpose. 
```sh
#JUST FOR INFORMATION, DO NOT RUN THIS COMMAND, PLEASE FOLLOW THE EXECUTION INSTRUCTION BELOW. 
$ ros2 launch turtlebot3_navigation2 navigation2.launch.py map:=/home/map.yaml 
```

### Navigation (Optional) 

The below steps are just to validate & verify the navigation on map passed is working properly or not. 

On RB5 shell, Source the workspace and launch bring up command 
```sh
sh4.4 ros2 launch turtlebot3_bringup robot.launch.py 
```
On host system source the workspace and launch navigation command  
```sh
$ ros2 launch turtlebot3_navigation2 navigation2.launch.py map:=/home/map.yaml 
```

Running this command will open visualization with map provided on the RViz. 

## Execution instruction
- On RB5 shell, Source the workspace and launch bring up command 
```sh
sh4.4 ros2 launch turtlebot3_bringup robot.launch.py 
```
- On host system run the free fleet server  
```sh 
$ ros2 launch ff_examples_ros2 turtlebot3_world_ff_server.launch.xml 
```
- On RB5 shell, Source workspace and launch client node 
```sh
sh4.4 ros2 launch ff_examples_ros2 turtlebot3_world_ff.launch.xml 
```
- On host system run navigation  
```sh
$ ros2 launch turtlebot3_navigation2 navigation2.launch.py map:=/home/map.yaml 
```
- On host system send navigation goal to navigate bot 
```sh
$ ros2 run ff_examples_ros2 send_destination_request.py -f turtlebot3 -r ros2_tb3_0 -x 1.725 -y -0.39 --yaw 0.0 -i 2 
```