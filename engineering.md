# PID Line-Following Robot — Engineering Factors & Setup Guide

**Project Stack:** ESP32 DevKit V1 · 5-Channel IR Sensor Array · TB6612FNG · 2 × N20 Motors · 2S Battery · 3-Wheel Differential Drive

This document serves as a technical reference for designing, assembling, calibrating, and optimizing the PID-based line-following robot.

The performance of a PID controller depends on the complete robotic system, including sensor positioning, motor characteristics, chassis geometry, battery voltage, track conditions, and software timing.

The objective is to measure and optimize these factors systematically rather than rely on arbitrary PID values.

---

# 1. Current Hardware Configuration

| Component | Selected Hardware | Status |
|---|---|---|
| Microcontroller | ESP32 DevKit V1 (ESP-WROOM-32) | Selected |
| Line Sensor | 5-Channel IR Sensor Array | Selected |
| Motor Driver | TB6612FNG | Selected |
| Motors | 2 × 12V N20 Metal Gear Motors | Selected |
| Battery | 2S Lithium Battery (7.4V nominal) | Selected |
| Drive Wheels | 2 × Rubber Wheels | Selected |
| Support Wheel | Ball Caster | Selected |
| Chassis | Custom 3-Wheel Differential Drive | To be fabricated |

**Important electrical consideration:** A 2S battery supplies 7.4V nominally and 8.4V when fully charged. Your 12V N20 motors will operate below their rated voltage. Actual performance will depend on the motor's load, driver voltage drop, battery condition, and motor specifications.

The motor stall current must be checked against the TB6612FNG's operating limits before connecting the motors.

---

# 2. Chassis Design and Mechanical Configuration

## 2.1 Recommended Chassis Shape

The recommended chassis is a compact, rounded arrowhead or trapezoidal design.

The design should prioritize:

- Low overall weight.
- Low centre of gravity.
- Mechanical rigidity.
- Balanced weight distribution.
- Proper sensor positioning.
- Minimum unnecessary friction.

The front section should accommodate the five-channel sensor array. The middle section should hold the ESP32, motor driver, and battery. The two N20 motors should be mounted symmetrically on opposite sides.

The ball caster should be mounted along the chassis centreline.

The shape itself does not improve PID performance. Its purpose is to provide a mechanically stable platform with suitable sensor and motor geometry.

## 2.2 Three-Wheel Configuration

The robot will use differential-drive steering.

| Wheel | Function |
|---|---|
| Left wheel | Powered by left N20 motor |
| Right wheel | Powered by right N20 motor |
| Ball caster | Passive support |

The two powered wheels control forward movement, reverse movement, and turning by varying their relative speeds.

A ball caster is preferred over a conventional swivel caster because it can roll in any horizontal direction without requiring a swivel mechanism to realign.

However, caster friction, mounting height, and weight distribution must still be checked.

## 2.3 Important Chassis Dimensions

Record the following measurements during fabrication.

| Parameter | Description | Measurement |
|---|---|---|
| Chassis length | Total front-to-back length | ______ mm |
| Chassis width | Total left-to-right width | ______ mm |
| Wheel diameter | Outside diameter of rubber wheels | ______ mm |
| Track width | Distance between drive-wheel centres | ______ mm |
| Wheelbase | Distance between drive axle and caster contact point | ______ mm |
| Sensor-to-axle distance | Distance from sensor sensing line to drive axle | ______ mm |
| Sensor bar width | Distance between first and last sensor | ______ mm |
| Total weight | Complete robot weight | ______ g |
| Battery position | Position relative to chassis centre | ______ mm |

### Why these dimensions matter

**Wheel diameter:** Determines the distance travelled per wheel revolution and affects linear speed.

**Track width:** Influences turning behaviour and the robot's rotational response.

**Sensor-to-axle distance:** Determines how early the robot detects an upcoming curve.

**Chassis weight:** Affects acceleration, braking, traction, and turning behaviour.

**Centre of gravity:** Influences stability, especially during sharp turns and rapid direction changes.

---

# 3. IR Sensor Array Positioning

