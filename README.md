# ⚡ ESP32 PID Line Following Robot

> An autonomous, high-performance line-following robot designed around PID-based steering control, precise line detection, and adaptive motor-speed management.

## 📌 Overview

This project aims to develop an intelligent line-following robot using an ESP32 microcontroller, a five-channel infrared sensor array, and a dual-channel motor driver.

Unlike conventional line-following robots that rely on simple sensor-based left/right decisions, this robot uses a **PID (Proportional–Integral–Derivative) control algorithm** to continuously calculate steering corrections based on the detected position of the line.

The primary objective is to achieve an optimal balance between:

- **Stability:** Minimize unnecessary oscillations and zigzagging.
- **Accuracy:** Maintain precise tracking of the line.
- **Speed:** Maximize forward speed without sacrificing tracking reliability.
- **Reliability:** Handle curves, direction changes, and temporary line-detection failures.

The project focuses on combining mechanical optimization, sensor processing, motor control, and feedback-based algorithms into a compact autonomous robotic platform.

---

## 🎯 Project Objectives

1. Develop a reliable PID-based line-following system.
2. Implement accurate line-position estimation using five IR sensors.
3. Independently control two DC gear motors using PWM.
4. Minimize oscillations and improve cornering performance.
5. Explore adaptive speed control based on tracking error.
6. Implement sensor calibration and line-loss recovery.
7. Provide configurable PID parameters for experimentation and tuning.

---

## 🏗️ Hardware Architecture

The robot uses a three-wheel differential-drive configuration.

### 1. Microcontroller

**ESP32 DevKit V1 (ESP-WROOM-32)**

The ESP32 acts as the central controller responsible for:

- Reading infrared sensor measurements.
- Calculating line-position error.
- Executing the PID algorithm.
- Generating PWM motor-control signals.
- Managing motor direction.
- Handling sensor calibration and telemetry.

The ESP32 provides sufficient processing capability for real-time PID calculations and additional control features.

### 2. Line Detection System

**Five-Channel Infrared Line Sensor Array**

Five infrared sensors are arranged horizontally to detect the position of a line relative to the robot.

Proposed sensor arrangement:

| Sensor | Position Weight |
|---|---:|
| S1 | -2000 |
| S2 | -1000 |
| S3 | 0 |
| S4 | +1000 |
| S5 | +2000 |

The sensor readings are processed to estimate the line's position.

For analog sensors, a weighted-average calculation can be used:

\[
Position = \frac{\sum_{i=1}^{5}w_iq_i}{\sum_{i=1}^{5}q_i}
\]

Where:

- \(w_i\) represents the position weight.
- \(q_i\) represents the calibrated line-detection strength.

The tracking error is calculated as:

\[
Error = Position - TargetPosition
\]

The target position is normally zero, representing the center of the sensor array.

*Note: Sensor polarity and calibration depend on the actual sensor module and whether it provides analog or digital outputs.*

### 3. Motor Driver

**TB6612FNG Dual H-Bridge Motor Driver**

The TB6612FNG controls the two DC gear motors independently.

Responsibilities include:

- Independent left and right motor control.
- PWM-based speed adjustment.
- Forward and reverse motor operation.
- Direction-based steering correction.
- Standby control.

The driver is selected instead of the L298N because it uses MOSFET-based output stages and generally offers better efficiency for small robotic applications.

Motor voltage and stall-current compatibility must be verified before operation.

### 4. Motors

**2 × N20 Metal Gear DC Motors (12V)**

The current motor selection consists of two N20 metal gear motors.

Each motor independently drives one wheel.

Motor specifications to verify:

- Rated voltage: 12V
- Gearbox RPM: To be confirmed
- Stall current: To be measured or obtained from the manufacturer
- Rated torque: To be confirmed

The motors will initially operate from the 2S battery through the TB6612FNG.

**Important:** The 2S battery supplies approximately 7.4V nominally and 8.4V when fully charged. Therefore, these 12V motors will operate below their rated voltage. Actual speed and torque performance must be tested. The TB6612FNG must also be checked against the motors' stall-current requirements.

### 5. Mechanical Structure

**Three-Wheel Differential Drive Chassis**

The proposed configuration contains:

- Two powered N20 drive wheels.
- One passive ball caster.
- A lightweight, rigid chassis.
- An adjustable front-mounted sensor assembly.

The preferred chassis shape is a compact rounded arrowhead or trapezoidal design.

Design considerations:

- Keep the centre of gravity low.
- Mount both drive wheels symmetrically.
- Position the sensor array ahead of the drive axle.
- Ensure the ball caster rolls smoothly in every direction.
- Keep the battery near the centre of the chassis.
- Minimize unnecessary mechanical weight.

The ball caster is preferred over a conventional swivel caster because it avoids swivel-alignment delays during direction changes.

---

