# Road Runner Quickstart

This repository is a fork of the acmerobotics road-runner-quickstart repository with fixes for the SparcFun OTOS odometry board.

[8/12/2025] Issues have been written against the acmerobotics code, but in the meantime, this contains the fixes and works for tuning a robot with the OTOS board. It has only been tested with mecanum drive. The fix needed the base road runner FTC code, so it has been copied into this repository (under the RoadRunner and web directories - only change is to tuning.kt).

Check out the baseline tuning [docs](https://rr.brott.dev/docs/v1-0/tuning/).

Here's my additional notes to the tuning document above that are specific to OTOS and mecanum drive. My values are for a REV mecanum drivetrain kit.

---
## OTOSAngularScalarTuner

This measures the IMU heading accuracy. 

In OTOSLocalizer line 18:  
`        public double angularScalar = 1.0;`

1. Run the test
2. Uncorrected degrees turned: 3625.   Calculated Angular Scalar: 0.9931  
`        public double angularScalar = 0.99;`
3. If you re-run the test with the new value, Calculated Angular Scalar should be 1.0

---
## OTOSLinearScalerTuner

This measures the accuracy of the OTOS when measuring distance.

In OTOSLocalizer line 19:  
`        public double linearScalar = 1.0;`

1. Measured 95 inches
2. Run the test
3. Uncorrected was 55, but depended on speed (78 when slow, 39 when fast)
4. Tuning instructions say Uncorrected / Actual, but really Actual / Uncorrected.  
`        public double linearScalar = 95.0 / 55.0;`
5. If value is less than 0.872 or greater than 1.127, then set it to the limit. Otherwise it will be ignored.  
      You will have to adjust for the difference in your distances - maybe use a function to convert actual
      value to sent value.
6. If you re-run after changing linearScalar: uncorrected now should be close to actual (since it is corrected)

---
## OTOSHeadingOffsetTuner

This accounts for a heading difference between the OTOS sensor and the robot.

In OTOSLocalizer line 22:  
`        public SparkFunOTOS.Pose2D offset = new SparkFunOTOS.Pose2D(0, 0, 0);`
1. Run the test
2. My OTOS had a Heading Offset of 0 (aligned straight ahead with the robot), but let's pretend it was 0.01 radians  
`        public SparkFunOTOS.Pose2D offset = new SparkFunOTOS.Pose2D(0, 0, 0.01);`
3. If you re-run the test after setting the Pose h value, the Heading Offset should be 0

---
## OTOSPositionOffsetTuner

This accounts for a position difference between the OTOS sensor and the center of the robot.

In OTOSLocalizer line 22:  
`        public SparkFunOTOS.Pose2D offset = new SparkFunOTOS.Pose2D(0, 0, 0.01);`
1. Run the test
2. Enter X Offset and Y Offset values into the first 2 values in Pose offset
   (values need to be negated):  
`        public SparkFunOTOS.Pose2D offset = new SparkFunOTOS.Pose2D(0.2, 7.5, 0.01);`
3. If you re-run the test after setting the Pose values, the Offset values should be close to 0

---
## Push tests

Since inchesPerTick is 1 for OTOS, no need to do the push tests

---
## ForwardRampLogger

This test calculates kS and kV parameters. kV determines speed per power. For my robot with
Rev UltraPlanetary 20:1 motors, kV was about 0.25. As the gear ratio gets higher, kV should get
higher. kS is an offset of what voltage is required to start moving the robot. A value between 0 and 0.5
is what I get with my low-weight robot.

1. Run the test. I run it multiple times and take the average values.
2. Open the web page and download the data
3. Copy the values for kS and kV to MecanumDrive.java lines 71-72:  
`        public double kS = 0.1;
        public double kV = 0.25;`

---
## LateralRampLogger

Not needed for OTOS - keep lateralInPerTick set to inPerTick.

---
## AngularRampLogger

This test calculates trackWidth in ticks. But since OTOS has 1 in/tick, it is just the width between the
center of the wheels in inches. You can measure that and put it in MecanumDrive.java line 68 trackWidthTicks.

You can also run the test and compare the computed value to the measured value - this verifies that kV and kS
are correct.

Due to a bug in OTOS.kt getRobotAngularVelocity, the value may need to be converted between degrees and radians.
If the trackWidth is much larger than the expected number of inches, divide the value by 57.3, which is the number
of degrees in a radian.

1. Run the test.
2. Open the web page and download the data
3. Enter the values for kS and kV.
4. If the value for track width is very large, divide it by 57.3. Mine was 837, but robot is about 15 inches wide.  
       837 / 57.3 = 14.6. Measured value is 15, so kV is probably off by a little. 
3. Copy the values for track width to MecanumDrive.java lines 68:  
`        public double trackWidthTicks = 15.0;`

---
## ManualFeedforwardTuner

1. Run test as written
2. Put value of kA in MecunumDrive.java line 73:  
`        public double kA = 0.075;`

My value of kA is 0.075

---
## ManualFeedbackTuner

1. Run test as written
2. Put gain values in MecunumDrive.java lines 85-87:  
`        public double axialGain = 2.0;
        public double lateralGain = 2.0;
        public double headingGain = 2.0; // shared with turn`

I chose 2 for my values.

---
At this point, it is done. Run StraightTest to verify it goes in a straight line 24 inches, and run SplineTest to verify it does a half circle 60 inches in diameter.