The five-channel IR sensor array is the primary feedback source for the PID controller.

Its mounting position directly affects line detection, tracking accuracy, and steering response.

## 3.1 Sensor Height

There is no universal sensor height suitable for every IR array.

The correct height depends on:

- Sensor module design.
- Infrared emitter and receiver characteristics.
- Optical shrouding.
- Track surface.
- Line material.
- Ambient lighting.

### Recommended procedure

1. Check the sensor manufacturer's recommended operating distance.
2. Mount the sensor array parallel to the track.
3. Use an adjustable mounting bracket.
4. Begin within the manufacturer's recommended distance.
5. Read sensor values over both the line and background.
6. Select a height that produces strong and consistent contrast.
7. Recalibrate after changing the height.

Do not blindly use a fixed 3–8 mm height. This is only a possible test range for some sensor modules.

### Sensor Height Record

| Parameter | Value |
|---|---|
| Sensor model | __________ |
| Sensor height | ______ mm |
| Line colour | __________ |
| Background colour | __________ |
| Line width | ______ mm |
| Surface material | __________ |
| Ambient lighting | __________ |

## 3.2 Sensor-to-Axle Distance

The sensor array should generally be positioned ahead of the drive axle.

This allows the robot to detect an upcoming turn before its wheels reach that section.

However:

- Too little distance can result in late turn detection.
- Too much distance can cause premature steering corrections and oscillation.
- The optimal distance depends on speed, track curvature, chassis geometry, and control-loop delay.

Use an adjustable mounting arrangement during testing.

## 3.3 Analog vs Digital Sensor Output

### Analog Sensors

Analog sensors provide a range of readings rather than only HIGH and LOW.

Advantages:

- Better line-position estimation.
- More detailed reflectance information.
- Smoother weighted-average calculations.
- Better potential tracking resolution.

### Digital Sensors

Digital sensors provide binary detection.

Advantages:

- Simple implementation.
- Lower processing requirements.
- Straightforward sensor-state interpretation.

However, digital readings provide less information about the actual position of the line.

**Recommendation:** Prefer analog outputs if your sensor module supports them reliably.

ESP32 GPIOs are not 5V tolerant. Always verify sensor output voltage before connecting it to the ESP32.

## 3.4 Sensor Weight Assignment

For five sensors arranged from left to right:

| Sensor | Position | Weight |
|---|---|---:|
| S1 | Far Left | -2000 |
| S2 | Left | -1000 |
| S3 | Centre | 0 |
| S4 | Right | +1000 |
| S5 | Far Right | +2000 |

These are software position weights, not physical distances.

For calibrated line-strength values:

\[
Position = \frac{\sum w_iq_i}{\sum q_i}
\]

Where:

- \(w_i\) = Position weight.
- \(q_i\) = Calibrated line-detection strength.

The error is:

\[
Error = Position - TargetPosition
\]

The target position is normally zero.

If the sensor readings indicate that no line is detected, the controller must recognize line loss instead of dividing by zero.

## 3.5 Sensor Calibration

Before PID tuning:

1. Read all sensors over the background.
2. Move the array over the line.
3. Record the readings of each sensor independently.
4. Repeat the process several times.
5. Determine the minimum and maximum readings for each channel.
6. Normalize readings using individual calibration ranges.
7. Verify that line and background readings are distinguishable.
8. Test on actual track material and lighting conditions.

### Calibration Table

| Sensor | Background Reading | Line Reading |
|---|---:|---:|
| S1 | ______ | ______ |
| S2 | ______ | ______ |
| S3 | ______ | ______ |
| S4 | ______ | ______ |
| S5 | ______ | ______ |

---

# 4. Motor Specifications and Performance

The two N20 motors are responsible for the robot's movement and steering.

Their characteristics directly influence the PID controller's response.

## 4.1 Motor Parameters

The following specifications should be identified.

| Parameter | Description |
|---|---|
| Rated Voltage | Intended operating voltage |
| RPM | Motor output speed after gearbox |
| Gear Ratio | Determines speed and torque characteristics |
| Stall Current | Maximum current under a locked shaft condition |
| No-load Current | Current consumed without mechanical load |
| Rated Current | Current during normal specified operation |
| Torque | Rotational force produced by the gearbox |
| Encoder | Optional feedback for measuring wheel rotation |

