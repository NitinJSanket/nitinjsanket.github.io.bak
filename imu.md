---
layout: default
title: "What is an IMU?"
permalink: tutorials/attitudeest/imu
sitemap: 
  exclude: 'yes' 
---

## Table of Contents:

- [What is an IMU?](#whatisanimu)
- [Mathematical Model of an IMU](#mathimu)
- [Isn't Attitude Estimation trivial?](#trivial)
- [Complementary Filter](#cf)
- [References](#ref)


<a name='whatisanimu'></a>

## What is an IMU?

<br>
An IMU is the abbreviation for **Inertial Measurement Unit**. It generally comes packaged in two flavors: 6-DoF and 9-DoF, i.e., six degrees of freedom or nine degrees of freedom.

The first component of an IMU is called the **Gyroscope** or **Gyro** and it measures the angular velocity across an axis. So, you would need 3 gyros to compute angles in 3D. The best part about a gyro is that it is not affected by external forces and acceleration. Gyros work very well under dynamic conditions when rotational velocities are high, however they drift significantly with regard to time. Hence, the simplest filtering operation perfromed on gyro data is a high pass filter to remove low frequency drift. 

The second component of an IMU is called the **Accelerometer** or **Acc** and it measures the effective acceleration along an axis.  So, you would need 3 acc to compute angles in 3D given information about external forces. Acc are affected by vibration and other external forces and hence cannot be directly used for computing angles/attitude accurately. However, an acc works well in static conditions as opposed to the gyro. Hence, the simplest filtering operation perfromed on acc data is a low pass filter to remove dynamic noise from vibrations and other external factors. 

Also, note that the readings from an IMU are greatly affected by temparature changes. The noise changes with temparature and as the IMU is being used due to "heating up" of the sensor. 

A combination of 3 gyros and 3 acc (one in each of $$X, Y, Z$$ axis) is called a 6-DoF IMU. 

A 9-DoF IMU also includes a 3-axis magnetometer which measures Earth's magnetic field which can be used to obtain orientation information as well. However, for indoor operation which has a lot of metal structures, the magnetometer is generally inaccurate as is exluded from the data fusion. The magnetometer is also often called the **Digital Compass** as it can be directly used to compute the North pole direction. 

<a name='mathimu'></a>

## Mathematical Model of an IMU

<br>

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

<a name='trivial'></a>

## Isn't Attitude Estimation trivial?

<br>
You might be thinking if the gyro gives angular acceleration wouldn't attitude estimation be as trivial as integrating these measurements. You would be right! In an **ideal** world this would work perfectly and we would have the perfect attitude. However, real world is very different and the gyro has both an unknown changing bias and noise. Even if you want the system to work for a small amount of time, say 4 sec where you could assume that the bias doesnt change and can be obtained as averaging measurements when the gyro is at rest. The noise though being small accumulates over time when integrated eventually becoming unbounded in magnitude completely masking the signal. 

Keen readers might think, why cant we just low pass filter this signal and then integrate? This is a good idea but would cause a significant lag in the measurements and alter the magnitudes of the high frequency signals where gyro works well. This is no good for us. Also, note that integtation works as a form of [low pass filtering](https://webhome.phy.duke.edu/~schol/phy271/faqs/faq8/node6.html). 

Next, one might be thinking why cant we use acc measurements to estimate attitude when only gravity is acting on it? The answer is yes you can do it in an **ideal world**. In reality, the measurements are too noisy and don't work well when the device is in motion or for high speed motion. Hence necessitating fusing both these sensors. The gyro works well at higher frequency motion (fast movements but drifts over time) while the acc works well at lower frequency motion (works well over long period of time but is inaccurate for fast motion). The gyro and acc have complementary errors, i.e., both are errnoeous but in different bands of the frequency spectrum and hence can be fused to get much more accurate measurements then one could ever get with either of the sensors separately. Hence the name **Complementary Filter** which is described next. 

<a name='cf'></a>

## Complementary Filter for Attitude Estimation

<br>
The simplest  way to estimate angles/attitude/orientation of the IMU in world is by fusing data from gyros and acc using a **Complementary Filter**. 

Like we mentioned before, we high pass the gyro data and low pass the acc data. 

Let $$\mathbf{\omega} = [\omega_x, \omega_y, \omega_z]^T$$ and $$\mathbf{a} = [a_x, a_y, a_z]^T$$ denote the raw measured data from the gyros and acc respectively. 

The low pass filtered acc data is given by

$$\mathbf{\hat{a}}_{t+1} = (1-\alpha)\mathbf{a}_{t+1} + \alpha \mathbf{\hat{a}}_{t}$$


The high pass filtered gyro data is given by

$$\mathbf{\hat{\omega}}_{t+1} = (1-\alpha)\mathbf{\hat{\omega}}_{t} + (1-\alpha)(\mathbf{\omega}_{t+1} - \mathbf{\omega}_{t})$$

Here the $\hat{x}$ denotes the filtered version of $x$ and $t$ denotes time sample and $\alpha$ controls the frequency boundary where to switch from trusting the acc to trusting the gyros. 

$$\alpha$$ can be chosen as $$\alpha = \frac{\tau}{\tau + dt}$$ where $$\tau$$ is the desired time constant - how fast you want the readings to respond and $$dt = f_s^{-1}$$ is the inverse of sampling frequency $$f_s$$. Generally $$\alpha > 0.5$$ is used. 

The final equation for fusing gyro and acc data into a complementary filter is given below

$$ Ang_{t+1} = (1 - \alpha)(Ang_t + \mathbf{\omega}_{t+1}dt) + \alpha\mathbf{a}_{t+1}$$
 
Here the gyro data is integrated to obtain angles.

- Why you cant just integrate gyros?
- Why you cant rely just on acc?

<a name='ref'></a>

## References

- [My IMU estimation experience blog](https://sites.google.com/site/myimuestimationexperience/filters/complementary-filter)
- [IMU Attitude Estimation](http://philsal.co.uk/projects/imu-attitude-estimation)