## ⚡ Power Architecture

### Battery

**2S Lithium Battery Pack**

| Parameter | Specification |
|---|---|
| Configuration | 2S |
| Nominal voltage | 7.4V |
| Fully charged voltage | 8.4V |
| Application | Motor power and regulated electronics supply |

### Power Distribution

The proposed power architecture separates the motor power path from the regulated logic supply.

```text
              2S Battery Pack
             7.4V / 8.4V
                    |
              Main Power Switch
                    |
              Power Distribution
                /          \
               /            \
       TB6612FNG          Buck Converter
       Motor Supply       5V Regulated
           |                  |
       /       \           ESP32
    Motor A  Motor B       DevKit
```

All system grounds must share a common reference.

### Power Considerations

- The TB6612FNG motor supply (VM) will receive battery voltage.
- Its logic supply (VCC) will receive an appropriate regulated logic voltage.
- The ESP32 will receive regulated 5V through its appropriate 5V/VIN input.
- Motor and logic power distribution should be designed to minimize electrical noise.
- A suitable battery protection arrangement, fuse, and power switch should be included.
- Motor stall current must remain within the motor driver's safe operating limits.

---

## 🧠 PID Control System

The robot uses a feedback-based PID controller to determine the required steering correction.

The PID algorithm consists of three components.

### Proportional (P)

Responds to the current line-position error.

\[
P = K_p \times Error
\]

A higher proportional gain produces a stronger response to tracking errors.

Excessive proportional gain can cause oscillation and zigzagging.

### Integral (I)

Corrects accumulated tracking error.

\[
I = K_i \times \sum Error \times \Delta t
\]

Integral control can compensate for persistent mechanical or motor-related bias.

Excessive integral gain can cause overshooting and integral windup.

### Derivative (D)

Responds to the rate of change of the tracking error.

\[
D = K_d \times \frac{Error-PreviousError}{\Delta t}
\]

Derivative control helps reduce oscillations and anticipate rapid changes in the tracking error.

### Complete PID Equation

\[
Output = K_pE + K_i\int E\,dt + K_d\frac{dE}{dt}
\]

Where:

- \(K_p\): Proportional gain.
- \(K_i\): Integral gain.
- \(K_d\): Derivative gain.
- \(E\): Tracking error.

The initial implementation will use PD control with \(K_i=0\). Integral correction can be introduced if testing identifies persistent tracking bias.

---

## 🚗 Motor Control Strategy

The PID output is used to modify the speed of the left and right motors.

For a differential-drive configuration:

\[
LeftSpeed = BaseSpeed + PIDOutput
\]

\[
RightSpeed = BaseSpeed - PIDOutput
\]

The correction direction will be adjusted according to the sensor-weight convention and actual motor wiring.

Both motor commands will be constrained within their valid PWM limits.

### Adaptive Speed Control (Planned)

The robot will explore dynamic base-speed adjustment.

| Tracking Condition | Expected Behaviour |
|---|---|
| Small tracking error | Maintain or increase speed |
| Moderate tracking error | Reduce speed gradually |
| Large tracking error | Reduce speed significantly |
| Rapidly changing error | Anticipate the turn and slow down |
| Line lost | Initiate recovery procedure |

This approach aims to improve lap time without compromising stability.

---

## 🔌 Proposed ESP32 Pin Configuration

The following GPIO allocation is intended for the standard ESP32 DevKit V1.

### TB6612FNG Connections

| ESP32 GPIO | TB6612FNG Pin | Function |
|---|---|---|
| GPIO 25 | AIN1 | Motor A direction |
| GPIO 26 | AIN2 | Motor A direction |
| GPIO 18 | PWMA | Motor A speed |
| GPIO 27 | BIN1 | Motor B direction |
| GPIO 14 | BIN2 | Motor B direction |
| GPIO 19 | PWMB | Motor B speed |
| GPIO 23 | STBY | Driver standby control |

### Five-Channel Sensor Connections

| Sensor | ESP32 GPIO |
|---|---|
| S1 | GPIO 36 |
| S2 | GPIO 39 |
| S3 | GPIO 34 |
| S4 | GPIO 35 |
| S5 | GPIO 32 |

These GPIOs are ADC1-capable inputs on the classic ESP32.

**Electrical note:** ESP32 GPIOs are not 5V tolerant. Sensor outputs must remain within safe input-voltage limits. The exact pin allocation must also be checked against the selected sensor module and board revision.

---

## ⚙️ Performance Optimization

The following factors will be considered during PID tuning and mechanical optimization.