### Current Motor Configuration

- Motor quantity: 2
- Motor type: N20 metal gearmotor
- Rated voltage: 12V
- RPM: To be confirmed
- Stall current: To be confirmed
- Encoder: To be confirmed

Do not intentionally stall a motor for a prolonged period to measure its current. Prefer manufacturer specifications or a suitable current-limited measurement method.

## 4.2 Motor RPM

RPM determines how quickly the motor shaft rotates.

Higher RPM generally produces higher wheel speed, but it does not automatically mean better performance.

Excessively high motor speed can result in:

- Difficulty handling sharp turns.
- Increased overshooting.
- Reduced available torque at the wheel.
- Increased sensitivity to steering corrections.

The ideal motor RPM depends on:

- Wheel diameter.
- Gear ratio.
- Chassis weight.
- Track curvature.
- Battery voltage.
- Motor torque.
- Required lap time.

## 4.3 Motor Torque

Torque determines the motor's ability to rotate the wheels under load.

Insufficient torque can cause:

- Slow acceleration.
- Difficulty changing direction.
- Wheel-speed drops under load.
- Inconsistent performance during sharp turns.

A suitable balance between RPM and torque is more important than maximizing either one individually.

## 4.4 Left and Right Motor Matching

Even identical motors can have slightly different RPM characteristics.

This can cause the robot to drift towards one side.

Test both motors independently and compare:

- Startup PWM.
- No-load speed.
- Loaded speed.
- Direction.
- Current consumption.

If encoders are available, independent wheel-speed feedback can compensate for motor differences.

---

# 5. Wheel and Traction Factors

Wheel characteristics influence the robot's movement and turning stability.

| Factor | Effect |
|---|---|
| Wheel Diameter | Changes linear speed |
| Wheel Width | Influences contact area and grip |
| Rubber Material | Determines traction |
| Wheel Alignment | Affects straight-line movement |
| Wheel Friction | Influences motor load |
| Wheel Weight | Influences rotational inertia |

### Important considerations

- Both drive wheels should have matching diameters.
- Both wheels should be mounted parallel to the chassis.
- Wheel hubs should be firmly attached.
- Avoid excessive wheel wobbling.
- Use suitable rubber tyres for the track surface.
- Check that the caster does not create excessive friction.

Poor traction can cause the robot to lose control even when PID parameters are correctly tuned.

---

# 6. TB6612FNG Motor Driver

The selected motor driver is the TB6612FNG dual H-bridge.

It independently controls the two DC motors.

### Main connections

| Driver Pin | Function |
|---|---|
| VM | Motor supply |
| VCC | Logic supply |
| GND | Common ground |
| AIN1 | Motor A direction |
| AIN2 | Motor A direction |
| PWMA | Motor A speed |
| AO1, AO2 | Motor A terminals |
| BIN1 | Motor B direction |
| BIN2 | Motor B direction |
| PWMB | Motor B speed |
| BO1, BO2 | Motor B terminals |
| STBY | Driver standby control |

### Important electrical conditions

- Verify the driver's voltage ratings.
- Verify the stall current of both motors.
- Ensure the driver can handle the required current and thermal load.
- Connect the ESP32 and driver grounds together.
- Use appropriate decoupling capacitors.
- Keep motor wiring away from sensor signal wiring.

Do not select a motor driver based only on its advertised peak-current rating.

---

# 7. Battery and Power Supply

The current battery configuration is:

**2S Lithium Battery**

| Parameter | Specification |
|---|---|
| Configuration | 2S |
| Nominal Voltage | 7.4V |
| Fully Charged Voltage | 8.4V |
| Application | Motor power and regulated electronics supply |

## 7.1 Power Distribution

The power system should separate motor power from regulated logic power.

