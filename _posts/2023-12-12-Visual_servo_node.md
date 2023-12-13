---
layout: post
title: "Visual Servoing Control Node"
date: 2023-12-12
excerpt: "Our implementation of Visual Servoing Control Node"
comments: false
project: true
code: true
---
### Here's our python implementation of Visual Servoing Control Node: 
```python
import numpy as np
import rclpy
from rclpy.node import Node

from geometry_msgs.msg import Point


class VisualServoNode(Node):
    def __init__(self, node_name="visual_servo_node") -> None:
        super().__init__(node_name=node_name)

        self.paddle_z_offset = 0.19 # Distance from center of paddle to EE tip is 19cm
        self.paddle_y = -1.8        # Half-table length is 137cm, we add a bit extra to avoid collision with the table
        
        # publishers and subscribers
        self.bse_sub = self.create_subscription(
            Point, "/ball_state_estimate", self.on_bse, 10
        )
        
        self.target_pub = self.create_publisher(
            Point, "/target_prediction", 10
        )


    def on_bse(self, ball: Point) -> None:
        target = Point()
        target.x = ball.x / 100
        target.y = self.paddle_y
        target.z = (ball.z / 100) - self.paddle_z_offset
        target.x = max(min(target.x, 0.6), -0.6)    # Keeps x coordinate of paddle between +- 0.6
        target.z = max(min(target.z, 0.55), -0.1)      # Keeps z coordinate of paddle between 0.35 and 1
        print(f"Target: {target}")
        self.target_pub.publish(target)


def main(args=None):
    rclpy.init(args=args)
    visual_servo_node = VisualServoNode()
    rclpy.spin(visual_servo_node)
    rclpy.shutdown()
```