| Factor | Impact |
|---|---|
| Base motor speed | Overall tracking performance |
| Sensor placement | Detection and steering response |
| Sensor calibration | Position-estimation accuracy |
| Sensor height | Detection reliability |
| Motor RPM | Forward speed |
| Motor torque | Acceleration and steering response |
| Wheel diameter | Speed and turning geometry |
| Wheel traction | Cornering stability |
| Chassis weight | Mechanical inertia |
| Centre of gravity | Stability |
| Battery voltage | Motor performance |
| Control-loop frequency | Response consistency |
| Track curvature | Steering requirements |
| PID gains | Stability, accuracy and responsiveness |

### PID Tuning Procedure

1. Calibrate all five sensors.
2. Verify motor direction and PWM operation.
3. Start with \(K_i=0\) and \(K_d=0\).
4. Increase \(K_p\) gradually.
5. Introduce \(K_d\) to reduce oscillations.
6. Test on straight lines, curves and S-bends.
7. Introduce \(K_i\) only if necessary.
8. Gradually increase the base motor speed.
9. Repeat tests and compare performance.

---

## 🗺️ Development Roadmap

### Phase 1 — Mechanical Assembly
- [x] Select three-wheel differential-drive configuration.
- [x] Select two N20 drive motors.
- [x] Select ball caster.
- [ ] Design and fabricate chassis.
- [ ] Mount motors and wheels.
- [ ] Mount sensor array.

### Phase 2 — Electronics
- [x] Select ESP32 DevKit V1.
- [x] Select TB6612FNG motor driver.
- [x] Select 2S battery architecture.
- [ ] Complete power distribution.
- [ ] Verify motor-driver compatibility.
- [ ] Complete wiring and electrical testing.

### Phase 3 — Firmware Development
- [ ] Configure ESP32 GPIOs.
- [ ] Implement motor-control functions.
- [ ] Implement sensor-reading functions.
- [ ] Develop sensor calibration.
- [ ] Implement weighted line-position estimation.
- [ ] Implement PID steering controller.
- [ ] Implement line-loss detection.

### Phase 4 — PID Optimization
- [ ] Tune proportional gain.
- [ ] Tune derivative gain.
- [ ] Evaluate integral correction.
- [ ] Implement adaptive speed control.
- [ ] Test sharp turns and S-bends.
- [ ] Optimize overall lap time.

### Phase 5 — Testing and Validation
- [ ] Measure successful lap completion rate.
- [ ] Measure lap time.
- [ ] Evaluate tracking stability.
- [ ] Test under different battery conditions.
- [ ] Compare PID configurations.
- [ ] Document final performance.

---

## 📊 Performance Metrics

The robot will be evaluated using measurable parameters rather than visual performance alone.

| Metric | Purpose |
|---|---|
| Lap time | Measure overall speed |
| Successful lap rate | Measure reliability |
| Average tracking error | Measure accuracy |
| Maximum tracking error | Identify difficult track sections |
| Oscillation frequency | Evaluate stability |
| Recovery time | Evaluate line-loss handling |
| Battery voltage | Monitor power performance |

---

## 🛠️ Technology Stack

| Category | Technology |
|---|---|
| Microcontroller | ESP32 DevKit V1 |
| Firmware | C++ |
| Development Environment | PlatformIO / Arduino IDE |
| Control Algorithm | PID / PD |
| Line Detection | Five-channel IR sensor |
| Motor Driver | TB6612FNG |
| Motors | N20 Metal Gear DC Motors |
| Chassis | Three-wheel differential drive |
| Power | 2S Lithium Battery |
| Speed Control | PWM |
| Communication | Serial / Wi-Fi (planned) |

---

## 🔮 Future Improvements

- Encoder-based wheel-speed feedback.
- Independent PI motor-speed controllers.
- Live PID tuning through a wireless interface.
- Real-time sensor visualization.
- Data logging for performance analysis.
- Automatic PID parameter experimentation.
- Improved adaptive speed planning.
- More advanced line-loss recovery.

---

## 📁 Repository Structure

```text
PID-Line-Follower/
│
├── README.md
│
├── firmware/
│   ├── main.cpp
│   ├── motor_control.cpp
│   ├── sensor_processing.cpp
│   ├── pid_controller.cpp
│   └── config.h
│
├── hardware/
│   ├── wiring_diagram.png
│   ├── chassis_design.png
│   └── component_list.md
│
├── documentation/
│   ├── architecture.md
│   ├── pid_tuning.md
│   └── testing.md
│
└── media/
    ├── robot_front.jpg
    ├── robot_top.jpg
    └── testing.mp4
```

---

## 👨‍💻 Project Status

<details>
<summary>Development status</summary>

**Current stage:** Hardware selection and system architecture.

The project is under development. Hardware integration, firmware implementation, PID tuning and track testing are planned.

Performance claims will be updated after physical testing and validation.

</details>

## 📜 License

This project is intended for educational, experimental and robotics-development purposes.

Choose an appropriate open-source license before publishing the repository.

---

**Built with curiosity, engineering and a passion for robotics.** ⚡
