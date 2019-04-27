---
layout: default
title: "Madgwick Filter"
permalink: tutorials/attitudeest/madgwick
sitemap: 
  exclude: 'yes' 
---

## Table of Contents:

- [Mathematical Model of an IMU](#mathimu)
- [Madgwick Filter](#madgwickfilt)


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

<a name='madgwickfilt'></a>

## Madgwick Filter

<br>
Before we start talking about the madgwick filter formulation, let us formally define coordinate axes we will use. Let the letters $$I, W, B$$ denote inertial, world and body frames respectively. Generally $$B$$ and $$I$$ are the same but they don't have to be. A pre-subscript denotes the source coordinate frame and a pre-superscript denotes the destination coordinate frame. For eg., $${}^{B}_{A}X$$ transforms $$X$$ from coordinate frame $$A$$ to $$B$$. If only a pre-superscript is present, it means that the quantity was measured and is represented in the same coordinate frame represented by the pre-superscript.

The desired output is to estimate the attitude/angle/orientation of the IMU sensor in the world frame, i.e., estimating $${}^{W}_{I}\mathbf{q}$$. We use $$\mathbf{q}$$ to denote the orientation represented in the form of a **Quaternion**. If you don't know much about quaternions, I would highly recommend to read [this Wikipedia article](https://en.wikipedia.org/wiki/Quaternion) on quaternions.  

The Madgwick filter is a glorified [Complementary Filter](tutorials/attitudeest/imu) with significant improvements to accuracy without significant markup in computation time. Even today, it remains to be one of the most popular filters used in racing quadrotors where time is money. 

The Madgwick filter formulates the attitude estimation problem in quaternion space. The general idea of the Madgwick filter is to estimate $${}^{I}_{W}\mathbf{q}_{t+1}$$ by fusing/combining attitude estimates by integrating gyro measurements $${}^{I}_{W}\mathbf{q}_{\omega}$$ and direction obtained by the accelerometer measurements. In essence, the gyro estimates of attitude are used as accurate depictions in a small amount of time and faster movements and the acc estimates of attitude are used as accurate directions to compensate for long term gyro drift by integration. 

As in [Complementary Filter](tutorials/attitudeest/imu), the attitude is estimated from the gyro by numerical integration. The attitude estimation from the acc is done by using a gradient descent algorithm to solve the following minimzation problem. 

$$
\min_{{}^{I}_{W}\mathbf{\hat{q}} \in \mathbb{R}^{4\times 1}} f\left({}^{I}_{W}\mathbf{\hat{q}}, {}^{W}\mathbf{\hat{g}}, {}^{I}\mathbf{\hat{a}} \right)  
$$ 