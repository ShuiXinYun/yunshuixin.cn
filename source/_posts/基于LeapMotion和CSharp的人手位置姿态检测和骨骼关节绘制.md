---
title: 基于LeapMotion和C#的人手位置姿态检测和骨骼关节绘制
date: 2016-05-18
tags:
categories: DIY
---
参加学校天宫杯科创培育项目时写的小程序，主要功能是获得手掌各个关节的空间坐标位置，在此基础上解算出各关节之间的夹角，如果能够配合3D打印的机械手，可以实现如下效果(这块没实践过，价格不菲)：
<video controls="controls" style="clear: both; display: block; margin-left: auto; margin-right: auto; max-width: 80%;">
	<source src="http://yunshuixin-cn-video.oss-cn-beijing.aliyuncs.com/201708/robotic_arm/Robotic-Hand-controlled-by-Leap-Motion.mp4?_=1">
</video>

Leap Motion的C# SDK见：[LeapMotion C# SDK Documentation](https://developer.leapmotion.com/documentation/csharp/index.html)

Leap Motion这款产品个人还是很喜欢的，但受其自身工作原理限制可靠性不是很好，很难拿去做产品级应用。目前其对ARM平台还没有相关的支持，我觉得在树莓派运行起来应该问题不大，估计官方也不会再去更新这方面的东西。

程序效果：
<video controls="controls" style="clear: both; display: block; margin-left: auto; margin-right: auto; max-width: 80%;">
	<source src="http://yunshuixin-cn-video.oss-cn-beijing.aliyuncs.com/201708/robotic_arm/LeapMotion_Hand_Plot_GUI.MP4?_=2">
</video>
主体代码：
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;
using System.Threading;
using System.Diagnostics;
using System.Drawing;
using System.IO;
using System.IO.Ports;
using Leap;

namespace Leap_Arm
{
    public partial class MainWindow : Window, ILeapEventDelegate
    {
        private Controller controller = new Controller();
        private LeapEventListener listener;
        private Boolean isClosing = false;
        Leap.Vector ThumbTip, ThumbDip,ThumbPip, ThumbMcp;
        Leap.Vector IndexTip,IndexDip,IndexPip,IndexMcp;
        Leap.Vector MiddleTip, MiddleDip, MiddlePip, MiddleMcp;
        Leap.Vector RingTip, RingDip, RingPip, RingMcp;
        Leap.Vector PinkyTip, PinkyDip, PinkyPip, PinkyMcp;
        double[] ThumbAngle = new double[2];
        double[] IndexAngle = new double[3];
        double[] MiddleAngle = new double[3];
        double[] RingAngle = new double[3];
        double[] PinkyAngle = new double[3];
        public float factor = 3;
        Boolean ispolyline = false;

        Leap.Matrix basis;
        public MainWindow()
        {
            InitializeComponent();
            this.controller = new Controller();
            this.listener = new LeapEventListener(this);
            controller.AddListener(listener);
            controller.SetPolicy(Controller.PolicyFlag.POLICY_IMAGES);
        }

        delegate void LeapEventDelegate(string EventName);
        public void LeapEventNotification(string EventName)
        {
            if (this.CheckAccess())
            {
                switch (EventName)
                {
                    case "onInit":
                        Debug.WriteLine("Init");
                        break;
                    case "onConnect":
                        this.connectHandler();
                        break;
                    case "onFrame":
                        if (!this.isClosing)
                            this.newFrameHandler(this.controller.Frame());
                        break;
                }
            }
            else
            {
                Dispatcher.Invoke(new LeapEventDelegate(LeapEventNotification), new object[] { EventName });
            }
        }

        void connectHandler()
        {
            this.controller.SetPolicy(Controller.PolicyFlag.POLICY_IMAGES);
            this.controller.Config.SetFloat("Gesture.Swipe.MinLength", 100.0f);
        }
        
        double anglecaculate(Leap.Vector a, Leap.Vector b)
        {
            float ab = a.x * b.x + a.y * b.y + a.z * b.z;
            double adist = Math.Sqrt(a.x * a.x + a.y * a.y + a.z * a.z);
            double bdist = Math.Sqrt(b.x * b.x + b.y * b.y + b.z * b.z);
            double cos = ab / (adist * bdist);
            double angle = Math.Acos(cos) * 57.2958;
            return angle;
        }

