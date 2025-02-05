---
layout: page
title: Project Description
tags: [about, Jekyll, theme, moon]
date: 2023-12-10
comments: false
---
## Abstract

Implement a KUKA manipulator controller to intercept an incoming high-speed ball using both:

(1) pre-hit trajectory forecast for anticipatory planning and

(2) post-hit forecast updates for fine-grained planning.

## About Ping Pong Robotics (PPR) Project
<img style="float: right; margin: 0px 0px 15px 15px;" src="../assets/img/robot-zero-config.png"/>

Navigating high-speed environments presents a significant challenge for robots, especially in tasks like ping pong where predicting the trajectory of a fast-moving ball is crucial for timely responses. This PPR project is fascinating as it delves into the challenges of controlling a robot in a high-speed environment, specifically intercepting a swiftly moving ping pong ball. Predicting the trajectory of the ball before and after impact demands precision in interpreting visual cues and making split-second decisions for effective robot responses. We're tackling the complexities of real-time prediction and response to fast movements, using computer vision to track the ball and refining our calculations with post-hit data. Developing a custom Jacobian controller and implementing a Visual Servoing Algorithm highlights our need for sophisticated control systems adaptable to dynamic scenarios. Our ultimate goal—to employ a KUKA manipulator not just to intercept but also return a high-speed hit—underscores the comprehensive nature of this challenge, demanding seamless coordination between perception, prediction, and action.

Our focus is on developing a system that uses visual cues from the game to predict future movements, helping a ping-pong-playing robot anticipate actions better despite uncertainties. With a good combination of visual modeling, prediction, and a control system that leverages these cues, we tried to simulate a robot so that it can make proper contact with the ping pong ball coming in with a high enough speed.

The applications of our work span beyond the realm of ping pong. The technology and methodologies we're developing hold promise in various real-world robotics applications. Numerous subfields of robotics that require precise object interception or manipulation in high-speed environments, such as manufacturing assembly lines dealing with swiftly moving components or automated quality control systems, could significantly benefit from our predictive and adaptive capabilities. Moreover, fields like sports training or rehabilitation robotics, emphasizing hand-eye coordination and precise motor control, stand to gain from the techniques we're refining through this project. Overall, our fusion of predictive modeling, computer vision, and adaptable control systems opens doors to enhancing robot performance in diverse scenarios where rapid and accurate decision-making is paramount.

## High-level Goals

Our original goal was to maximize the probability that the paddle intercepts the incoming, high-speed ping pong ball, with our MVP being a successful interception. 

The figure below illustrates our ideated structure of our ROS code initially. This brainstorming was instrumental in getting ramped up on the integration of the different components of the project.

<img src="../assets/img/ros_init.png"/>

**Computer Vision & Planning**

1.  Compute the ball trajectory
    
2.  Update estimates for the ball's future position using post-hit data stream.
    

**Controller**

1.  Implement a custom Jacobian controller
    
2.  Implement Visual Servoing Algorithm
    

**Ultimate Goal:**

Use KUKA to Return a High-speed Hit

## ACKNOWLEDGEMENTS

This project was one of the research projects of FA23 offering of EECS C106A. We would like to express our sincere gratitude to Nima Rahmanian, Gaurav Bhatnagar, Srisai Nachuri, and Jet Situ for providing this amazing opportunity to work on Ping Pong Robotics (PPR) hardware projects and detailed advice and help on how to improve and debug our custom controller and trajectory update.