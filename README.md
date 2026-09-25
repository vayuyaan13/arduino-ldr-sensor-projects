# Arduino LDR Sensor Projects

An LDR (Light Dependent Resistor) is a light-sensitive component whose resistance changes with the amount of light falling on its surface. This guide covers the LDR full form, working principle, bare LDR versus module, Arduino Uno wiring, analog readings, Serial Monitor, an automatic night lamp, relative light-intensity readings, calibration, troubleshooting, applications, and practical project ideas.

## LDR full form

LDR stands for **Light Dependent Resistor**. It is also called a photoresistor or photocell.

For a concise explanation of the term and sensor basics, see [LDR full form](https://vayuyaan.com/blog/what-is-ldr-sensor/).

## Working principle

An LDR uses a light-sensitive semiconductor material. Its electrical resistance changes as light changes.

- In bright light, resistance generally decreases.
- In darkness, resistance generally increases.
- The response is gradual rather than simply on or off.

An LDR does not directly produce a voltage. For Arduino measurement, it is normally combined with a fixed resistor as a voltage divider.

A common arrangement is:

    5 V
     |
    [LDR]
     |
     +------ A0
     |
    [10 kΩ]
     |
    GND

With this arrangement, the voltage at A0 generally rises as light rises because LDR resistance falls. The exact response depends on the LDR and resistor values.

## LDR module vs bare LDR

### Bare LDR

A bare LDR is simply the light-dependent resistor. It has no built-in comparator, potentiometer, indicator LED, or complete sensor interface.

For an Arduino analog input, you provide the voltage-divider resistor yourself.

Advantages:

- Simple and inexpensive
- Easy to use in custom circuits
- Good for learning voltage-divider circuits
- Suitable when you want an analog reading

### LDR sensor module

A common LDR module combines an LDR with a resistor network and comparator circuit. Many modules expose VCC, GND, AO (analog output), and DO (digital output).

AO provides a changing voltage. DO provides a threshold-based digital signal, usually controlled by an onboard potentiometer.

Module designs differ, so check the labels and documentation for your exact board.

## Arduino Uno wiring

For a bare LDR voltage divider:

| Component | Connection |
|---|---|
| LDR leg 1 | 5V |
| LDR leg 2 | A0 |
| 10 kΩ resistor leg 1 | A0 |
| 10 kΩ resistor leg 2 | GND |

The LDR is not polarized, so either physical leg can be used in either position.

## Analog reading and Serial Monitor

The Arduino Uno uses a 10-bit ADC for its analog inputs. With the usual 5 V reference, analogRead() returns a value from 0 to 1023.

Example:

    const int LDR_PIN = A0;

    void setup() {
      Serial.begin(9600);
    }

    void loop() {
      int ldrValue = analogRead(LDR_PIN);
      Serial.println(ldrValue);
      delay(500);
    }

Upload the sketch and open the Serial Monitor. Set the Serial Monitor to 9600 baud.

Shine a light on the LDR and then cover it. The reading should change. The direction of that change depends on the voltage-divider arrangement.

## Understanding the analog value

An ADC value is not automatically a measurement in lux.

For a 5 V analog reference, the approximate input voltage is:

    Voltage = ADC reading × 5.0 / 1023

For example, a reading of 512 corresponds to about 2.50 V when the actual reference is 5 V.

This value is useful for relative light detection. It should not be presented as a calibrated lux value without suitable calibration and sensor characterization.

## Automatic night lamp

A basic night lamp turns an LED on when the measured light level falls below a chosen threshold.

### Components

- Arduino Uno
- LDR
- 10 kΩ resistor
- LED
- 220 Ω resistor
- Breadboard
- Jumper wires

### LED wiring

Connect Arduino digital pin 9 to a 220 Ω resistor, then to the LED anode. Connect the LED cathode to GND.

The series resistor limits LED current.

### Example code

The following example assumes the voltage-divider arrangement above, where the analog value is generally lower in darkness.

    const int LDR_PIN = A0;
    const int LED_PIN = 9;
    const int DARK_THRESHOLD = 400;

    void setup() {
      pinMode(LED_PIN, OUTPUT);
      Serial.begin(9600);
    }

    void loop() {
      int lightValue = analogRead(LDR_PIN);
      Serial.println(lightValue);

      if (lightValue < DARK_THRESHOLD) {
        digitalWrite(LED_PIN, HIGH);
      } else {
        digitalWrite(LED_PIN, LOW);
      }

      delay(200);
    }

The threshold is an example, not a universal value. It depends on the LDR, fixed resistor, supply, environment, and physical installation.

If your circuit produces higher readings in darkness, reverse the comparison.

## Relative light-intensity reading

You can display the ADC value and approximate voltage:

    const int LDR_PIN = A0;

    void setup() {
      Serial.begin(9600);
    }

    void loop() {
      int rawValue = analogRead(LDR_PIN);
      float voltage = rawValue * (5.0 / 1023.0);

      Serial.print("LDR: ");
      Serial.print(rawValue);
      Serial.print("  Voltage: ");
      Serial.print(voltage, 2);
      Serial.println(" V");

      delay(500);
    }

For relative light intensity, you can normalize readings between a measured dark value and a measured bright value:

    Relative % = (reading - darkValue) × 100 / (brightValue - darkValue)

Use measurements from your own circuit. If your divider produces the opposite direction, reverse the scale.

## Calibration

There is no universal LDR-to-lux conversion because LDR characteristics vary between components.

For a threshold project, calibrating the actual circuit is usually more useful than trying to calculate exact lux.

### Simple calibration procedure

1. Assemble the final circuit.
2. Put the LDR in its intended position.
3. Record several readings in the darkest expected condition.
4. Record several readings in the brightest expected condition.
5. Select a threshold between the observed ranges.
6. Test the circuit under different lighting conditions.
7. Adjust the threshold if the lamp switches too early or too late.

Averaging several readings can reduce noise:

    const int LDR_PIN = A0;

    int readAverageLDR() {
      long total = 0;

      for (int i = 0; i < 10; i++) {
        total += analogRead(LDR_PIN);
        delay(5);
      }

      return total / 10;
    }

    void setup() {
      Serial.begin(9600);
    }

    void loop() {
      Serial.println(readAverageLDR());
      delay(200);
    }

## Troubleshooting

### Serial Monitor always shows 0

Check the LDR, resistor, A0, GND, breadboard connections, and power supply.

### Reading stays near 1023

A0 may be pulled close to 5 V. Recheck the voltage-divider wiring and confirm that the fixed resistor is connected to GND in the arrangement used here.

### Reading changes in the opposite direction

This is not necessarily a fault. The direction depends on which resistor is connected to 5 V and which is connected to GND. Adjust the software comparison or swap the divider positions.

### LED flickers near the threshold

Light can fluctuate around the threshold. Average readings or use hysteresis with separate ON and OFF thresholds:

    const int LDR_PIN = A0;
    const int LED_PIN = 9;
    const int ON_THRESHOLD = 350;
    const int OFF_THRESHOLD = 450;

    bool lampOn = false;

    void setup() {
      pinMode(LED_PIN, OUTPUT);
      Serial.begin(9600);
    }

    void loop() {
      int value = analogRead(LDR_PIN);

      if (!lampOn && value < ON_THRESHOLD) {
        lampOn = true;
      }

      if (lampOn && value > OFF_THRESHOLD) {
        lampOn = false;
      }

      digitalWrite(LED_PIN, lampOn ? HIGH : LOW);
      Serial.println(value);
      delay(100);
    }

The threshold values must be calibrated for the actual circuit.

### LDR responds poorly

Check the sensor's physical position. Nearby lamps, LEDs, shadows, windows, and reflective surfaces can affect readings. Also check that the fixed resistor value suits the LDR's resistance range.

## Applications

LDRs are useful for detecting and comparing changes in ambient light.

Common applications include:

- Automatic night lamps
- Street-light prototypes
- Light-activated switches
- Display brightness control
- Light/dark detection
- Simple light-sensitive alarms
- Solar-light prototypes
- Classroom electronics projects
- Interactive robotics projects

For accurate illuminance measurement, a calibrated digital ambient-light sensor can be a better choice than an LDR.

## Practical project ideas

### 1. Automatic night lamp

Turn an LED on when ambient light falls below a calibrated threshold.

### 2. Automatic street-light prototype

Use several LEDs or an appropriate driver circuit to simulate a street-light system that activates in darkness.

### 3. Light-level monitor

Read the LDR continuously and display the value on the Serial Monitor or an LCD.

### 4. Dual-threshold light alarm

Trigger a buzzer or indicator when light becomes unusually high or low.

### 5. Automatic display brightness

Use the LDR reading to select LED brightness levels with PWM.

### 6. Light-following robot

Use multiple light sensors with motor control to make a robot move toward a stronger light source.

### 7. Room light monitor

Record relative readings over time and compare light levels at different locations.

## Key points

- LDR means Light Dependent Resistor.
- Its resistance generally decreases as light increases.
- A bare LDR needs a circuit such as a voltage divider for Arduino analog measurement.
- An LDR module may provide analog and comparator-based digital outputs.
- Arduino Uno analog inputs provide 10-bit readings from 0 to 1023 with the usual 5 V reference.
- An analog LDR reading is not automatically a lux measurement.
- Calibration is important for reliable threshold-based projects.
- LDR characteristics vary, so example thresholds must be adjusted for the actual circuit.
- For precise illuminance measurement, use a sensor designed and calibrated for that purpose.

## Repository purpose

This repository contains Arduino LDR sensor experiments, wiring examples, beginner projects, and practical light-detection applications.
