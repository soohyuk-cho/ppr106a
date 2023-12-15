---
layout: post
title: "Jacobian Controller"
date: 2023-12-12
excerpt: "Jacobian Controller Code"
comments: false
project: true
code: true
---
### Here's our python implementation of Jacobian joint controller: 
```python
import kinpy
import numpy as np
from sensor_msgs.msg import JointState

class WorkspaceVelocityController:
    
    def __init__(
        self, 
        robot_description, 
        base_link="link_0", 
        end_effector_link="link_ee", 
        world_config=kinpy.Transform(), 
        Kp=np.ndarray([1, 1, 1, 1, 1, 1]),
    ):
        self.chain = kinpy.build_serial_chain_from_urdf(                                        # Create robot model from urdf
            data=robot_description,
            root_link_name=base_link,
            end_link_name=end_effector_link,
        )
        self.world_config = world_config.matrix()                                               # Base to world frame transform
        self.Kp = np.diag(Kp)                                                                   # Gain matrix
        self.have_stopped = False                                                               # To check if error is small enough
        self.velocity_limits = np.array([1.4, 1.4, 1.7, 1.25, 2.2, 2.3, 2.3])                   # Max joint velocities


    def hat(self, v):                                                                           # Computes hat map of v in R3
        if v.shape == (3, 1) or v.shape == (3,):
            return np.array([
                    [0, -v[2], v[1]],
                    [v[2], 0, -v[0]],
                    [-v[1], v[0], 0]
                ])
        elif v.shape == (6, 1) or v.shape == (6,):
            return np.array([
                    [0, -v[5], v[4], v[0]],
                    [v[5], 0, -v[3], v[1]],
                    [-v[4], v[3], 0, v[2]],
                    [0, 0, 0, 0]
                ])
        else:
            raise ValueError


    def adjoint(self, g):                                                                       # Computes adjoint of a twist
        if g.shape != (4, 4):
            raise ValueError

        R = g[0:3,0:3]
        p = g[0:3,3]
        result = np.zeros((6, 6))
        result[0:3,0:3] = R
        result[0:3,3:6] = self.hat(p) * R
        result[3:6,3:6] = R
        return result


    def g_matrix_log(self, g: np.ndarray) -> Tuple[np.ndarray, float]:                          # Computes matrix log
        R = g[:3, :3]                                                                           # SE(3) -> se(3)
        p = g[:3, 3]
        w, theta = self.rot_matrix_log(R)
        if w.any():
            A = np.matmul(np.eye(3) - R, self.hat(w)) + np.outer(w, w) * theta
            v = np.linalg.solve(A, p)
        else: 
            w = np.zeros(3)
            theta = np.linalg.norm(p)
            v = p / theta
        xi = np.hstack((v, w))
        return xi, theta


    def rot_matrix_log(self, R: np.ndarray) -> Tuple[np.ndarray, float]:                        # Computes matrix log
        tr_R = min(3, max(-1, sum(R[i, i] for i in range(3))))                                  # SO(3) -> so(3)
        theta = np.arccos((tr_R - 1) / 2.0)
        w = (1 / (2 * np.sin(theta))) * np.array([R[2, 1] - R[1, 2],
                                                  R[0, 2] - R[2, 0],
                                                  R[1, 0] - R[0, 1]])
        return w, theta


    def step_control(self, target, joint_state, sample_period):                                 # Controller method
        theta = np.array(joint_state.position)                                                  # Get current joint positions
        theta[2], theta[3] = theta[3], theta[2]                                                 # Fixes an evil bug
        target_position = (np.linalg.inv(self.world_config) @ np.append(target, 1))[:3]         # Puts target position in KUKA frame
        current_config = self.chain.forward_kinematics(theta).matrix()                          # Get current SE(3) config of KUKA EE
        target_config = kinpy.Transform(                                                        # Target is facing upwards towards table
            np.array([np.sqrt(2)/2, 0, 0, np.sqrt(2)/2]),
            target_position
        ).matrix()
        error_config = np.linalg.inv(current_config) @ target_config                            # Compute error config
        xi, twist_theta = self.g_matrix_log(error_config)                                       # Compute error twist
        error_twist = xi * twist_theta

        if np.any(error_twist > 10) or np.any(error_twist < -10):                               # Throw error if twist is too large
            raise Exception                                                                     # (this comes from singular configurations)
            return

        jacobian_pinv = np.linalg.pinv(self.chain.jacobian(theta), rcond=0.1)                   # Compute Jacobian psuedoinverse
        theta_dot = jacobian_pinv @ self.Kp @ self.adjoint(current_config) @ error_twist        # Joint velocity computation
        K = np.max(np.abs(theta_dot) / self.velocity_limits)                                    # Scaling factor to keep joint velocities
        theta_dot = theta_dot / K                                                               # in safe region

        if np.linalg.norm(error_config[:3, 3]) < 0.01:                                          # Joint velocity is 0 when controller is stopped
            if self.have_stopped:
                return theta, np.zeros(7)
            else:
                self.have_stopped = True
                return theta + sample_period * theta_dot, np.zeros(7)
        else:
            self.have_stopped = False
            return theta + sample_period * theta_dot, theta_dot                                 # Returns next position and velocity
```