```text
                 2S BATTERY
             7.4V / 8.4V
                    |
              MAIN SWITCH
                    |
                  FUSE
                    |
              POWER SPLIT
                /       \
               /         \
          TB6612 VM    BUCK CONVERTER
          Motor Supply   Regulated 5V
               |             |
          N20 Motors       ESP32
```

The TB6612FNG logic supply (VCC) should receive an appropriate logic voltage, typically 3.3V in this system.

All grounds must be connected together.

## 7.2 Battery Voltage

Battery voltage affects motor speed and available torque.

As battery voltage decreases:

- Motor speed can decrease.
- Acceleration can change.
- Motor response can become inconsistent.
- PID behaviour may change.

Record battery voltage before and after testing.

## 7.3 Power Stability

Motors can produce electrical noise and sudden current demand.

Possible symptoms include:

- ESP32 resets.
- Unstable sensor readings.
- Sudden motor-speed changes.
- Unexpected controller behaviour.

Recommended measures:

- Use a separate regulated supply for the ESP32.
- Keep motor-current wiring short and adequately sized.
- Use suitable decoupling.
- Keep motor wires away from sensor wiring.
- Use a suitable battery protection system.
- Install a main switch and fuse.

Use an appropriate charger for the battery chemistry and cell count. A standard 2S lithium-ion or lithium-polymer pack requires an appropriate 8.4V CC/CV charging system.

---

# 8. ESP32 Pin Configuration

The following is a proposed pin allocation for the classic ESP32 DevKit V1.

## 8.1 TB6612FNG Connections

| ESP32 GPIO | TB6612FNG Pin |
|---|---|
| GPIO 25 | AIN1 |
| GPIO 26 | AIN2 |
| GPIO 18 | PWMA |
| GPIO 27 | BIN1 |
| GPIO 14 | BIN2 |
| GPIO 19 | PWMB |
| GPIO 23 | STBY |

## 8.2 Five-Channel Sensor Connections

| Sensor | ESP32 GPIO |
|---|---|
| S1 | GPIO 36 |
| S2 | GPIO 39 |
| S3 | GPIO 34 |
| S4 | GPIO 35 |
| S5 | GPIO 32 |

These sensor pins are ADC1-capable inputs on the classic ESP32.

GPIO 34–39 are input-only, which is suitable for sensor readings.

Verify the exact board revision and pin availability before wiring.

If adding quadrature encoders, reserve up to four additional input signals and revise the pin allocation.

**Never connect a 5V signal directly to an ESP32 GPIO.**

---

# 9. PID Control Parameters

The PID controller calculates steering correction using three components.

\[
Output = K_pE + K_i\int E\,dt + K_d\frac{dE}{dt}
\]

Where:

- \(E\) = Current tracking error.
- \(K_p\) = Proportional gain.
- \(K_i\) = Integral gain.
- \(K_d\) = Derivative gain.

## 9.1 Proportional Gain (Kp)

Proportional control responds to the current tracking error.

\[
P = K_p \times Error
\]

### Factors affecting Kp

- Current tracking error.
- Robot speed.
- Track curvature.
- Sensor spacing.
- Sensor-to-axle distance.
- Motor response.
- Steering authority.
- Chassis geometry.

**Kp too low:**
- Slow correction.
- Weak steering response.
- Possible line loss on sharp turns.

**Kp too high:**
- Excessive steering.
- Zigzagging.
- Overshooting.
- Oscillations.

## 9.2 Integral Gain (Ki)

Integral control corrects accumulated tracking error.

\[
I = K_i \times \sum(Error \times \Delta t)
\]

### Factors affecting Ki

- Persistent motor mismatch.
- Wheel alignment.
- Mechanical friction differences.
- Sensor bias.
- Duration of accumulated error.
- Motor output saturation.

**Ki too low:** Persistent tracking bias may remain.

**Ki too high:** Integral windup and excessive overshooting can occur.

Start with Ki = 0. Add integral correction only if testing demonstrates persistent tracking bias.

## 9.3 Derivative Gain (Kd)

Derivative control responds to how quickly the tracking error changes.

\[
D = K_d \times \frac{Error-PreviousError}{\Delta t}
\]

### Factors affecting Kd