        void Angle_Bone_Caculate(Finger finger,Label label1,Label label2,Label label3)
        {
            double[] angle = new double[3];
            Leap.Vector bone1,bone2,bone3,bone4;
            bone1 = finger.Bone(Bone.BoneType.TYPE_DISTAL).Basis.zBasis;
            bone2 = finger.Bone(Bone.BoneType.TYPE_INTERMEDIATE).Basis.zBasis;
            bone3 = finger.Bone(Bone.BoneType.TYPE_PROXIMAL).Basis.zBasis;
            bone4 = finger.Bone(Bone.BoneType.TYPE_METACARPAL).Basis.zBasis;
            angle[0] = bone1.AngleTo(bone2) * 57.2958;
            angle[1] = bone2.AngleTo(bone3) * 57.2958;
            angle[2] = bone3.AngleTo(bone4) * 57.2958;
            label1.Content = angle[0].ToString("F2");
            label2.Content = angle[1].ToString("F2");
            label3.Content = angle[2].ToString("F2");
        }

        void Angle_JointPosition_Caculate(Finger finger,Leap.Vector Wrist_Vector, Label label1, Label label2, Label label3)
        {
            double[] angle_JointPosition = new double[3];
            Leap.Vector Tip, Dip, Pip, Mcp;
            Tip = finger.JointPosition(Finger.FingerJoint.JOINT_TIP);
            Dip = finger.JointPosition(Finger.FingerJoint.JOINT_DIP);
            Pip = finger.JointPosition(Finger.FingerJoint.JOINT_PIP);
            Mcp = finger.JointPosition(Finger.FingerJoint.JOINT_MCP);
            angle_JointPosition[0] = (Dip - Tip).AngleTo(Pip - Dip) * 57.2958;
            angle_JointPosition[1] = (Pip - Dip).AngleTo(Mcp - Pip) * 57.2958;
            angle_JointPosition[2] = (Mcp - Pip).AngleTo(Wrist_Vector - Mcp) * 57.2958;
            label1.Content = angle_JointPosition[0].ToString("F2");
            label2.Content = angle_JointPosition[1].ToString("F2");
            label3.Content = angle_JointPosition[2].ToString("F2");
        }

        void Angle_Arm_Caculation(Leap.Vector Hand_Center_Vector, Leap.Vector Wrist_Vector, Leap.Vector Elbow_Vector, Label label1, Label label2, Label label3)
        {
            double[] angle_Arm = new double[3];
            angle_Arm[0] = (Hand_Center_Vector - Wrist_Vector).AngleTo(Wrist_Vector - Elbow_Vector) * 57.2958;
            angle_Arm[1] = (Wrist_Vector - Elbow_Vector).Yaw * 57.2958;
            angle_Arm[2] = (Wrist_Vector - Elbow_Vector).Pitch * 57.2958;
            label1.Content = angle_Arm[0].ToString("F2");
            label2.Content = angle_Arm[1].ToString("F2");
            label3.Content = angle_Arm[2].ToString("F2");
        }

        void setpointposition(Ellipse e1, Ellipse e2, Ellipse e3, Ellipse e4, Leap.Vector v1, Leap.Vector v2, Leap.Vector v3, Leap.Vector v4)
        {
            Canvas.SetLeft(e1, v1.x * factor);
            Canvas.SetTop(e1, v1.z * factor);
            Hand_draw.Children.Add(e1);
            Canvas.SetLeft(e2, v2.x * factor);
            Canvas.SetTop(e2, v2.z * factor);
            Hand_draw.Children.Add(e2);
            Canvas.SetLeft(e3, v3.x * factor);
            Canvas.SetTop(e3, v3.z * factor);
            Hand_draw.Children.Add(e3);
            Canvas.SetLeft(e4, v4.x * factor);
            Canvas.SetTop(e4, v4.z * factor);
            Hand_draw.Children.Add(e4);
        }

