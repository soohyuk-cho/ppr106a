---
layout: page
title: Implementation
tags: [about, Jekyll, theme, moon]
date: 2023-12-10
comments: false
---

# Software Implementation

## Live Update
We implemented <span style="color:deepskyblue">live update</span> code to predict the position at which the paddle will intercept the ball based on the data from the cameras. To precisely predict the post-hit trajectory, we fit a curve--<span style="color:deepskyblue">quadratic in z, linear in x and y</span>--given the orientation of the spatial frame located at the center of the table to the ball position data points (computed from the camera images) <span style="color:deepskyblue">after the ball is hit</span> by the player but <span style="color:deepskyblue">before it bounces</span> on the table. The coefficients of this curve are used to generate the coefficients for a new curve that the ball is predicted to follow after bouncing on the table.

We are using a <span style="color:deepskyblue">linear regression model</span> for this prediction task (although we have a non-linear model, the data points for these nonlinearities go in the data matrix which we run least-squares on to attain the coefficients to the parabolic equations) 

Based on this implementation and our test results, the average error of our prediction is around 6 inches, which was sufficient enough for the paddle to make contact with the ball in simulation, given a small amount of adjustment time after the ball bounces on the table

## Joint Controller
We decided to implement our own custom controller because 1) the moveIt controller was too slow for the KUKA robot arm to acquire our desired speed and 2) we had compatibility issues as our research team moved back and forth between the ROS2 foxy version and ROS2 humble version for our KUKA control code. As a separate team works on figuring out how to resolve compatibility issues so that we can actually test our KUKA arm via our ROS2 code, we set up a virtual box container so that we can try testing the joint movement in simulation (using RViz and Gazebo). We confirmed that the simulation movement aligns with our robot using the joint trajectory executioner node, one of the demo files LBR-Stack demos for KUKA's Fast Robot Interface. 

We initially tried to implement the joint controller by manually calculating the Jacobian and finding the joint velocities for all 7 joints of the KUKA arm, but last week, we found and tested a kinpy package which has a function that calculates the Jacobian directly from our urdf file. Here's our implmentation of joint controller based on kinpy package and LBR executioner node: [joint controller code](../Joint_Controller)