- Rate of change of tracking error.
- Robot speed.
- Sensor noise.
- Sampling frequency.
- Motor response delay.
- Chassis inertia.

**Kd too low:** Insufficient damping and excessive overshooting.

**Kd too high:** Noisy and abrupt steering corrections.

Filtering the derivative term can reduce noise, but excessive filtering introduces delay.

## 9.4 Important Gain Consideration

PID gains depend on the units and scale used for tracking error.

For example, gains tuned for an error range of -2000 to +2000 cannot be directly reused if the error is normalized to -1 to +1.

The same applies when changing control-loop timing or sensor geometry.

---

# 10. Control Loop Frequency

The control loop should execute at a consistent interval.

A starting experimental range could be 200–500Hz, provided the sensor and firmware can support it.

The correct frequency depends on:

- Sensor acquisition time.
- PID calculation time.
- PWM update rate.
- Motor response.
- Sensor noise.
- Execution-time consistency.

Avoid blocking delays inside the control loop.

Avoid excessive serial printing because it can introduce timing inconsistencies.

Measure the actual loop period rather than assuming that the requested frequency is being achieved.

---

# 11. PWM and Motor Speed Control

PWM controls the average power delivered to the motors.

The motor speed depends on:

- PWM duty cycle.
- Battery voltage.
- Motor characteristics.
- Mechanical load.
- Wheel traction.
- Driver voltage drop.

The robot should use independent PWM commands for its left and right motors.

\[
LeftSpeed = BaseSpeed + PIDOutput
\]

\[
RightSpeed = BaseSpeed - PIDOutput
\]

The exact correction signs must be verified experimentally.

Motor commands must be constrained within valid limits.

---

# 12. Adaptive Speed Control

A fixed base speed may not be ideal for every section of the track.

The robot should eventually use adaptive speed management.

| Tracking Condition | Expected Behaviour |
|---|---|
| Small error | Maintain or increase speed |
| Moderate error | Reduce speed |
| Large error | Reduce speed significantly |
| Rapidly changing error | Anticipate the turn and slow down |
| Line lost | Stop or initiate recovery |

This allows the robot to travel faster on straight sections while maintaining control through sharp curves.

The instantaneous tracking error is only an approximation of track curvature, so the speed strategy must be validated on the actual track.

---

# 13. Environmental Conditions

Environmental factors can change sensor readings and vehicle behaviour.

| Environmental Factor | Possible Effect |
|---|---|
| Ambient Lighting | Changes IR sensor readings |
| Surface Reflectivity | Changes detection contrast |
| Dust | Reduces sensor reliability |
| Line Thickness | Affects position estimation |
| Track Texture | Changes rolling resistance |
| Surface Friction | Influences turning and traction |
| Uneven Surface | Changes sensor height |
| Battery Discharge | Changes motor response |
| Temperature | Affects motor and electronics performance |

Test under the same lighting and track conditions expected during competition.

---

# 14. Line-Loss Detection and Recovery

The controller should detect when the line is no longer visible.

Possible line-loss conditions:

- All five sensors detect the background.
- Total calibrated line strength falls below a threshold.
- Sensor readings become invalid or inconsistent.

### Recovery Strategy

1. Reduce speed or stop.
2. Use the last valid tracking direction.
3. Search slowly within a bounded steering range.
4. Resume normal PID control when the line is detected again.
5. Stop safely if recovery fails within a defined time or distance.

Do not continue applying the previous steering command indefinitely.

---

# 15. Systematic PID Tuning Procedure

PID tuning should be performed only after verifying the mechanical and electrical systems.

## Stage 1 — Mechanical Validation

- Check wheel alignment.
- Verify both wheels rotate freely.
- Check ball-caster movement.
- Verify chassis rigidity.
- Confirm sensor alignment.
- Check weight distribution.

## Stage 2 — Electrical Validation

- Verify battery voltage.
- Check regulated ESP32 supply.
- Verify motor-driver wiring.
- Confirm motor direction.
- Check motor current compatibility.
- Check driver temperature.

## Stage 3 — Sensor Calibration