        void polyline_points_add(Polyline polyline, Leap.Vector v1, Leap.Vector v2, Leap.Vector v3, Leap.Vector v4)
        {
            polyline.Points.Add(new System.Windows.Point(v1.x * factor, v1.z * factor));
            polyline.Points.Add(new System.Windows.Point(v2.x * factor, v2.z * factor));
            polyline.Points.Add(new System.Windows.Point(v3.x * factor, v3.z * factor));
            polyline.Points.Add(new System.Windows.Point(v4.x * factor, v4.z * factor));
        }

        void polyline_points_add(Polyline polyline, Leap.Vector v1, Leap.Vector v2, Leap.Vector v3, Leap.Vector v4, Leap.Vector v5)
        {
            polyline.Points.Add(new System.Windows.Point(v1.x * factor, v1.z * factor));
            polyline.Points.Add(new System.Windows.Point(v2.x * factor, v2.z * factor));
            polyline.Points.Add(new System.Windows.Point(v3.x * factor, v3.z * factor));
            polyline.Points.Add(new System.Windows.Point(v4.x * factor, v4.z * factor));
            polyline.Points.Add(new System.Windows.Point(v5.x * factor, v5.z * factor));
        }

        void polyline_form_set(Polyline polyline, System.Windows.Media.Color color)
        {
            polyline.Stroke = new SolidColorBrush(color);
            polyline.StrokeThickness = 5;
            Hand_draw.Children.Add(polyline);
        }

