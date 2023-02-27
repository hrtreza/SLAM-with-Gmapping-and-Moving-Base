# SLAM with Gmapping and Moving Base
This ROS robot simulation features a circular path that resembles a racetrack and uses a laser scanner to measure distances from obstacles. Users can control the robot's movements through various means, such as inputting x,y coordinates, using the keyboard, or utilizing a driving assistance feature.
<br>

Control of a robot in a simulated environment.


<!-- TABLE OF CONTENTS -->
<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#Introduction">Introduction</a></li>
    <li><a href="#Installing_and_Running">Installing_and_Running</a></li>
    <li><a href="#How_it_works">How_it_works</a></li>
    <li><a href="#PseudoCode">PseudoCode</a></li>
    <li><a href="#Simulation_and_Results">Simulation_and_Results</a></li>
    <li><a href="#Robot_Movement_Improvement">Robot_Movement_Improvement</a></li>
  </ol>
</details>


# Introduction

An overview of this program function.<br>


<!-- In this project, it is supposed to move and control this robot autonomously without any collision with the walls surrounding the whole complex path. Meanwhile, it is also essential to have a simple user interface to communicate with the robot in the case of increasing/decreasing the speed, resetting the robot position, etc. In addition, it is worth mentioning that all requirements should not be considered in more than two nodes.
 -->
This program manage a robot, endowed with laser scanners, which should move autonomously inside a map.<br>
You can use the user interface to:
<ol>
    <li>Let the robot to autonomously reach a x,y coordinate inserted by command line.</li>
    <li>Drive the robot with the keyboard.</li>
    <li>Drive the robot with the keyboard availing of a simple driving assistance.</li>
</ol>

The map is this one:<br>
## Gazebo:
<p align="center">
<img src="https://user-images.githubusercontent.com/94115975/221689972-e9425967-7ca6-4488-aab5-f66a1d31d0a5.png" width="900" height="500">
</p>

## Rviz:

<p align="center">
<img src="https://user-images.githubusercontent.com/94115975/221690077-f9f397e6-4ba1-4c01-b25a-99ee7c71c638.png" width="900" height="500">
</p>


# Installing_and_Running

Open the terminal, and download this repository:

<pre><code>git clone https://github.com/ParinazRmp/SLAM-with-Gmapping-and-Moving-Base.git </code></pre>

Copy or move the folder final_assignment into the src folder of your ROS workspace.<br> 
Go into the root folder of your ROS workspace and type: 

<pre><code>catkin_make</code></pre>

By using the xterm tool, it is possible to launch all nodes using the launch file final_assignment.launch, but first we need to install xterm:

<pre><code>sudo apt install xterm</code></pre>

Now we can type:

<pre><code>roslaunch final_assignment final_assignment.launch</code></pre>



# How_it_works

The program use the launch file "simulation_gmapping.launch" to run the simulated environment, and the launch file "move_base.launch" to run the action move_base that provides several topics, including:
<ul>
    <li>move_base/goal to publish the goal position;</li>
    <li>move_base/feedback to receive the feedback;</li> 
    <li>move_base/cancel to cancel the current goal.</li>
</ul>

There are 3 subscribers that run simultaneously thanks to a multi-thread architecture given by the ROS class AsyncSpinner:
<ul>
    <li>sub_pos: subscribes to the topic /move_base/feedback through the function currentStatus that continuosly update the current goal ID and check whether the robot has reached the goal position.</li>
    <li>sub_goal: subscribes to the topic /move_base/goal through the function currentGoal that continuosly update the current goal coordinates.</li>
    <li>sub_laser: subscribes to the topic /scan through the function drivingAssistance that continuosly take data by the laser scanner and, if the driving assistance is enabled, help the user to drive the robot stopping its if there is a wall too close in the current direction.</li>
</ul>

The robot can:
<ol>
    <li>Autonomously reaching a goal position: 
        <ul>
            <li>ask to the user to insert the coordinates x and y to reach;</li>
            <li>save the current time;</li>
            <li>set the frame_id to "map" (corresponding to the environment that is used) and the new coordinates to reach;</li>
            <li>publish the new goal to move_base/goal.</li>
        </ul>
    </li>
    <li>Cancel the current goal:
        <ul>
            <li>take the current goal ID;</li>
            <li>publish its to the topic move_base/cancel.</li>
        </ul>
    </li>
    <li>Be driven by the user through the keyboard (the list of commands is printed on the console).</li>
</ol>

You can change 3 constant values to modify some aspect of the program:
    <ul>
        <li>DIST: minimum distance from the wall with the driving assistance enabled.</li>
        <li>POS_ERROR: position range error.</li>
        <li>MAX_TIME: maximum time to reach a goal (microseconds).</li>
    </ul>

## PseudoCode
A short description of the program behavior is this one:
<pre><code>
FUNCTION manualDriving
    WHILE user input is not to quit
        TAKE user input through the keyboard
        EXECUTE corresponding task
        PUBLISH new robot velocity
    END WHILE
END FUNCTION

FUNCTION drivingAssistance WITH (msg)
    COMPUTE minimum distance on the right
    COMPUTE minimum distance in the middle
    COMPUTE minimum distance on the left
    
    IF driving assistance is enabled AND the robot is going against a wall THEN
        SET robot velocity TO 0
        PUBLISH robot velocity
    END IF

    IF a goal position is set THEN
        COMPUTE the time elapsed
        IF the time elapsed IS GREATER THAN 120 seconds THEN
            DELETE current goal
        END IF
    END IF
END FUNCTION

FUNCTION currentStatus WITH (msg) 
    SET current robot position
    COMPUTE the difference between the current robot position and the current goal position
    IF the difference IS LESS THAN 0.5 THEN
        STOP to compute the elapsed time
    END IF
END FUNCTION

FUNCTION currentGoal WITH (msg)
    SET current goal position
END FUNCTION

FUNCTION userInterface 
    WHILE user input is not to quit
        TAKE user input through the keyboard
        EXECUTE corresponding task
    END WHILE
END FUNCTION

FUNCTION main WITH (argc, argv)
    INITIALIZE the node "final_robot"

    SET the first publisher TO "move_base/goal"
    SET the second publisher TO "move_base/cancel"
    SET the third publisher TO "cmd_vel"

    SET the first subscriber TO "/move_base/feedback" WITH currentStatus
    SET the second subscriber TO "/move_base/goal" WITH currentGoal
    SET the third subscriber TO "/scan" WITH drivingAssistance

    INITIALIZE spinner WITH 3 threads
    START spinner
    CALL userInterface
    STOP spinner
    CALL ros::shutdown
    CALL ros::waitForShutdown

    RETURN 0
END FUNCTION

</code></pre>

# Simulation_and_Results
![rosgraph](https://user-images.githubusercontent.com/94115975/221713615-f280fb6c-fa7d-4b0c-bf68-9f28d757a033.png)


# Robot_Movement_Improvement

The driving assistance can be improved by move the robot in the right direction when the user is driving 
its against a wall, instead of just stop it.