- Calibrate all five channels.
- Verify sensor polarity.
- Confirm centred line gives approximately zero error.
- Test different track positions.
- Verify line-loss detection.

## Stage 4 — PID Tuning

1. Set Ki = 0 and Kd = 0.
2. Start with a low base speed.
3. Gradually increase Kp.
4. Observe when oscillation begins.
5. Reduce Kp to a stable value.
6. Gradually increase Kd.
7. Add Ki only if persistent tracking bias remains.
8. Test on straight sections, curves and S-bends.

## Stage 5 — Speed Optimization

1. Keep the tuned steering controller fixed.
2. Increase base speed gradually.
3. Perform multiple laps.
4. Record lap time and tracking accuracy.
5. Check the successful lap rate.
6. Repeat with different battery conditions.

The goal is to achieve the fastest repeatable lap time, not merely the fastest single run.

---

# 16. Performance Evaluation Metrics

Performance should be measured objectively.

| Metric | Purpose |
|---|---|
| Lap Time | Measures overall speed |
| Successful Lap Rate | Measures reliability |
| Average Tracking Error | Measures accuracy |
| Maximum Tracking Error | Identifies difficult sections |
| Oscillation Frequency | Evaluates stability |
| Recovery Time | Evaluates line-loss handling |
| Battery Voltage | Monitors power performance |
| Motor Temperature | Monitors operating conditions |
| Driver Temperature | Monitors thermal performance |

---

# 17. Test and Tuning Log

Maintain a record of every experiment.

| Test | Kp | Ki | Kd | Base PWM | Loop Hz | Sensor Height | Battery Voltage | Lap Time | Successful Laps | Observations |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | | 0 | 0 | | | | | | | |
| 2 | | 0 | | | | | | | | |
| 3 | | | | | | | | | | |

Also record:

- Average tracking error.
- Maximum tracking error.
- Number of line losses.
- Recovery time.
- Motor temperature.
- Driver temperature.
- Any ESP32 resets.
- Any unexpected sensor readings.

Change one major variable at a time to understand its effect.

---

# 18. Final Build Checklist

## Mechanical

- [ ] Two matching drive wheels.
- [ ] Motors securely mounted.
- [ ] Wheels aligned correctly.
- [ ] Ball caster moves freely.
- [ ] Chassis is rigid.
- [ ] Battery secured.
- [ ] Sensor height adjustable.
- [ ] Sensor-to-axle distance adjustable.

## Sensors

- [ ] Exact sensor module identified.
- [ ] Analog/digital output type confirmed.
- [ ] Output voltage safe for ESP32.
- [ ] Sensor order verified.
- [ ] Calibration completed.
- [ ] Ambient-light testing completed.

## Electronics

- [ ] TB6612FNG pinout verified.
- [ ] Motor stall current checked.
- [ ] Battery and charger compatible.
- [ ] Regulated ESP32 supply verified.
- [ ] Common ground established.
- [ ] Main switch installed.
- [ ] Fuse installed.
- [ ] Electrical noise checked.

## Firmware

- [ ] Motor-control functions implemented.
- [ ] Sensor-reading functions implemented.
- [ ] Sensor calibration implemented.
- [ ] Weighted position estimation implemented.
- [ ] PID controller implemented.
- [ ] Output saturation implemented.
- [ ] Fixed-period control loop verified.
- [ ] Line-loss recovery implemented.
- [ ] Telemetry and logging implemented.

---

# 19. Core Engineering Principle

**Do not use PID tuning to compensate for mechanical or electrical faults.**

Before changing PID gains, verify:

- Sensor calibration.
- Sensor mounting height.
- Sensor-to-axle distance.
- Wheel alignment.
- Motor matching.
- Battery voltage.
- Driver current capability.
- Control-loop timing.
- Track conditions.

A robot with good mechanical design, stable power, accurate sensor readings and predictable motor response will be significantly easier to tune.

The ultimate objective is to minimize lap time while maintaining consistent tracking accuracy, stability and reliability.

---

**Project Philosophy:** Measure first. Understand the system. Tune scientifically. Optimize through testing.
