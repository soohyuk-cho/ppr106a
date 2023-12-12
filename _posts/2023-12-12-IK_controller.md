---
layout: post
title: "IK Controller"
date: 2023-12-12
excerpt: "IK Controller Code"
comments: false
project: true
code: true
---
### Here's our python implementation of IK controller: 
```python
import kinpy
import numpy as np
from lbr_fri_msgs.msg import LBRCommand, LBRState
from sensor_msgs.msg import JointState
from typing import Tuple

class IKController:
    
    def __init__(
        self,
        robot_description: str,
        base_link: str = "link_0",
        end_effector_link: str = "link_ee",
        world_config: kinpy.Transform = kinpy.Transform(),
    ) -> None:
        self.chain = kinpy.build_serial_chain_from_urdf(
            data=robot_description,
            root_link_name=base_link,
            end_link_name=end_effector_link,
        )
        self.world_config = world_config.matrix()
        self.velocity_limits = np.array([1.4, 1.4, 1.7, 1.25, 2.2, 2.3, 2.3])
        # self.velocity_limits = np.array([1, 1, 1, 1, 1, 1, 1])

    def step_control(self, target: np.ndarray, joint_state: JointState) -> np.ndarray:
        """
        Parameters
        ----------
        target: 3x' :obj:`numpy.ndarray` of the desired target position in the world frame
        lbr_state: LBRState of the current joint configuration
        sample_period: float of time between calls to controller

        Returns
        -------
        7x' :obj:`numpy.ndarray` of joint positions for arm to move towards
        """
        theta = np.array(joint_state.position)
        theta[2], theta[3] = theta[3], theta[2] # Fixes an evil bug
        target_position = (np.linalg.inv(self.world_config) @ np.append(target, 1))[:3]
        current_config = self.chain.forward_kinematics(theta).matrix()
        target_config = kinpy.Transform(np.array([np.sqrt(2)/2, 0, 0, np.sqrt(2)/2]), target_position)
        error_config = np.linalg.inv(current_config) @ target_config.matrix()
        print(f"Error: {np.linalg.norm(error_config[:3, 3])}")
        target_theta = self.chain.inverse_kinematics(target_config)
        if np.linalg.norm(error_config[:3, 3]) < 0.01:
            print("Stopped")
            return None, None
        required_time = np.max((theta - target_theta) / self.velocity_limits)
        return target_theta, required_time

    def get_ee_position(self, joint_state: JointState):
        theta = np.array(joint_state.position)
        theta[2], theta[3] = theta[3], theta[2] # Fixes an evil bug
        current_config = self.chain.forward_kinematics(theta)
        # current_config = self.robot_model.get_global_link_transform("link_ee", theta)
        return current_config
```