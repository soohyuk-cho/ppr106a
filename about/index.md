---
layout: page
title: Project Description
tags: [about, Jekyll, theme, moon]
date: 2023-12-10
comments: false
---
Implement a KUKA manipulator controller to intercept an incoming high-speed ball using both:

(1) pre-hit trajectory forecast for anticipatory planning and

(2) post-hit forecast updates for fine-grained planning.

## Introduction


Navigating high-speed environments presents a significant challenge for robots, especially in tasks like ping pong where predicting the trajectory of a fast-moving ball is crucial for timely responses.

Our focus is on developing a system that uses visual cues from the game to predict future movements, helping a ping-pong-playing robot anticipate actions better despite uncertainties. With a good combination of visual modeling, prediction, and a control system that leverages these cues, we tried to simulate a robot so that it can make proper contact with the ping pong ball coming in with a high enough speed.

## High-level Goals
<img style="float: right; margin: 0px 0px 15px 15px;" src="assets/img/robot-zero-config.png" width="100" />


**Computer Vision & Planning**

1.  Compute the ball trajectory
    
2.  Update estimates for the ball's future position using post-hit data stream.
    

**Controller**

1.  Implement a custom controller
    
2.  Implement Visual Servoing Algorithm
    

**Ultimate Goal:**

Use KUKA to Return a High-speed Hit

## ACKNOWLEDGEMENTS

This project was one of the research projects of FA23 offering of EECS C106A. We would like to express our sincere gratitude to Nima Rahmanian, Gaurav Bhatnagar, and Srisai Nachuri for providing this amazing opportunity to work on Ping Pong Robotics (PPR) hardware projects and detailed advice and help on how to improve and debug our custom controller and trajectory update.