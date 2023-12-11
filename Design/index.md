---
layout: page
title: Project Description
tags: [about, Jekyll, theme, moon]
date: 2023-12-10
comments: false
---
# Software design
## Vision & Sensing

## Controller
Similar to other robot arms (e.g. Sawyer bot), KUKA arm also supports the Moveit! controller, which is a widely used robotics manipulation platform. However, there exists a severe problem with Moveit! controller; Moveit! controller is too slow for the KUKA robot arm to acquire our desired speed to hit the high-speed ping-pong ball. In addition to that, since there exists compatibility issues our research team moved back and forth between the ROS2 foxy version (used for physical robot control) and ROS2 humble version (used for Gazebo simulation) for our KUKA control code.
To resolve this issue and make our KUKA arm to successfully reach our endgame goal, we implemented the custom joint controller based on kinpy package.
