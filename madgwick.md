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

If you don't know what an IMU is, I would recommend going through my [What is an IMU? tutorial](tutorials/attitudeest/imu).

Let us assume that our IMU is a 6-DoF one, i.e., it has a 3 axis gyro and a 3 axis acc. A 9-DoF IMU is commonly called MARG (Magnetic, Angular Rate and Gravity) sensor. A simple mathematical model of the gyro and acc is given below.

Gyroscope Model:<br>

$$ \omega = \hat{\omega} + \mathbf{b}_g + \mathbf{n}_g $$

Here, $$\omega$$ is the measured angular velocity from the gyro, $$\hat{\omega}$$ is the latent ideal angular velcoity we wish to recover, $$\mathbf{b}_g$$ is the gyro bias which changes with time and other factors like temparature, $$\mathbf{n}_g$$ is the gyro noise.

The gyro bias is modelled as $$ \mathbf{\dot{b}}_g \sim 
mathcal{N}(0, Q_g) $$ where $$ Q_g$$ is the covariance matrix which models gyro noise. 