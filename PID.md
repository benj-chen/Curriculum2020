# PID

[Press here to return to main page.](https://github.com/iron-claw-972/Curriculum2020)

## 1. Purpose

This activity will introduce the PID, a useful tool in precisely controlling motors. Furthermore, this activity will describe tuning PIDs.

## 2. Materials

- FRC VS Code
  - Downloading FRC VS Code is detailed [here](https://github.com/iron-claw-972/Curriculum2020/blob/master/InstallingFrcPrereqs.md#frc-vscode).
- Access to the device, system you are attempting to control
  - Most common example is a motor
- Something that measures the state of the device or system you are trying to control
  - Most common example is an encoder, which reports the true speed of a motor

## 3. Instructions

### When is PID useful/applied?

PID is used whenever we need to set a variable (a motor's speed, the robot's position, the angle of a robotic arm) on the robot to a particular value, but the variable's value doesn't remain stable or is inaccurate (a motor's speed starts to shift and change, the robot doesn't park in the desired place, the robotic arm is at the wrong angle). PID is a control system, which means it controls the variable so that it remains roughly in place. 

### Creating a PID object

Since we have already written a ready-to-use PID object, how to write a PID object from scratch will be covered later. 
The constructor of a PID object should look like so:

```java

public PID(double pFactor_, double iFactor_, double dFactor_) {
	pFactor = pFactor_;
	iFactor = iFactor_;
	dFactor = dFactor_;
}

```

As you can see, there are 3 "factors". Changing these values dictates the behavior of the PID.
Hence, initializing/declaring one looks like this:

```java

import frc.robot.util.*;

public class SubsystemWithPID {

  public PID example;
  
  //Filled in during tuning period
  private final pFactor;
  private final dFactor;
  private final iFactor;
  
  public SubsystemWithPID() {
    example = new PID(pFactor, iFactor, dFactor);
  }
  
}

```

### Using a PID object

This is best explained through an example. In this case, it will be attempting to stabilize a motor's speed with an encoder, but this can be replaced with any variable and device. The real meat of PID is accomplished in what our ready-made defines as the update method, shown here:

```java
public double update(double setpoint, double actual, double timeFrame) {
	//Actual PID math
	double present = setpoint - actual;
	integral += present * timeFrame;
	double deriv = (present - lastError) / timeFrame;
	lastError = present;
	return present * pFactor + integral * iFactor + deriv * dFactor;
}
```

As you can see, it takes three parameters, the setpoint (meaning the value you want the motor speed [variable] to be), the actual (meaning the value the motor speed is measured to be from the encoder), and the timeFrame (the time from the previous robot update cycle to the next). Combining this understanding 

```java

import frc.robot.util.*;
import com.ctre.phoenix.motorcontrol.can.TalonFX;

public class StableMotorSpeed {

  private final TICKS_PER_ROTATION = Context.TICKS_PER_ROTATION; 
  //How many counts an encoder makes for a revolution; For a TalonFX this is 2048

  public TalonFX motor;
  
  //Your motor's ID of choice
  private final motorID;
  
  public PID motorPID;
  
  //Filled in during tuning period
  private final pFactor;
  private final dFactor;
  private final iFactor;
  
  public double nextInput; //Output from PID
  
  public double actualSpeed; //In m/s
  public double desiredSpeed;
  
  public long currentTime; //Time of update in milliseconds
  public long previousTime; //Time on last update in milliseconds
  public double timeFrame; //Time since last update in seconds
  
  public StableMotorSpeed() {
    	motorPID = new PID(pFactor, iFactor, dFactor);
    	motor = new TalonFX(motorID);
	motor.configSelectedFeedbackSensor(FeedbackDevice.IntegratedSensor);
  }
  
  public loop(double desiredSpeed_) {
  	desiredSpeed = desiredSpeed_;
	actualSpeed = motor.getSelectedSensorVelocity()/TICKS_PER_ROTATION * WHEEL_RADIUS;
	currentTime = System.currentTimeMillis();
	timeFrame = (currentTime - previousTime)/1000.0; //1000 for milliseconds
	previousTime = currentTime;
	nextInput = motorPID.update(desiredSpeed, actualSpeed, timeFrame);
	motor.set(ControlMode.PercentOutput, nextInput);
	
	//Here, we are feeding in nextInput in as the actual proportion of the max current (read: max RPM).
	//However, it may even be preferable to have the PID return the actual next RPM, and 
	//then convert that into a current input.
  }
  
}

```



## 4. Further Reading

[JanisMac's Control Challenges](https://janismac.github.io/ControlChallenges/) are very simple JavaScript based challenges that can be used to illustrate PIDs.
