# Controlling Advanced Robot Mechanisms

[Press here to return to main page.](https://github.com/iron-claw-972/Curriculum2020)

## 1. Purpose

This activity will introduce how to control several advanced robot mechanisms, namely:
- Pneumatics
- Reading Motor encoders
  - TalonSRX
  - TalonFX
  - CANSparkMax
- Encoders connected directly to the RoboRio

## 2. Materials

- FRC VS Code
  - Downloading FRC VS Code is detailed [here](https://github.com/iron-claw-972/Curriculum2020/blob/master/InstallingFrcPrereqs.md#frc-vscode).

## 3. Instructions

### Pneumatics

A piston, the main thing we use pneumatics for, is controlled by a solenoid. There are two kinds, single solenoids (```Solenoid```) and double solenoids (```DoubleSolenoid```). A single solenoid only requires one port on the PCM and when turned on, it extends the piston, and turned off retracts. This is simple enough, but it presents the risk that when the robot is shut off at the end of a match, the piston will automatically close. A double solenoid does not suffer from this issue and can be turned off without losing its state; it requires two PCM ports. 

The official WPI documentation for the DoubleSolenoid object is [here](https://first.wpi.edu/FRC/roborio/release/docs/java/edu/wpi/first/wpilibj/DoubleSolenoid.html). 
The solenoid constructor needs to know the two port IDs the double solenoid is using for extension and retraction (from 0-7 on the PCM, the extension one is the port corresponding to the solenoid connected to the back of the piston and the one at the front for retraction). 

```java 
DoubleSolenoid piston = new DoubleSolenoid(int forwardChannel, int reverseChannel); //forwardChannel for extension, reverseChannel for retraction, from 0 - 7
```

Once you've created a DoubleSolenoid object, extension and retraction is done through the set method, which accepts as parameters DoubleSolenoid.Value.kOff, DoubleSolenoid.Value.kForward, DoubleSolenoid.Value.kReverse, corresponding respectively to telling the DoubleSolenoid to stay where it is, extending forward, and retracting backward. 

A Solenoid object is almost the same affair (docs [here](https://first.wpi.edu/FRC/roborio/release/docs/java/edu/wpi/first/wpilibj/Solenoid.html)). The difference is that the constructor only needs the single port the solenoid is connected to:

```java 
Solenoid piston = new Solenoid(int channel); //One on and off channel
```

and the set method just accepts a boolean, true for extension, false for retraction. 

Here's it fully written out:

```java

DoubleSolenoid pistonA = new DoubleSolenoid(0, 1); //Forward channel is leftmost port on PCM and reverse is to its right
Solenoid pistonB = new Solenoid(2); //Right next to the reverse channel for the other solenoid

//Assuming there's a pause between each of these operations

pistonA.set(DoubleSolenoid.Value.kForward); //Piston extends
pistonA.set(DoubleSolenoid.Value.kOff); //Shuts off, stays extended
pistonA.set(DoubleSolenoid.Value.kReverse); //Piston retracts
pistonA.set(DoubleSolenoid.Value.kOff); //Shuts off, stays retracted

pistonB.set(true); //Piston extends
pistonB.set(false); //Shuts off, piston immediately retracts

```

### Motor Encoders

#### Talon FX

The Talon FX motor controller has an integrated encoder, but before you are able to collect data from it through the TalonFX, you have to do:

```java
TalonFX motor = new TalonFX(/*Relevant motor ID*/);
motor.configSelectedFeedbackSensor(TalonFXFeedbackDevice.IntegratedSensor); //You tell the TalonFX to set its associated encoder to the integrated one
```
To get the measured rotation angle of the encoder, you need to do:

```java
int encoderCounts = motor.getSelectedSensorPosition();                  //An encoder measures rotation in "ticks"; a TalonFX encoder has 2048 CPR (Counts per rotation)
double encoderAngle = encoderCounts * 360.0/2048;                       //Converting to angle
double wheelTravelDistance = encoderCounts * 2 * Math.PI * Context.WHEEL_RADIUS / 2048; //Converting to the distance a wheel travels in m (assuming WHEEL_RADIUS in m)
```
Same goes for rotation speed:

```java
int encoderSpeed = motor.getSelectedSensorVelocity();                    //Encoder ticks per 100 ms; 2048 CPR
double encoderRPM = encoderSpeed/2048 * 10 * 60;                         //Converting to RPM
double wheelTravelSpeed = encoderSpeed * 2 * Math.PI * Context.WHEEL_RADIUS / 2048 * 10; //Converting to the linear speed of the wheel in m/s (assuming WHEEL_RADIUS in m)
```

#### Talon SRX

The Talon SRX motor controller does not have an integrated encoder, but the encoder will still plug right into it. Usually you will be using a mag encoder from CTRE, so you will do:

```java
TalonSRX motor = new TalonSRX(/*Relevant motor ID*/);
motor.configSelectedFeedbackSensor(CTRE_MagEncoder_Relative); //You tell the TalonSRX to set its encoder to a mag encoder mimicking a quadrature encoder
```
To get the measured rotation angle of the encoder, you need to do (redundant from TalonFX, just change CPR to 1024):

```java
int encoderCounts = motor.getSelectedSensorPosition();                  //An encoder measures rotation in "ticks"; a CTRE Mag Encoder has 1024 CPR (Counts per rotation)
double encoderAngle = encoderCounts * 360.0/1024;                       //Converting to angle
double wheelTravelDistance = encoderCounts * 2 * Math.PI * Context.WHEEL_RADIUS / 1024; //Converting to the distance a wheel travels in m (assuming WHEEL_RADIUS in m)
```
Same goes for rotation speed:

```java
int encoderSpeed = motor.getSelectedSensorVelocity();                    //Encoder ticks per 100 ms; 2048 CPR
double encoderRPM = encoderSpeed/1024 * 10 * 60;                         //Converting to RPM
double wheelTravelSpeed = encoderSpeed * 2 * Math.PI * Context.WHEEL_RADIUS / 1024 * 10; //Converting to the linear speed of the wheel in m/s (assuming WHEEL_RADIUS in m)
```

#### CAN Spark Max (For NEOs)

The CAN Spark Max motor controller is used for NEOs. Setting up the encoder is a bit different, as there is a separate CANEncoder object for the encoder, whereas with Talon it's written into the controller.

```java
CANSparkMax motor = new CANSparkMax(/*int motor ID*/, CANSparkMaxLowLevel.MotorType.kBrushless); //If you happen to be using a brushed motor (unlikely), change this to kBrushed
CANEncoder encoder = motor.getEncoder();                                                         //This is the object you will reference to get encoder values
```

The CANEncoder, unlike its Talon counterparts, returns its position as a double for the total rotations completed.

```java
double encoderRotations = encoder.getPosition(); //0.5 is half a rotation, 1 is a full rotation, -0.5 is a half rotation back, etc.
double encoderAngle = encoderRotations * 360;    //Degrees
double wheelTravelDistance = encoderRotations * 2 * Math.PI * Context.WHEEL_RADIUS;
```

Rotation speed is returned in RPM:

```java
double encoderSpeed = encoder.getVelocity(); //in RPM
double encoderAngularSpeed = encoderSpeed * 360/60; //rotations/minute to degrees/second
double wheelTravelSpeed = encoderSpeed * 2 * Math.PI * Context.WHEEL_RADIUS / 60;
```

## 4. Further Reading
