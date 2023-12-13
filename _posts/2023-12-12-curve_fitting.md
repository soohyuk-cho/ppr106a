---
layout: post
title: "Curve Fitting code"
date: 2023-12-12
excerpt: "Our code implementaiton of curve fitting for live updates"
comments: false
project: true
code: true
---
### Here's our python implementation of curve fitting algorithm for live updates process: 
```python
from visual_servo import VisualServo
import numpy as np
from data_segmentation.src.data_structure.processed_frame import ProcessedFrame
from data_segmentation.src.utils import split_posthit_segment_on_bounce
from typing import List
from copy import deepcopy
import matplotlib.pyplot as plt


class VisualServoCurveFitting(VisualServo):
    """
    Visual servo model that computes trajectory by interpolating parabolas.
    If the ball has bounced, interpolates a parabola from the post-bounce trajectory
    and intersects it with the paddle plane.
    Otherwise, interpolates a parabola from the pre-bounce trajectory and compute a 
    post-bounce parabola through linear regression on x slope, y slope, height, and b 
    coefficient of the pre-bounce parabola.
    """

    # Coefficients for linear regression prediction model
    # These coefficients describe affine functions that predict how quantities update post-bounce
    # E.g., x_predictor_coeffs describes how dx/dt changes after the bounce
    # So mx2 (post-bounce slope) = ~0.6 * mx1 (pre-bounce slope) + 0.004
    # y_predictor_coeffs describes how dy/dt changes after the bounce
    # b_predictor_coeffs describes how the coefficient b in z = at^2 + bt + c changes after the bounce
    # and h_predictor_coeffs describes how the height of the curve changes after the bounce
    # These 4 quantities are sufficient to compute the 7-parameter curve because we know that the
    # pre-bounce and post-bounce curves intersect at the point of the bounce, which gives us 3 data points (x, y, z of bounce point)
    x_predictor_coeffs = np.array([0.64621385, 0.00466148])
    y_predictor_coeffs = np.array([0.61873083, 0.05157728])
    b_predictor_coeffs = np.array([1.68778697, 6.63926297])
    h_predictor_coeffs = np.array([0.58883477, 6.97740610])

    def __init__(
            self, 
            paddle_y, 
            paddle_start_position, 
            ball_z_offset, 
            min_window_size_of_pre_bounce_curve, 
            min_window_size_of_post_bounce_curve
        ) -> None:
        self.paddle_y = paddle_y
        self.ball_z_offset = ball_z_offset
        self.min_window_size_pre = min_window_size_of_pre_bounce_curve
        self.min_window_size_post = min_window_size_of_post_bounce_curve
        self.prev_target_position = paddle_start_position

    def get_coefficients(self, segment_frames, return_error=False):
        """
        Interpolate a curve based on the positions of the ball.
        The curve is linear in x and y and quadratic in z.
        
        """
        ball_traj = np.array([np.array([f.ball_position[0], f.ball_position[1], f.ball_position[2] + self.ball_z_offset, t]) for t, f in enumerate(segment_frames)])

        xs, ys, zs, ts = ball_traj[:, 0], ball_traj[:, 1], ball_traj[:, 2], ball_traj[:, 3]
        lin_A = np.vstack([ts, np.ones(len(ts))]).T
        x_coeffs, x_error, _, _ = np.linalg.lstsq(lin_A, xs, rcond=None)
        m_x, c_x = x_coeffs
        y_coeffs, y_error, _, _ = np.linalg.lstsq(lin_A, ys, rcond=None)
        m_y, c_y = y_coeffs

        quad_A = np.vstack([np.square(ts), ts, np.ones(len(ts))]).T
        z_coeffs, z_error, _, _ = np.linalg.lstsq(quad_A, zs, rcond=None)
        a_z, b_z, c_z = z_coeffs

        coeffs = m_x, c_x, m_y, c_y, a_z, b_z, c_z
        if return_error:
            error = (x_error + y_error + z_error)[0]
            return coeffs , error
        else:
            return coeffs

    def affine_predictor(self, old, coeffs):
        """
        Affine function described by an array of two real numbers.
        The first entry of coeffs is the linear coefficient and the second entry is the constant term.
        E.g., if coeffs = [3 5], then this returns 3 * old + 5 as the predicted value.
        """
        return coeffs[0] * old + coeffs[1]

    def get_z_coeffs_from_b_h_t0(self, b, h, t0):
        """
        Computes the coefficients of the curve z = at^2 + bt + c where
        b, the height of the curve h, and the smallest root of the parabola t0, are all given.
        Note that a satisfies the quadratic equation (t0 a)^2 + (h + b t0) a + b^2 / 4 = 0.
        If this equation has multiple solutions, we choose the a that has t0 as the smaller root of the parabola.
        And c can be computed from the equaiton c = -t0 (b + a t0).
        """
        x, y, z = t0 ** 2, h + t0 * b, (b ** 2) / 4
        a_values = np.roots([x, y, z])
        if len(a_values) == 0:
            return None
        elif len(a_values) == 1:
            a = a_values[0]
            return a, b, c
        else:
            a1, a2 = a_values
            if -b / (2 * a1) > t0:
                a = a1
            else:
                a = a2
        c = -t0 * (b + a * t0)
        return a, b, c

    def get_x_coeffs_from_m_c_t0(self, m, c, t0):
        """
        Computes the coefficeints of the linear function x = mt + c.
        """
        mx = self.affine_predictor(m, self.x_predictor_coeffs)
        return mx, -mx * t0 + m * t0 + c

    def get_y_coeffs_from_m_c_t0(self, m, c, t0):
        """
        Computes the coefficients of the linear function y = mt + c
        """
        my = self.affine_predictor(m, self.y_predictor_coeffs)
        return my, -my * t0 + m * t0 + c

    def get_new_coeffs_from_b_h(self, m_x, c_x, m_y, c_y, a_z, b_z, c_z):
        """
        Computes the coefficeints of a post-bounce curve linear in x and y, quadratic in z,
        given the coefficients of the pre-bounce curve.
        Uses affine predictors in mx, my, height of z parabola, and bz.
        Note that the height of a parabola is defined by h = (-b^2 / 4a) + c.
        """
        t0 = np.max(np.roots([a_z, b_z, c_z]))
        mx1, cx1 = self.get_x_coeffs_from_m_c_t0(m_x, c_x, t0)
        my1, cy1 = self.get_y_coeffs_from_m_c_t0(m_y, c_y, t0)
        b = self.affine_predictor(b_z, self.b_predictor_coeffs)
        h = self.affine_predictor((-b_z ** 2) / (4 * a_z) + c_z, self.h_predictor_coeffs)
        a, b, c = self.get_z_coeffs_from_b_h_t0(b, h, t0)
        return mx1, cx1, my1, cy1, a, b, c

    def get_target_from_post_bounce_curve(self, mx, cx, my, cy, az, bz, cz):
        """
        Interesects the given curve with the plane y = self.paddle_y.
        """
        t0 = (self.paddle_y - cy) / my
        x = mx * t0 + cx
        z = az * t0 * t0 + bz * t0 + cz
        return np.array([x, self.paddle_y, z])

    def predict_goal_state(self, posthit_segment_frames: List[ProcessedFrame]) -> np.ndarray:
        """
        Predicts the goal state of the ball by interpolating a parabola based on the given points,
        and using the linear regressor defined above if the ball has not yet bounced.
        """
        pre_bounce_frames, post_bounce_frames = split_posthit_segment_on_bounce(posthit_segment_frames)
        if post_bounce_frames:
            if len(post_bounce_frames) < self.min_window_size_post:
                return self.prev_target_position
            post_bounce_coeffs = self.get_coefficients(post_bounce_frames)
        else:
            if len(pre_bounce_frames) < self.min_window_size_pre:
                return self.prev_target_position
            pre_bounce_coeffs = self.get_coefficients(pre_bounce_frames)
            post_bounce_coeffs = self.get_new_coeffs_from_b_h(*pre_bounce_coeffs)
        target = self.get_target_from_post_bounce_curve(*post_bounce_coeffs)
        self.prev_target_position = target
        return target
```