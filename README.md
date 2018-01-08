# maze solver (ROS)
This repository contains an algorithm to solve a maze. Bascially it uses wall detection and wall follow. 

![maze](/uploads/0c6118ab0a00198acdcb42b54ebe1c1f/maze.png)

## Installing / Getting started (Linux)

### Install Ros and other Tools

**Warning! Please use Ubuntu 16.04 as this is the Long Term Support Version (LTS). If you are using Ubuntu 17.04 you have to use ROS Lunar release on your own risk!**

First goto http://wiki.ros.org/kinetic/Installation/Ubuntu and follow the instructions there. The installation can take some time depending on your internet connection.

After the ros setup is completed you should install these extra tools

```
sudo apt-add-repository ppa:webupd8team/atom
sudo apt-get update
sudo apt-get install git gitg htop terminator atom
sudo apt-get install ros-kinetic-turtlebot-simulator ros-kinetic-turtlebot-teleop
```

### Update your gazebo with the following commands

Add OSRF package repo

    sudo sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list'

setup keys

    wget http://packages.osrfoundation.org/gazebo.key -O - | sudo apt-key add -

update gazebo

    sudo apt-get update
    sudo apt-get install gazebo7



### Mounting Hokuyo Laser Scanner onto the Turtlebot

We have modified the `turtlebot_description` package that has the simulated model of the turtlebot and put in some upgrades that should make your life easier. After following these
instructions, you should be able to visualise the laser ray projections in gazebo, and you will have a new stable topic that gets the laser data from the new scanner, which is `/laserscan`. We
have also enhanced the field of vision of the scan to 180 degrees and the maximum range to 30 m.

* Delete the existing `turtlebot_description` package using the following commands:
```
cd /opt/ros/kinetic/share
sudo rm -r turtlebot_description
```

* Take the modified `turtlebot_description` package from `mod_turtlebot_description` to your `Downloads`
folder. Extract the package from the compressed file.

* Move the extracted package from the `Downloads` folder to the location of the deleted package using the following commands:
```
cd ~/Downloads
sudo mv turtlebot_description /opt/ros/kinetic/share
```
If everything went alright, there should be a file named `hokuyo_urdf.xacro` at the location `/opt/ros/kinetic/share/turtlebot_description/urdf/sensors`

* Run the following commands to setup the new modified turtlebot as the default whenever you launch it in gazebo:
```
echo "export TURTLEBOT_3D_SENSOR=hokuyo" >> ~/.bashrc
source ~/.bashrc
```
Note that by doing this, the turtlebot will be launched with hokuyo laser scanner everytime you launch `turtlebot_gazebo`. If you want to go back to how everything was before you did all this, just delete
the `export TURTLEBOT_3D_SENSOR` line in the `.bashrc` file.

* Launch the `turtlebot_gazebo` package and hopefully, you should see your new turtlebot with enhanced superpowers like below and you should be able to see the scan data in the `/laserscan` topic. 
  
    To launch the simulation use: `roslaunch turtlebot_gazebo turtlebot_world.launch`

![](https://fbe-gitlab.hs-weingarten.de/mat-iki/amr-mat/raw/master/.img/turtlebot_hokuyo.png)

### Loading the Maze into the Simulation

All you have to do to load it is run the following command after launching the `turtlebot_gazebo package`:

`rosrun gazebo_ros spawn_model -file ~/catkin_ws/src/<repo name>/maze_practice/model.sdf -sdf -model -maze -x 16 -y 5`

Be sure to enter your repository name correctly in the above command. An example:

`rosrun gazebo_ros spawn_model -file ~/catkin_ws/src/ch-171744_tier4/maze_practice/model.sdf -sdf -model -maze -x 16 -y 5`

Your gazebo simulation should now look like below:

![maze](/uploads/0c6118ab0a00198acdcb42b54ebe1c1f/maze.png)



## Description of the Solution

### Algorithm (shorted pseudocode)

```
save the actual x- and y-position (loop-detection)
state = "WallDetection"

while not rospy.is_shutdown():
   if(state == "WallDetection")
        search for a wall to follow
        adjust the angle to next wall
        drive to the wall
        while(not wall_arrived):
            drive
        save the actual x- and y-position (loop-detection)
        make a turn 
        state = "wallFollow"

    elif(state == "wallFollow")
        if(loop_detected):
            state = "wallDetection"
        else:
            if(obstacles_detected)
                do turn ~90 degree
            else  
                follow wall using PID-controller
```

### Wall Detection

* In this case the algorithm detects two walls. Beyond this two walls the wall with the biggest distance will be choosen. 

    ![wall1](/uploads/cc804a6a30a68fbe4ef532f0910bd25d/wall1.png)
    
    How does the wall detection work:
    
    There must be a predefined amount of points between the upper- and lower bound. Another constraint is that the points must be next to each other. 
    Otherwise it will not be detected as a wall.  
    
    ![wallDet6](/uploads/69e3766f6929081cb594986d43d67130/wallDet6.png)

* Do a turn and drive to the detected wall.

    ![wallD2](/uploads/f7a27bbbd6447538e1d0f6a603f1ff5a/wallD2.png)

* Save the actual x- and y-position (green circle) and do a turn. After that use the wall follow function.

    ![wallD3](/uploads/be8dab7568350e134cf5cd9d9282cfa0/wallD3.png)

### Wall Follow

A PID-controller is used to adopt the direction of movement .

![wallF](/uploads/569fb71f0dcb1b3193d32ea072b9df15/wallF.gif)


### Loop Detection

The robot saves the start position. After that the robot will follow this wall until it hits the start position. This whole drive along the wall is skipped in the video.
The robot hits the start position and does a wall detection. This is the reason why the robot can escape from this loop. Otherwise the robot would drive along this wall in a loop.

![loopDetec](/uploads/d869d099a6487baf842f13d862310e76/loopDetec.gif)


## Author
Christian Högerle

## Licensing
The code in this project is licensed under MIT license.
