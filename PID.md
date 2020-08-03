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
  
  //Filled in during tuning
  private final pFactor;
  private final dFactor;
  private final iFactor;
  
  public SubsystemWithPID() {
    example = new PID(pFactor, iFactor, dFactor);
  }
  
}

```

### Using a PID object

This is best demonstrated using an example. In this case, it will be attempting to stabilize a motor. The real meat of PID is accomplished in what our ready-made defines as the update method, shown here:

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

## 4. Further Reading

[JanisMac's Control Challenges](https://janismac.github.io/ControlChallenges/) are very simple JavaScript based challenges that can be used to illustrate PIDs.
