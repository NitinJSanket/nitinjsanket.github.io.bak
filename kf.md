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
Before we start talking about the kalman filter formulation, let us formally define coordinate axes we will use. Let the letters $$I, W, B$$ denote inertial, world and body frames respectively. Generally $$B$$ and $$I$$ are the same but they don't have to be. A pre-subscript denotes the source coordinate frame and a pre-superscript denotes the destination coordinate frame. For eg., $${}^{B}_{A}X$$ transforms $$X$$ from coordinate frame $$A$$ to $$B$$. If only a pre-superscript is present, it means that the quantity was measured and is represented in the same coordinate frame represented by the pre-superscript.

The desired output is to estimate the attitude/angle/orientation of the IMU sensor in the world frame, i.e., estimating $$[\phi, \theta, \psi]^T$$. 

In any of the filters we looked at before there was a tradeoff parameter which determined when the filter should trust which sensor more (gyro or acc). However, these parameters didn't have much physical significance and is hard to tune. Also, it seems that changing these parameters at every time instant would yeild the best results. A **Kalman Filter** does this in a theoretically optimal fashion. 

A Kalman filter formulates this problem as minimizing a quadratic cost function. This cost function includes the sensor noise (how much should you trust each sensor) as well as the underlying dynamics of the system (is the IMU placed on a car/quadrotor/hand-held). What if you don't really know where the sensor is going to be used? The answer is simple - you craft a generic enough system dynamics model which would work "well" in most scenarios. 

<a name='assumptions'></a>

### Assumptions

<br>
Let's first look at the assumptions a Linear Kalman Filter or a Kalman Filter makes. 
- All the noise in the system (process noise and measurement noise) is additive white Gaussian noise
- The prior state is modelled by a Gaussian distribution
- Both the process and measurement model is linear
- Markov Property: The future state of the system is conditionally independent of the past states given the current state

<a name='mathformulation'></a>

### Mathematical Formulation

<br>
Now let's look at the mathematical formulation of a Kalman Filter. 

The filter starts by taking as input the current state to predict the future state. Now, you might be wondering what a state is? A state in a Kalman filter is a vector which you would like to estimate. In our case, we would like to estimate the attitude of the IMU. Along with estimating the attitude we would also like to estimate the bias of the gyro so that we could get more accurate estimation. 


Following are the steps for attitude estimation using a Kalman filter (Refer to Fig. 1 shown below for an overview of the algorithm).

<div class="fig fighighlight">
  <img src="https://nitinjsanket.github.io/assets/img/tutorials/MadgwickFilterOverview.png" width="100%">
  <div class="figcaption">
  	Fig 1: Overview of Madgwick Filter.
  </div>
  <div style="clear:both;"></div>
</div> <br>

- **Step 1: Obtain sensor measurements**<br> Obtain gyro and acc measurements from the sensor. Let $${}^I\omega_t$$ and $${}^I\mathbf{a}_t$$ denote the gyro and acc measurements respectively.


- **Step 2: Predict the next state**<br> Compute the predicted next state using the system dynamics. <br>
<center> $$ 
\hat{\mu_{t+1}} = \mathbf{A}_{t+1}\hat{\mu_{t}} + \mathbf{B}_t\hat{\mathbf{u}_{t}}$$ <br>
$$
\hat{\Sigma_{t+1}} = \mathbf{A}_{t+1}\hat{\Sigma_{t}}\mathbf{A}_{t+1}^T + \mathbf{Q}_{t+1}$$
$$ <br></center>	
Here, $$ \mu_t$$, $$\Sigma_t$$ denote the mean and co-variance of the state at time $$t$$ and $$\hat{\mu_{t+1}}, \hat{\Sigma_{t+1}}$$ denotes the estimated mean and co-variance of the state at time $$t+1$$. $$\mathbf{Q}_{t+1}$$ denotes the noise matrix modelling how noisy the system dynamics model is. Here, $$\mathbf{A}_{t+1}$$ denotes the **process/dynamics/system model** which mathematically models how the state changes from $$t$$ to $$t+1$$. If no correction is given this prediction would drift due to error in process model. 

Now, let's look at how these vectors and matrices look for our particular case of attitude estimation. <br>

Let the state vector be given by<br>
<center>
$$
\mathbf{x} = \begin{bmatrix}
\phi \\
\theta \\ 
\psi \\
\mathbf{b}_g
\end{bamtrix}
$$ <br>
</center>
Here, $$\mathbf{b}_g \in \mathbb{R}^{3 \times 1} $$ denotes the gyro bias in 3D. 

- **Step 2 (b): Orientation increment from Gyro** <br> Compute orientation increment from gyro measurements (numerical integration).<br>
<center> $$
{}^{I}_{W}\mathbf{\dot{q}}_{\omega,t+1} = \frac{1}{2} {}^{I}_{W}\mathbf{\hat{q}}_{est,t}\otimes \begin{bmatrix} 0, {}^{I}\omega_{t+1} \end{bmatrix}^T $$ <br>
</center> 

Look at blue parts in Fig. 1.


- **Step 3: Fuse Measurements** <br> Fuse the measurments from both the acc and gyro to obtain the estimated attitude $$ {}^{I}_{W}\mathbf{\hat{q}}_{est, t+1}$$. <br>
<center> 
$$
{}^{I}_{W}\mathbf{\dot{q}}_{est, t+1} = {}^{I}_{W}\mathbf{\dot{q}}_{\omega, t+1} + {}^{I}_{W}\mathbf{q}_{\nabla, t+1} 
$$ <br>
$$ {}^{I}_{W}\mathbf{q}_{est, t+1} = {}^{I}_{W}\mathbf{\hat{q}}_{est, t+1} + {}^{I}_{W}\mathbf{\dot{q}}_{est, t+1} \Delta t 
$$  </center> <br>
Here, \\(\Delta t\\) is the time elapsed between two samples at \\(t\\) and \\(t+1\\). Look at red parts in Fig. 1.

**Repeat steps 1 to 3 for every time instant.** <br>


<a name='ref'></a>

## References

- Sebastian OH Madgwick, Andrew JL Harrison, and Ravi Vaidyanathan. [Estimation of IMU and MARG orientation using a gradient descent algorithm.](https://ieeexplore.ieee.org/document/5975346) 2011 IEEE international conference on rehabilitation robotics. IEEE, 2011.