        void newFrameHandler(Leap.Frame frame)
        {
            //this.displayFPS.Content = frame.CurrentFramesPerSecond.ToString();
            /***********************************
            Leap.Image image = frame.Images[0];
            BitmapSource image_bitmap = imagedisplay(image, frame);
            image1.Source = image_bitmap;
            ***********************************/
            FingerList fingersInFrame = frame.Fingers;
            Leap.Vector Hand_Center = frame.Hands.Frontmost.PalmPosition;
            Leap.Vector Wrist= frame.Hands.Frontmost.Arm.WristPosition;
            Leap.Vector Elbow = frame.Hands.Frontmost.Arm.ElbowPosition;
            double x_dis = frame.Hands.Frontmost.PalmPosition.x;
            Hand_X.Content = x_dis.ToString("F2");
            double y_dis = frame.Hands.Frontmost.PalmPosition.y;
            Hand_Y.Content = y_dis.ToString("F2");
            double z_dis = frame.Hands.Frontmost.PalmPosition.z;
            Hand_Z.Content = z_dis.ToString("F2");
            double pitch = frame.Hands.Frontmost.Direction.Pitch*57.3;
            Hand_Pitch.Content = pitch.ToString("F2");
            double yaw = frame.Hands.Frontmost.Direction.Yaw*57.3;
            Hand_Yaw.Content = yaw.ToString("F2");
            double roll = frame.Hands.Frontmost.Direction.Roll*57.3;
            Hand_Roll.Content = roll.ToString("F2");
            //Angle_Arm_Caculation(Hand_Center, Wrist, Elbow, Arm_Wrist_Twist, Arm_Yaw, Arm_Pitch);
            Polyline Thumbpolyline = new Polyline();
            Polyline Indexpolyline = new Polyline();
            Polyline Middlepolyline = new Polyline();
            Polyline Ringpolyline = new Polyline();
            Polyline Pinkypolyline = new Polyline();
            Polyline Mcppolyline = new Polyline();
            Polyline Arm_Yaw_Polyline = new Polyline();
            Polyline Arm_Pitch_Polyline = new Polyline();
            double Arm_Yaw = (Wrist - Elbow).Yaw;
            double Arm_Pitch = (Wrist - Elbow).Pitch;
            Arm_Yaw_Polyline.Points.Add(new System.Windows.Point(100,670));
            Arm_Yaw_Polyline.Points.Add(new System.Windows.Point(100+100*Math.Sin(Arm_Yaw), 670+100*Math.Cos(Arm_Yaw)));
            polyline_form_set(Arm_Yaw_Polyline, System.Windows.Media.Colors.Blue);
            Hand_draw.Children.Clear();
            Thumbpolyline.Points.Clear();
            foreach (Finger finger in fingersInFrame)
            {
                switch (finger.Type)
                {
                        //大拇指
                    case Finger.FingerType.TYPE_THUMB:
                        //使用关节向量计算角度
                        Bone bone_Thumb1 = finger.Bone(Bone.BoneType.TYPE_DISTAL);
                        Bone bone_Thumb2 = finger.Bone(Bone.BoneType.TYPE_INTERMEDIATE);
                        Bone bone_Thumb3 = finger.Bone(Bone.BoneType.TYPE_PROXIMAL);
                        Bone bone_Thumb4 = finger.Bone(Bone.BoneType.TYPE_METACARPAL);
                        Leap.Vector bone_Thumb_Vector1 = bone_Thumb1.Basis.zBasis;
                        Leap.Vector bone_Thumb_Vector2 = bone_Thumb2.Basis.zBasis;
                        Leap.Vector bone_Thumb_Vector3 = bone_Thumb3.Basis.zBasis;
                        Leap.Vector bone_Thumb_Vector4 = bone_Thumb3.Basis.zBasis;
                        double Angle_bone_Thumb1 = anglecaculate(bone_Thumb_Vector1, bone_Thumb_Vector2);
                        double Angle_bone_Thumb2 = anglecaculate(bone_Thumb_Vector2, bone_Thumb_Vector3);
                        double Angle_bone_Thumb3 = anglecaculate(bone_Thumb_Vector3, bone_Thumb_Vector4);
                        Thumb_bone_angle1.Content = Angle_bone_Thumb1.ToString("F2");
                        Thumb_bone_angle2.Content = Angle_bone_Thumb2.ToString("F2");
                        

                        
                        ThumbTip = finger.JointPosition(Finger.FingerJoint.JOINT_TIP);
                        ThumbDip = finger.JointPosition(Finger.FingerJoint.JOINT_DIP);
                        ThumbPip = finger.JointPosition(Finger.FingerJoint.JOINT_PIP);
                        ThumbMcp = finger.JointPosition(Finger.FingerJoint.JOINT_MCP);
                        //使用关节点坐标计算角度
                        /*
                        ThumbAngle[0] = anglecaculate(ThumbDip - ThumbTip, ThumbPip - ThumbDip);
                        Thumb1.Content = ThumbAngle[0].ToString("F2");
                        ThumbAngle[1] = anglecaculate(ThumbPip - ThumbDip, ThumbMcp - ThumbPip);
                        Thumb2.Content = ThumbAngle[1].ToString("F2");
                         */
                        //点与关节对应
                        setpointposition(Thumbpoint1, Thumbpoint2, Thumbpoint3, Thumbpoint4, ThumbTip, ThumbDip,ThumbPip, ThumbMcp);
                        polyline_points_add(Thumbpolyline, ThumbTip, ThumbDip, ThumbPip, ThumbMcp, Wrist);
                        break;

                        //食指
                    case Finger.FingerType.TYPE_INDEX:
                        //使用关节向量计算角度
                        Angle_Bone_Caculate(finger, Index_bone_angle1, Index_bone_angle2, Index_bone_angle3);
                        //点与关节对应
                        IndexTip = finger.JointPosition(Finger.FingerJoint.JOINT_TIP);IndexDip=finger.JointPosition(Finger.FingerJoint.JOINT_DIP);
                        IndexPip = finger.JointPosition(Finger.FingerJoint.JOINT_PIP);IndexMcp=finger.JointPosition(Finger.FingerJoint.JOINT_MCP);
                        setpointposition(Indexpoint1, Indexpoint2, Indexpoint3, Indexpoint4, IndexTip, IndexDip, IndexPip, IndexMcp);
                        polyline_points_add(Indexpolyline, IndexTip, IndexDip, IndexPip, IndexMcp);
                        //使用关节点坐标计算角度
                        //Angle_JointPosition_Caculate(finger, Wrist, Index1, Index2, Index3);
                        break;

                        //中指
                    case Finger.FingerType.TYPE_MIDDLE:
                        Angle_JointPosition_Caculate(finger, Wrist, Middle1, Middle2, Middle3);
                        //点与关节对应
                        MiddleTip = finger.JointPosition(Finger.FingerJoint.JOINT_TIP);MiddleDip=finger.JointPosition(Finger.FingerJoint.JOINT_DIP);
                        MiddlePip = finger.JointPosition(Finger.FingerJoint.JOINT_PIP);MiddleMcp=finger.JointPosition(Finger.FingerJoint.JOINT_MCP);
                        setpointposition(Middlepoint1, Middlepoint2, Middlepoint3, Middlepoint4, MiddleTip, MiddleDip, MiddlePip, MiddleMcp);
                        polyline_points_add(Middlepolyline, MiddleTip, MiddleDip, MiddlePip, MiddleMcp);
                        break;

                        //无名指
                    case Finger.FingerType.TYPE_RING:
                        Angle_JointPosition_Caculate(finger, Wrist, Ring1, Ring2, Ring3);
                        //点与关节对应
                        RingTip = finger.JointPosition(Finger.FingerJoint.JOINT_TIP);RingDip=finger.JointPosition(Finger.FingerJoint.JOINT_DIP);
                        RingPip = finger.JointPosition(Finger.FingerJoint.JOINT_PIP);RingMcp=finger.JointPosition(Finger.FingerJoint.JOINT_MCP);
                        setpointposition(Ringpoint1, Ringpoint2, Ringpoint3, Ringpoint4, RingTip, RingDip, RingPip, RingMcp);
                        polyline_points_add(Ringpolyline, RingTip, RingDip, RingPip, RingMcp);
                        break;

                        //小指
                    case Finger.FingerType.TYPE_PINKY:
                        Angle_JointPosition_Caculate(finger, Wrist, Pinky1, Pinky2, Pinky3);
                        //点与关节对应
                        PinkyTip = finger.JointPosition(Finger.FingerJoint.JOINT_TIP);PinkyDip=finger.JointPosition(Finger.FingerJoint.JOINT_DIP);
                        PinkyPip = finger.JointPosition(Finger.FingerJoint.JOINT_PIP);PinkyMcp=finger.JointPosition(Finger.FingerJoint.JOINT_MCP);
                        setpointposition(Pinkypoint1, Pinkypoint2, Pinkypoint3, Pinkypoint4, PinkyTip, PinkyDip, PinkyPip, PinkyMcp);
                        polyline_points_add(Pinkypolyline, PinkyTip, PinkyDip, PinkyPip, PinkyMcp, Wrist);
                        polyline_points_add(Mcppolyline,ThumbPip, IndexMcp, MiddleMcp, RingMcp, PinkyMcp);
                        break;       
                }
            }

            if (ispolyline)
            {
                polyline_form_set(Thumbpolyline, System.Windows.Media.Colors.Red);
                polyline_form_set(Indexpolyline, System.Windows.Media.Colors.Green);
                polyline_form_set(Middlepolyline, System.Windows.Media.Colors.Blue);
                polyline_form_set(Ringpolyline, System.Windows.Media.Colors.Cyan);
                polyline_form_set(Pinkypolyline, System.Windows.Media.Colors.IndianRed);
                polyline_form_set(Mcppolyline, System.Windows.Media.Colors.Black);
            }
            
        }

