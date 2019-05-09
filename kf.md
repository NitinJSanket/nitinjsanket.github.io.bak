---
layout: default
title: "Kalman Filter"
permalink: tutorials/attitudeest/kf
sitemap: 
  exclude: 'yes' 
---

## Table of Contents:

- [Mathematical Model of an IMU](#mathimu)
- [Kalman Filter](#kf)
	- [Assumptions](#assumptions)
	- [Mathematical Formulation](#mathformulation)
- [References](#ref)


<a name='mathimu'></a>

## Mathematical Model of an IMU

<br>
If you don't know what an IMU is, I would recommend going through my [What is an IMU? tutorial](tutorials/attitudeest/imu).

Let us assume that our IMU is a 6-DoF one, i.e., it has a 3 axis gyro and a 3 axis acc. A 9-DoF IMU is commonly called MARG (Magnetic, Angular Rate and Gravity) sensor. A simple mathematical model of the gyro and acc is given below.

Gyroscope Model:<br>

$$ \omega = \hat{\omega} + \mathbf{b}_g + \mathbf{n}_g $$

Here, $$\omega$$ is the measured angular velocity from the gyro, $$\hat{\omega}$$ is the latent ideal angular velocity we wish to recover, $$\mathbf{b}_g$$ is the gyro bias which changes with time and other factors like temparature, $$\mathbf{n}_g$$ is the white gaussian gyro noise.

The gyro bias is modelled as $$ \mathbf{\dot{b}}_g = \mathbf{b}_{bg}(t) \sim \mathcal{N}(0, Q_g) $$ where $$ Q_g$$ is the covariance matrix which models gyro noise. 


Accelerometer Model:<br>

$$ \mathbf{a} = R^T(\mathbf{\hat{a}} - \mathbf{g}) + \mathbf{b}_a + \mathbf{n}_a $$

Here, $$\mathbf{a}$$ is the measured acceleration from the acc, $$\mathbf{\hat{a}}$$ is the latent ideal acceleration we wish to recover, $$R$$ is the orientation of the sensor in the world frame, $$\mathbf{g}$$ is the acceleration due to gravity in the world frame, $$\mathbf{b}_a$$ is the acc bias which changes with time and other factors like temparature, $$\mathbf{n}_a$$ is the the white gaussian acc noise.

The acc bias is modelled as $$ \mathbf{\dot{b}}_a = \mathbf{b}_{ba}(t) \sim \mathcal{N}(0, Q_a) $$ where $$ Q_a$$ is the covariance matrix which models acc noise. 

Here the orientation of the sensor is either known from external sources such as a motion capture system or a camera or estimated by sensor fusion. 

<a name='kf'></a>

## Kalman Filter

<br>
Before we start talking about the Kalman Filter (KF) formulation, let us formally define coordinate axes we will use. Let the letters $$I, W, B$$ denote inertial, world and body frames respectively. Generally $$B$$ and $$I$$ are the same but they don't have to be. A pre-subscript denotes the source coordinate frame and a pre-superscript denotes the destination coordinate frame. For eg., $${}^{B}_{A}X$$ transforms $$X$$ from coordinate frame $$A$$ to $$B$$. If only a pre-superscript is present, it means that the quantity was measured and is represented in the same coordinate frame represented by the pre-superscript.

The desired output is to estimate the attitude/angle/orientation of the IMU sensor in the world frame, i.e., estimating $$[\phi, \theta, \psi]^T$$ which is commonly called **Roll, Pitch** and **Yaw** respectively. These are commonly interchanged with **Euler Angles**. However they **ARE NOT THE SAME**. Euler Angles can vary in convention and is generally chosen from 12 unique combinations. For our discussion we'll use Z-Y-X Euler Angles which we'll also refer to as Roll, Pitch and Yaw for X, Y and Z axes respectively. For a detailed explanation [refer to this awesome explanation by Peter Corke](https://petercorke.com/wordpress/roll-pitch-yaw-angles).

In any of the filters we looked at before there was a tradeoff parameter which determined when the filter should trust which sensor more (gyro or acc). However, these parameters didn't have much physical significance and is hard to tune. Also, it seems that changing these parameters at every time instant would yield the best result. A **Kalman Filter** (KF) does this in a theoretically optimal fashion. <br> 

A KF formulates this problem (state estimation or attitude estimation in our case) as minimizing a quadratic cost function with respect to the latent correct space and the estimated space. This cost function includes the sensor noise (how much should you trust each sensor) as well as the underlying dynamics of the system (is the IMU placed on a car/quadrotor/hand-held). What if you don't really know where the sensor is going to be used? The answer is simple - you craft a generic enough system dynamics model which would work "well" in most scenarios. <br>

The aim of a KF is to estimate a **state** (a vector of time varying quantities) given the data from one or more sensors and the knowledge of a process model/system dynamics ("how" the system is moving). The magic of a Kalman filter is that it dynamically weights the estimates from both the process model and sensor measurements. Note that the state could have variables not of all which can be measured like the bias of a gyroscope in our case. This can still be used in the process update. Such a state when not all the variables are not obeserved are called **Augmented State**. <br>

A KF operates in two steps, i.e., **process update** and **measurement update**. In the process update, the filter uses measurements from a sensor and underlying system dynamics to predict the future state. This is the best you can do withou a measurement update and the estimated state would drift over time. This drift is directly proportional to the amount of error in the process model (how accurately does your process model resemble the real world?). <br>

In the measurement update, the filter uses measurements from another sensor (hopefully complementary in error to that used in process model) to correct for errors in predicted state. However, none of the sensors used are perfect, how do you trust one more than the other? Simple, you only model the noise charactersistics of each sensor, i.e., the designer's opinion of accuracy of these sensors. You can obtain this from the manufacturer's datasheet or by experimentally obtaining these values. <br>

Now, let's look at the assumptions a Linear Kalman Filter or Kalman filter formulation makes to obtain the mathematical model.

<a name='assumptions'></a>

### Assumptions
- All the noise in the system (process noise and measurement noise) is additive white Gaussian noise
- The prior state is modelled by a Gaussian distribution
- Both the process and measurement model is linear
- Markov Property: The future state of the system is conditionally independent of the past states given the current state

<a name='mathformulation'></a>

### Mathematical Formulation

<br>
Now let's look at the mathematical formulation of a Kalman Filter. 

The filter starts by taking as input the current state to predict the future state. Now, you might be wondering what a state is? As discussed before, a state in a Kalman filter is a vector which you would like to estimate. In our case, we would like to estimate the attitude of the IMU. Along with estimating the attitude we would also like to estimate the bias of the gyro so that we could get more accurate estimation. Let us denote our state at time $$t$$ by $$\mathbf{x}_t$$ and is given by <br>

<center>
$$
\mathbf{x}_t = \begin{bmatrix}
\phi_t \\ 
\theta_t \\ 
\psi_t \\
\mathbf{b}_{g,t}
\end{bmatrix}
$$
</center>
Here, $$\mathbf{b}_{g,t} \in \mathbb{R}^{3 \times 1} $$ denotes the gyro bias in 3D. 

Following are the steps for attitude estimation using a Kalman filter.

- **Step 1: Obtain sensor measurements**<br> Obtain gyro and acc measurements from the sensor. Let $${}^I\omega_t$$ and $${}^I\mathbf{a}_t$$ denote the gyro and acc measurements respectively.


- **Step 2: Process Update using Gyro Measurements (Prediction)**<br> Compute the predicted next state using the system dynamics. Note that in a KF each state is characterized by its mean and co-variance matrix.<br>
<center> $$ 
\hat{\mu_{t+1}} = \mathbf{A}_{t+1}\hat{\mu_{t}} + \mathbf{B}_{t+1}\mathbf{u}_{t+1}$$ <br>
$$
\hat{\Sigma_{t+1}} = \mathbf{A}_{t+1}\hat{\Sigma_{t}}\mathbf{A}_{t+1}^T + \mathbf{Q}_{t+1}$$
 <br></center>

Here, $$ \mu_t$$, $$\Sigma_t$$ denote the mean and co-variance of the state at time $$t$$ and $$\hat{\mu_{t+1}}, \hat{\Sigma_{t+1}}$$ denotes the estimated mean and co-variance of the state at time $$t+1$$. $$\mathbf{Q}_{t+1}$$ denotes the noise matrix modelling how noisy the system dynamics model is. Here, $$\mathbf{A}_{t+1}$$ denotes the **process/dynamics/system model** which mathematically models how the state changes from $$t$$ to $$t+1$$ and is given below. <br>

<center>
$$
\mathbf{A}_{t+1} = \begin{bmatrix}
1 & 0 & 0 & -\Delta t & 0 & 0\\
0 & 1 & 0 & 0 & -\Delta t & 0\\
0 & 0 & 1 & 0 & 0 & -\Delta t\\
0 & 0 & 0 & 1 & 0 & 0\\
0 & 0 & 0 & 0 & 1 & 0\\
0 & 0 & 0 & 0 & 0 & 1\\
\end{bmatrix}
$$ <br>
</center>

Here, \\(\Delta t\\) is the time elapsed between two samples at \\(t\\) and \\(t+1\\). If no correction is given this prediction would drift due to error in process model. The process model in our case models a constant attitude within the small time instant but the bias integrates over this small time $$\Delta t$$. Also, $$\mathbf{u}_{t+1}$$ represents the input/process vector (in our case this is the vector of euler angle velocities of the IMU in world frame) and is given by <br> 

<center>
$$
\mathbf{u}_{t+1} = \begin{bmatrix}
\dot{\phi}\\
\dot{\theta}\\
\dot{\psi}\\
\end{bmatrix}
$$<br>
$$
\begin{bmatrix}
\dot{\phi}\\
\dot{\theta}\\
\dot{\psi}\\
\end{bmatrix} = \mathbf{R}^{-1} {}^I\omega_t  
$$ <br>
$$
\mathbf{R} = \begin{bmatrix}
\cos \theta & 0 & -\cos \phi \sin \theta \\
0 & 1 & \sin \phi \\
\sin \theta & 0 & \cos \phi \cos \theta
\end{bmatrix}
$$
<br>
</center>

$$\mathbf{B}_{t+1}$$ denotes the mapping of the input vector to the state vector and is given by <br>

<center>
$$
\mathbf{B}_{t+1} = \begin{bmatrix}
\Delta t & 0 & 0\\
0 & \Delta t & 0\\
0 & 0 & \Delta t\\
0 & 0 & 0\\
0 & 0 & 0\\
0 & 0 & 0\\
\end{bmatrix}
$$<br>
</center>

Here, the bias is assumed to be not dependent on the attitude which might not be true in real life.<br>

- **Step 3: Measurement Update using Acc Measurements (Fusion or Correction)** <br> 
In this step, compute the attitude using acc measurements and use it to obtain the corrected state, i.e., state which is a combination of both the process and measurement steps. This step entails the sensor fusion. <br>

<center>
$$
\mathbf{K}_{t+1} = \hat{\Sigma_{t+1}}\mathbf{C}^T \left( \mathbf{C} \hat{\Sigma_{t+1}} \mathbf{C}^T + \mathbf{R} \right)^{-1}
$$<br>
$$
\mu_{t+1} = \hat{\mu_{t+1}} + \mathbf{K}_{t+1} \left( \mathbf{z}_{t+1} - \mathbf{C} \hat{\mu_{t+1}}\right)
$$<br>
$$
\Sigma_{t+1} = \hat{\Sigma_{t+1}} - \mathbf{K}_{t+1}\mathbf{C}\hat{\Sigma_{t+1}}
$$<br>
</center>

Here, $$\mathbf{z}_{t+1}$$ denotes the observable state by the sensor (this could be a subset of the full state as in our case). The acc is used to obtain the angles as follows <br>

<center>
$$
\phi = \tan^{-1}\left( \frac{a_y}{\sqrt{a_x^2 + a_z^2}} \right)
$$ <br>
$$
\theta = \tan^{-1}\left( \frac{a_x}{\sqrt{a_y^2 + a_z^2}} \right)
$$ <br>
<br>
</center>

Here, $$\mathbf{C}$$ denotes the mapping from observed state to full state and is given by <br>

<center>
$$
\mathbf{C} = \begin{bmatrix}
1 & 0 & 0 & 0 & 0 & 0\\
0 & 1 & 0 & 0 & 0 & 0\\
0 & 0 & 0 & 0 & 0 & 0\\
0 & 0 & 0 & 0 & 0 & 0\\
0 & 0 & 0 & 0 & 0 & 0\\
0 & 0 & 0 & 0 & 0 & 0\\
\end{bmatrix}
$$ <br>
</center>

Note that, in our case we don't use the value of $$\psi$$ from the acc readings as it is generally inaccurate. In a real robotic system, the value of $$\psi$$ is obtained from a camera or a compass. The rows of all zeros in $$\mathbf{C}$$ indicate unobservable values in the state vector $$\mathbf{x}$$. 

**Repeat steps 1 and 2 for every time instant and step 3 whenever a measurement update from acc is available.** The measurement update is generally run about a factor of magnitude slower than the process update for keeping computation complexity low. <br>


<a name='ref'></a>

## References

- [IMU Attitude Estimation](http://philsal.co.uk/projects/imu-attitude-estimation)
- [MEAM620 Kalman Filter Notes](https://alliance.seas.upenn.edu/~meam620/wiki/index.php?n=Main.Schedule2015?action=download&upname=2015_kalmanFilter.pdf)
- Nitin J. Sanket. [Orientation Tracking based Panorama Stiching using Unscented Kalman Filter.](https://github.com/NitinJSanket/ESE650Project2/blob/master/Report/ESE650Project2.pdf)
- [Does anyone have a 6-DOF IMU Kalman Filter?](https://www.researchgate.net/post/Does_anyone_have_a_6-DOF_IMU_Kalman_Filter)
