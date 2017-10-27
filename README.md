# LineFollowingRobot

## Setting Thing Up

There are various alternatives when interfacing with LEGO Mindstorms.  But there are two easy ways for get going as I understood. Either  matlab simulink or LEGO Mindstorms SDK. Each of the have their own drawback and limitations. 

##### Problem when work with matlab simulink

1. Matlab linux version up to 17b, does not support bluetooth. Then communicate robot we have to use either usb or wifi. To use wifi, wifi adapter is required. 

2. In order to do real time monitoring, there is only one communication method (wifi) is supported by matlab simulink with LEGO Mindstorms hardware. Which mean when using usb or bluetooth, real time monitoring is not possible. 

##### Problem with LEGO Mindstorms SDK,

1. It does not support linux yet. 

After playing around those death ended problems I have decided to use LEGO Mindstorms SDK in windows. 

## Design Idea

![image alt text](image_0.png)

In line following robot,  basic goal is to move robot such that it won't move away from the path is be followed. If it moves away , need to bring it back to path in an optimal manner. To achieve this goal, PID controller can be employed to reduce the overall error.

## Requirement of PID Controller

Let’s define the problem statement. Idea is to follow the line. So try to track the edge on the path and reduce overall error in real time. 
Since color sensor is used to detect line, let’s say for black it returns 50 and for white it returns 40. To find out mid point where color sensor should be pointed, below equation can be used. 

  midpoint = black + (black-white)/2

Then error can be defined as 

    error = midpoint - color sensor reading value
    

If color sensor reading value is 40, then error will equal to 15 (55-40). To reduce this error we can do multiplying error term with some constant (Kp). This gives how much robot should move otherside to reduce the error. In this case Kp = 1, and it should be around 1 to reduce error. If it high error getting high and higher and no way to reduce it. Since we have two colors in other words two values (40 and 50), what happens when robot went away form the path. It keeps getting constant error. So how do we address this problem. We need to sum up previous errors and reduce it to get robot back to path. Let’s take robot moving away from path. Then when we are adding all those previous errors it will get bigger and bigger. To reduce it, we have to overshoot robot other direction  much as possible. This also experience robot oscillation. To avoid getting bigger integral error, we can multiply integral error by some constant (Ki) to reduce it. 
Up to now we are able to reduce the current error (Kp) and the previous error (Ki). To reduce error which can happen in the future, we have think a little bit more. Let’s put it on this way, the next error is expected to be the current error plus the change in the error between the two preceding sensor samples. The change in the error between two consecutive points is called the derivative. The derivative is the same as the slope of a line. So this is the solution for reducing an error which will happen in the future. Like previously, to control this error we can multiply some constant (Kd). In PID controller, basic idea is that and need to find proper value for Kp, Kd and Ki. Then, robot can move along the line very smoothly. 

## Color Sensor Calibration

This is noting but need to find value for black and white space. This can be done in matlab or using  LEGO Mindstorms SDK,
This matlab script can be used to read the color sensor values: 
This is the  LEGO Mindstorms configuration to read the color sensor values. In this I have added additional sensor (touch) for the convenient. First it reads black or white and when it press it reads again. In this first it should read white and then black. 
![image alt text](image_1.png)    
![image alt text](image_2.png)

## Intuition Behind Selecting Proper Values for Kp, Ki and Kd in PID Controller 

Pseudo code for PID controller 

                program LINE FOLLOWING
                  white = 0, black = 0
                  CALIBRATE()
                  midpoint = ( white - black ) / 2 + black
                  kp = 1, ki = 1, kd = 1
                  lasterror=0
                  repeat
                    value = Read Light Sensor
                    error = midpoint - value
                    integral = error + integral
                    derivative = error - lasterror
                    correction = kp * error + ki * integral + kd * derivative
                    Turn B+C Motors by correction
                    lasterror = error
                  done repeat
                done program

To turn Kp, Ki and Kd values, I have set Ki and Kd values to zero and put Kp to 1. Reason for putting it into 1 I explained above under section:Requirement of PID Controller. Then try to see the system and change it up when system keep oscillating. Then Ki is increased until the response time or transient process time get reduced upto considerable level. To decide where to stop I have used if system having high overshoot which mean it reached where it can not be increased anymore. So then I set Kd to 1 and check the it converges to stable state very fast. If it not try to reduce by factor of 2 and finally was able to find proper values for  Kp, Ki and Kd which I think the optimal values for this robot in this environment. Because by changing those coefficient I was able to increase the rise time, eliminate the steady-state error, control and improve the overshoot of robot. 

## Final Solution

LEGO Mindstorms file for PID controller can be found here.  

Result: 
