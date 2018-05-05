---
title: 基于LeapMotion和S曲线的飞行机器人及机械臂系统
date: 2016-11-17 09:41:38
tags:
categories: DIY
---
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Model_Render.jpg)
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Drone_with_Robotic_Arm-1.JPG)
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Drone_with_Robotic_Arm-2.JPG)
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Drone_with_Robotic_Arm-3.JPG)
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Drone_with_Robotic_Arm-4.JPG)
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Drone_with_Robotic_Arm-5.JPG)
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Drone_with_Robotic_Arm-6.JPG)
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Drone_with_Robotic_Arm-7.JPG)
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Drone_with_Robotic_Arm-8.JPG)
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Drone_with_Robotic_Arm-9.JPG)
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Drone_with_Robotic_Arm-10.JPG)
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Drone_with_Robotic_Arm-11.JPG)

总体来说这个项目分为四大块儿：
+ 飞行平台搭建
+ 机械臂的设计制作
+ 机械臂控制算法及界面的开发
+ 动力驱动及通信装置硬件部分的设计制作

飞行器部分就不多说了，本就是以其为平台:
![](/images/基于LeapMotion和S曲线的飞行机器人及机械臂系统/Drone_with_Robotic_Arm_System_Specification.jpg)
机械臂参考了一些小型机械臂的结构类型，6自由度：
<video controls="controls" autoplay=autoplay loop=loop style="clear: both;display: block;margin-left: auto; margin-right: auto;max-width: 80%;">
  <source src="http://yunshuixin-cn-video.oss-cn-beijing.aliyuncs.com/201708/robotic_arm/Robotic_Arm_Joint_Movement.mp4?_=1" />
</video>
机械臂控制部分思路是把Leap Motion检测到的手掌运动作为输入，映射为机械臂的末端运动。这块涉及到机械臂逆向运动学模型的推导，内容较多，后面有机会另写吧。在前序的研究里也试过ROS（机器人操作系统）的Moveit模块，可以实现在输入机械臂外形的基础上实现末端控制、运动轨迹规划、碰撞检测的功能.
<video controls="controls" width=800 height=480 style="clear: both;display: block;margin-left: auto; margin-right: auto;max-width: 100%;">
  <source src="https://yunshuixin-cn-video.oss-cn-beijing.aliyuncs.com/201708/robotic_arm/Robotic_Arm_Controlled_By_Human_Gesture.mp4" />
</video>

