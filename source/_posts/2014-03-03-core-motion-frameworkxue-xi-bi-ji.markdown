---
layout: post
title: "Core Motion Framework学习笔记"
date: 2014-03-03 15:22:55 +0800
comments: true
categories: iOS
---

本文是个人学习Core Motion Framework后所做的笔记，主要关注加速度传感器和陀螺仪数据，其他方面暂未涉及。

###Core Motion Framework简介

如果你只是想获取手机的大致朝向（比如横屏竖屏）请使用UIDevice，如果你只是想监测Shake事件请使用Shake-Motion Events，只有你需要访问加速度传感器和陀螺仪的数据时才需要使用Core Motion框架，该框架提供传感器采集到的原始数据以及经过算法加工后的数据（例如消除传感器误差后的数据）。

###传感器介绍

+ **加速度传感器**分别测量沿X、Y、Z三个轴的加速度值，手机上三个轴的指向参见下图。当手机水平朝上静止放置时Z轴上有一个朝下的重力加速度，因为朝向与+Z相反所以加速度为负值，另外框架里加速度的单位为G（重力加速度），所以此时Z轴上的加速度值为-1，而X轴Y轴的加速度值为0。当手机以其他非水平方向静止放置时，重力加速度会分解到XYZ轴上。

->![](/images/2014/03/1.png =380x390)<-

+ **陀螺仪**分别测量沿X、Y、Z三个轴的旋转速度，并通过右手法则判断旋转速度的正负，手机上三个轴的指向以及正负旋转方向参见下图。举例来说当手机水平纵向放置时（Home按钮在底部），翘起头部则在X轴有一个正的旋转速度。另外框架里旋转速度的单位为旋转的弧度每秒。

->![](/images/2014/03/2.png =345x401)<-


###Core Motion Framework介绍

Core Motion Framework的整体结构见下图，其中CMMotionManager是框架的管理类，提供创建实例、设置数据采集频率、开启数据采集、关闭数据采集等等功能。

->![](/images/2014/03/3.png =432x374)<-

CMAccelerometerData实际包含一个结构体acceleration(类型：CMAcceleration)，提供三轴加速度。
    typedef struct {
            double x;
            double y;
            double z;
    } CMAcceleration;
    // A structure containing 3-axis acceleration data. 

CMGyroData实际包含一个结构体rotationRate(类型：CMRotationRate)，提供三轴旋转速度。
    typedef struct {
            double x;
            double y;
            double z;
    } CMRotationRate;
    // A structure containing 3-axis rotation rate data.

CMDeviceMotion包含下面五种数据，其中已经将重力加速度和用户加速度独立开来，手机整体的加速度等于重力加速度和用户加速度之和。
    （1）attitude(类型：CMAttitude)
     // Returns the attitude of the device.  

    （2）rotationRate(类型：CMRotationRate)
     // Returns the rotation rate of the device for devices with a gyro.  

    （3）gravity(类型：CMAcceleration)
     // Returns the gravity vector expressed in the device's reference frame.  

    （4）userAcceleration(类型：CMAcceleration)
     // Returns the acceleration that the user is giving to the device.  

     // 磁力数据不在本文关注范围内
    （5）magneticField(类型：CMCalibratedMagneticField)
     // Returns the magnetic field vector with respect to the device for devices with a magnetometer.

CMAttitude主要包含下面五种数据。手机水平朝上放置时roll值和pitch值都为0，yaw值默认以开始采集时的朝向为0值（注：当使用某些CMAttitudeReferenceFrame时yaw的0值可能指向某个固定方向。），手机当前attitude的roll、pitch、yaw值是指从三个轴的0值经过每个轴上多少弧度的旋转才能形成当前attitude，注意三个轴上都有旋转时优先计算roll轴再pitch轴最后yaw轴。
    @property(readonly, nonatomic) double roll;
     // Returns the roll of the device in radians. 

    @property(readonly, nonatomic) double pitch;
     // Returns the pitch of the device in radians. 

    @property(readonly, nonatomic) double yaw;
     // Returns the yaw of the device in radians. 

    // 下面的旋转矩阵和四元数不在本文关注范围内
    @property(readonly, nonatomic) CMRotationMatrix rotationMatrix;
     // Returns a rotation matrix representing the device's attitude.  

    @property(readonly, nonatomic) CMQuaternion quaternion;
     // Returns a quaternion representing the device's attitude.

###Core Motion Framework开发
**1、创建CMMotionManager实例：**  
一个App只应该创建一个CMMotionManager实例，创建多个实例会影响数据的采集速度。  
**2、选择合适的数据采集方式：**  
**Pull：**App需要时主动从CMMotionManager中获取最新的Motion数据。   
**Push：**App指定数据采集间隔时间并实现处理Motion数据的block，Core Motion会将每次采集的数据传递给block并将block添加到你给定的NSOperationQueue中执行。  
**Pull和Push方式各自适用的场景见下表：**

![](/images/2014/03/4.png)

**3、设置合适的数据采集间隔：**  
在满足App需求的前提下应该设置尽量大的数据采集间隔，数据采集的间隔越大传递给App的事件越少，这样可以节省手机电量。下表列举了通常使用的数据采集频率以及在这些频率下可以实现的功能：

![](/images/2014/03/5.png)

**4、即时关闭数据采集功能：**  
当App不需要再采集Motion数据时应该即时关闭采集功能，这样可以节省手机电量。   
**5、开发示例：**
    - (void)startAccelerometer{
        // Create a CMMotionManager
        mManager = [[CMMotionManager alloc] init];
        // Check whether the accelerometer is available
        if (mManager.isAccelerometerAvailable) {
            // Determine the update interval
            NSTimeInterval updateInterval = 0.02;
            // Assign the update interval to the motion manager
            [mManager setAccelerometerUpdateInterval:updateInterval];
            // pull
            //[mManager startAccelerometerUpdates];
            //NSTimer* timer = [NSTimer scheduledTimerWithTimeInterval:0.1 target:self
            //                    selector:@selector(checkAccelerometer) userInfo:nil repeats:YES];
            //[timer fire];
            // push
            [mManager startAccelerometerUpdatesToQueue:[NSOperationQueue currentQueue]
                withHandler:^(CMAccelerometerData *accelerometerData, NSError *error) {
                    NSLog(@"latest X-axis acceleration:%f", accelerometerData.acceleration.x);
                }];
        }
    }

    - (void)checkAccelerometer{
        CMAccelerometerData* latestData = mManager.accelerometerData;
        NSLog(@"latest X-axis acceleration:%f", latestData.acceleration.x);
    }

    - (void)stopAccelerometer{
        if (mManager.isAccelerometerActive) {
            [mManager stopAccelerometerUpdates];
        }
    }

###参考资料

* [移动设备智能化的基石–从iPhone4的传感器谈起](http://www.kunli.info/2010/07/07/mobile-device-sensor/)
* [iOS4中Core Motion框架的介绍和使用](http://www.kunli.info/2010/07/30/motion/)
* [Event Handling Guide for iOS](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Introduction/Introduction.html)
* [CoreMotion可以测到的各种值](http://blog.csdn.net/kingkong1024/article/details/11605143)
