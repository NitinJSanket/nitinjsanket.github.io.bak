---
layout: default
title: "Mahony Filter"
permalink: tutorials/attitudeest/mahony
sitemap: 
  exclude: 'yes' 
---

## Table of Contents:

- [Mathematical Model of an IMU](#mathimu)
- [Mahony Filter](#mahonyfilt)


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

<a name='mahonyfilt'></a>

## Mahony Filter

<br>
Before we start talking about the mahony filter formulation, let us formally define coordinate axes we will use. Let the letters $$I, W, B$$ denote inertial, world and body frames respectively. Generally $$B$$ and $$I$$ are the same but they don't have to be. A pre-subscript denotes the source coordinate frame and a pre-superscript denotes the destination coordinate frame. For eg., $${}^{B}_{A}X$$ transforms $$X$$ from coordinate frame $$A$$ to $$B$$. If only a pre-superscript is present, it means that the quantity was measured and is represented in the same coordinate frame represented by the pre-superscript.

The desired output is to estimate the attitude/angle/orientation of the IMU sensor in the world frame, i.e., estimating $${}^{W}_{I}\mathbf{q}$$. We use $$\mathbf{q}$$ to denote the orientation represented in the form of a **Quaternion**. If you don't know much about quaternions, I would highly recommend to read [this Wikipedia article](https://en.wikipedia.org/wiki/Quaternion) on quaternions.  

The Mahony filter is a glorified [Complementary Filter](tutorials/attitudeest/imu) with significant improvements to accuracy without significant markup in computation time. Even today, it remains to be one of the most popular filters used in racing quadrotors where time is money, only to be bettered by the [Madgwick Filter](madgwickfilt) with comparable computation time and slightly better accuracy. 

The Mahony filter is a Complementary filter which respects the manifold transformations in quaternion space. The general idea of the Mahony filter is to estimate the attitde/angle/orientation $${}^{I}_{W}\mathbf{q}_{t+1}$$ by fusing/combining attitude estimates by integrating gyro measurements $${}^{I}_{W}\mathbf{q}_{\omega}$$ and direction obtained by the accelerometer measurements. An orientation error from previous step is first calculated to which a correction step based on a Proportional-Integral (PI) compensator is applied to correct the measured angular velocity. This angular velocity is propagated on the quaternion manifold and integrated to obtain the estimate of the attitude.

Following are the steps for attitude estimation using a Mahony filter (Refer to Fig. 1 shown below for an overview of the algorithm).


<div class="fig fighighlight">
  <img src="https://nitinjsanket.github.io/assets/img/tutorials/MadgwickFilterOverview.png" width="100%">
  <div class="figcaption">
  	Fig 1: Overview of Madgwick Filter.
  </div>
  <div style="clear:both;"></div>
</div> <br>

- **Step 1: Obtain sensor measurements**<br> Obtain gyro and acc measurements from the sensor. Let $${}^I\omega_t$$ and $${}^I\mathbf{a}_t$$ denote the gyro and acc measurements respectively. Also, $${}^I\mathbf{\hat{a}}_t$$ denotes the normalized acc measurements. 

- **Step 2: Orientation error using Acc Measurements**<br> Compute orientation error from previous estimate using acc measurements. <br>
<center> $$ \mathbf{v}\left( {}^{I}_{W}\mathbf{\hat{q}}_{est, t} \right) = \begin{bmatrix}  
2\left( q_2q_4 - q_1q_3\right) \\
2\left( q_1q_2 + q_3q_4\right)\\
\left( q_1^2 - q_2^2 -q_3^2 + q_4^2 \right) \\
\end{bmatrix} $$ <br>
$$ \mathbf{e}_{t+1} =  {}^I\mathbf{\hat{a}}_{t+1} \times \mathbf{v}\left( {}^{I}_{W}\mathbf{\hat{q}}_{est, t} \right) $$ <br> 
$$ \mathbf{e}_{i, t+1} = \mathbf{e}_{i, t} + \mathbf{e}_{t+1} \Delta t$$ <br>
</center>	
Here, \\(\Delta t\\) is the time elapsed between two samples at \\(t\\) and \\(t+1\\). 


- **Step 3: Update Gyro Measurements using PI Compensation (Fusion)** <br> Compute updated gyro measurements after the application of the PI compensation term (sensor fusion using feedback).<br>
<center> 
$$ {}^{I}\mathbf{\omega}_{t+1} = {}^{I}\mathbf{\omega}_{t+1} + \mathbf{K}_p\mathbf{e}_{t+1} + \mathbf{K}_i\mathbf{e}_{i, t+1} $$ <br>
</center> 

- **Step 4: Orientation increment from Gyro** <br> Compute orientation increment from gyro measurements.<br>
<center> $$
{}^{I}_{W}\mathbf{\dot{q}}_{\omega,t+1} = \frac{1}{2} {}^{I}_{W}\mathbf{\hat{q}}_{est,t}\otimes \begin{bmatrix} 0, {}^{I}\omega_{t+1} \end{bmatrix}^T $$ <br>
</center> 
Here, \\(\otimes\\) denotes quaternion multiplication. 


- **Step 5: Numerical Integration** <br> Compute orientation by integrating orientation increment.<br>
<center> $$
{}^{I}_{W}\mathbf{q}_{est,t+1} = {}^{I}_{W}\mathbf{\hat{q}}_{est,t} + {}^{I}_{W}\mathbf{\dot{q}}_{\omega,t+1} \Delta t  $$ <br>
</center> 

In a Mahony filter, the only tunable parameters are the PI compensator gians $$\mathbf{K}_p$$ and $$\mathbf{K}_i$$. Also, the user needs to specify the initial estimates of the attitude, biases and sampling time. The initial attitude can be assumed to be zero if th device is at rest or it has to be obtained by external sources such as a motion capture system or a camera. The bias is computed by taking an average of samples with the IMU at rest and computing the mean value. Note that this bias changes over time and the filter will start to drift over time. The sampling time is the inverse of the operating frequency of the IMU and is specified generally at the driver level.