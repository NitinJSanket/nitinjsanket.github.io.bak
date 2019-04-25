---
layout: default
title: "What is an IMU?"
permalink: tutorials/attitudeest/imu
sitemap: 
  exclude: 'yes' 
---

## Table of Contents:
- [What is an IMU?](#whatisanimu)
- [Complementary Filter](#cf)


<a name='whatisanimu'></a>

## What is an IMU?


An IMU is the abbreviation for **Inertial Measurement Unit**. It generally comes packaged in two flavors: 6-DoF and 9-DoF, i.e., six degrees of freedom or nine degrees of freedom.

The first component of an IMU is called the **Gyroscope** or **Gyro** and it measures the angular velocity across an axis. So, you would need 3 gyros to compute angles in 3D. The best part about a gyro is that it is not affected by external forces and acceleration. Gyros work very well under dynamic conditions when rotational velocities are high, however they drift significantly with regard to time. Hence, the simplest filtering operation perfromed on gyro data is a high pass filter to remove low frequency drift. 

The second component of an IMU is called the **Accelerometer** or **Acc** and it measures the effective acceleration along an axis.  So, you would need 3 acc to compute angles in 3D given information about external forces. Acc are affected by vibration and other external forces and hence cannot be directly used for computing angles/attitude accurately. However, an acc works well in static conditions as opposed to the gyro. Hece, the simplest filtering operation perfromed on acc data is a low pass filter to remove dynamic noise from vibrations and other external factors. 

Also, note that the readings from an IMU are greatly affected by temparature changes. The noise changes with temparature and as the IMU is being used to "heating up" of the sensor. 

A combination of 3 gyros and 3 acc (one in each of $$X, Y, Z$$ axis) is called a 6-DoF IMU. 

A 9-DoF IMU also includes a 3-axis magnetometer which measures Earth's magnetic field which can be used to obtain orientation information. However, for indoor operation which has a lot of metal structures, the magnetometer is generally inaccurate as is exluded from the data fusion. The magnetometer is also often called the **Digital Compass** as it can be directly used to compute the North pole direction. 

<a name='cf'></a>

## Complementary Filter for Attitude Estimation


The simplest  way to estimate angles/attitude/orientation of the IMU in world is by fusing data from gyros and acc using a **Complementary Filter**. 

Like we mentioned before, we high pass the gyro data and low pass the acc data. 

Let $$\mathbf{\omega} = [\omega_x, \omega_y, \omega_z]^T$$ and $$\mathbf{a} = [a_x, a_y, a_z]^T$$ denote the raw measured data from the gyros and acc respectively. 

The low pass filtered acc data is given by

$$\mathbf{\hat{a}}_{t+1} = (1-\alpha)\mathbf{a}_{t+1} + \alpha \mathbf{\hat{a}}_{t}$$


The high pass filtered gyro data is given by

$$\mathbf{\hat{a}}_{t+1} = (1-\alpha)\mathbf{\hat{a}}_{t} + (1-\alpha)(\mathbf{a}_{t+1} - \mathbf{a}_{t})$$

Here the $\hat{x}$ denotes the filtered version of $x$ and $t$ denotes time sample and $\alpha$ controls the frequency boundary where to switch from trusting the acc to trusting the gyros. 

$$\alpha$$ can be chosen as $$\alpha = \frac{\tau}{\tau + dt}$$ where $$\tau$$ is the desired time constant - how fast you want the readings to respond and $$dt = f_s^{-1}$$ is the inverse of sampling frequency $$f_s$$. Generally $$\alpha > 0.5$$ is used. 

The final equation for fusing gyro and acc data into a complementary filter is given below

$$ Ang_{t+1} = (1 - \alpha)(Ang_t + \mathbf{\omega}_{t+1}) + \alpha\mathbf{a}_{t+1}$$
 
Here the gyro data is integrated to obtain angles.