        void MainWindow_Closing(object sender, EventArgs e)
        {
            this.isClosing = true;
            this.controller.RemoveListener(this.listener);
            this.controller.Dispose();
        }

    }

    public interface ILeapEventDelegate
    {
        void LeapEventNotification(string EventName);
    }

    public class LeapEventListener : Listener
    {
        ILeapEventDelegate eventDelegate;

        public LeapEventListener(ILeapEventDelegate delegateObject)
        {
            this.eventDelegate = delegateObject;
        }
        public override void OnInit(Controller controller)
        {
            this.eventDelegate.LeapEventNotification("onInit");
        }
        public override void OnConnect(Controller controller)
        {
            controller.SetPolicy(Controller.PolicyFlag.POLICY_IMAGES);
            controller.EnableGesture(Gesture.GestureType.TYPE_SWIPE);
            this.eventDelegate.LeapEventNotification("onConnect");
        }

        public override void OnFrame(Controller controller)
        {
            this.eventDelegate.LeapEventNotification("onFrame");
        }
        public override void OnExit(Controller controller)
        {
            this.eventDelegate.LeapEventNotification("onExit");
        }
        public override void OnDisconnect(Controller controller)
        {
            this.eventDelegate.LeapEventNotification("onDisconnect");
        }

    }

}
```
