---
layout: page
title: Design
tags: [about, Jekyll, theme, moon]
date: 2023-12-10
comments: false
---
# Hardware design
Our KUKA arm contains 7 revolute joints, and the ping pong paddle is connected to the end effector of our robot with a laser-cut wooden mount. Joint Control can be done by using ROS2. For our KUKA arm, we also have a physical controller. To take full control of the robot using our lab PC, we change the mode from *T1* (manual control) to *AUT* (Automated control) mode. Then, we kill the existing processes and reconnect to the application *LBRServer* with the selected remote IP address. After setting the send period and choosing *position controller*, we can take control of the KUKA arm using plan and execute commands in RViz as well as with our custom controller. All of this can be seen in the video to the right.

# Software design

## Vision & Sensing
We use <span style="color:chocolate">four cameras</span> to view the table from different angles. For all four cameras, on every frame, two things happen. First, a <span style="color:chocolate">mask is applied to filter out</span> everything except for orange pixels (the color of the ping pong ball). Then, the <span style="color:chocolate">OpenCV Hough Circles algorithm</span>  is used to find any circles (the approximate shape of a ping pong ball) in the image. This particular algorithm works by computing all the edge pixels in the masked image. Then, for every edge pixel, it <span style="color:chocolate">computes the gradient vector</span> at that pixel and <span style="color:chocolate">considers all possible circles</span> whose center is in the direction of the gradient with a radius that intersects this edge point. It finally returns the circle that would hit the most edge pixels. After computing the ball position in all four camera frames, we triangulate the position of the ball in the world coordinates.

## Controller
Similar to other robot arms (e.g. Sawyer bot), KUKA arm also supports the Moveit! controller, which is a widely used robotics manipulation platform. However, there exists a severe problem with Moveit! controller; Moveit! controller is too slow for the KUKA robot arm to acquire our desired speed to hit the high-speed ping-pong ball. In addition to that, since there exists compatibility issues our research team moved back and forth between the ROS2 foxy version (used for physical robot control) and ROS2 humble version (used for Gazebo simulation) for our KUKA control code.
To resolve this issue and make our KUKA arm to successfully reach our endgame goal, we implemented the custom joint controller based on kinpy package.
