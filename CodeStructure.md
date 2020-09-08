# Code Structure

[Press here to return to main page.](https://github.com/iron-claw-972/Curriculum2020)

## 1. Purpose

This activity will introduce this year's code structure, describing how all of its components work together to make the robot complete complex tasks.

## 2. Materials & Prereqs

- GitHub Desktop
- FRC VS Code
- Understands basic drive code
- Knowledge of Object Oriented Programming in Java (classes, methods, inheritance, etc.)

## 3. Instructions

1. First clone the code structure repository from here: https://github.com/iron-claw-972/CodeStructure2020. This contains the code that we will start building off of. As we will show, using this code structure will make it much easier for many people to work on the same code.
2. This structure allows for the abstraction of the robot. For example, instead of directly programming an intake motor to spin when a button is pressed in teleop periodic, we would instead write an intake _controller_ which could have methods spin and stop the intake motor. Then, we would write an intakeball _action_ which would be triggered by a button press and call the spin method.

![image](https://github.com/iron-claw-972/Curriculum2020/blob/master/images/battlebot.png)

3. In the above example of a battlebot, you can see an simplified flowchart example of how one might program a robot to spin a motor on a button press. The SpinnerController.java contains methods to start and stop the motor. AttackAction.java starts the motor in its start method, which is automatically called when the AttackAction is activated. OperatorJoystick.java activates AttackAction when a certain button on the logitech controller is pressed. Finally, AttackAction.java has a method which is called when the button is released, and this method deactivates the action and calls the stop method of SpinnerController.java. Here are the 5 major files you will have to edit or make to accomplish the task described above: 

- AttackAction.java: contains the action to spin and stop the motor
- SpinnerController.java: provides the methods to control the motor
- OperatorJoystick.java: binds the action to a button
- RobotController.java: contains the motor controller objects and sensor objects needed by SpinnerController.java to spin the motor
- Context.java: stores the IDs for the motor controller objects created in RobotController.java

4. Next we will go though how to create a program that spins a motor on a button press, and stops spinning the motor when the button is released. First create a new Java file in the controllers folder (you can right click the folder and click New File). Name this file IntakeController.java, and add the code below. Here, we create an intake controller which has methods to start, stop, and reverse the intake roller. The controller will be instantiated and ran with motor objects in RobotController.java.
```
package frc.robot.controllers;

import com.ctre.phoenix.motorcontrol.*;
import com.ctre.phoenix.motorcontrol.can.*;

import edu.wpi.first.wpilibj.*;

public class Intake {
    private TalonSRX intakeTalon;

    public final double intakingSpeed = 1.0; //The speed to rotate at while intaking
    public final double stopSpeed = 0; //The speed to rotate at while not intaking


    public Intake(TalonSRX IntakeTalon) {
        intakeTalon = IntakeTalon;
    }

    public void initialize(){
        intakingSpeed = 0;
    }

    public void beginIntaking() {
        setSpeed = intakingSpeed;
    }

    public void stopIntaking() {
        setSpeed = stopSpeed;
    }

    public void loop() {
        intakeTalon.set(ControlMode.PercentOutput, setSpeed);
    }
}
```

5. Now we will make this controller run in RobotController.java! First lets make our motorcontroller. 
- Add an instance variable in RobotController.java for the motor: ```public TalonSRX intakeMotor;```. 
- In the constructor of RobotController.java, add `intakeMotor = new TalonSRX(Context.intakeMotorID)` where all the other motors are being created (don't worry, we'll definte intakeMotorID in context later). 
- In the constructor of RobotController.java, add ```intakeController = new IntakeController(intakeMotor);``` next to the rest of the controller. 
- Inside of the initAll method, add ```intakeController.initialize()```
- Inside of the loopAll method, add ```intakeController.loop()```
- Now we need to assign a motor ID for the intake motor. Navigate to Context.java (inside the util folder). Add an instance variable for the motor ID: ```public static final int intakeMotorID = -1;```

6. We have the controller running, but we haven't linked its methods of beginIntaking and stopIntaking to a button. To do this, we'll need to create an action. Navigate to the actions folder and create and action called SpinIntake.java. 
## 4. Further Reading
