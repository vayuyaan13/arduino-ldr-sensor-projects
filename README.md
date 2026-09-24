# Arduino LDR Sensor Projects

A practical beginner guide to LDRs, Arduino Uno wiring, analog readings, automatic lighting, calibration, and troubleshooting.

## What is an LDR?

LDR stands for Light Dependent Resistor. Its resistance changes with the amount of light falling on it. In bright light, resistance typically decreases; in darkness, it typically increases.

For a detailed explanation of the component and its applications, see the [LDR full form](https://vayuyaan.com/blog/what-is-ldr-sensor/) guide.

## LDR module vs bare LDR

A bare LDR normally needs a fixed resistor to form a voltage divider. An LDR module may include a comparator, potentiometer, and indicator LED.

Use a bare LDR when you want an analog light reading. Use a module when you mainly need an adjustable light/dark threshold.

## Components

- Arduino Uno
- LDR
- 10 kΩ resistor
- Breadboard
- Jumper wires
- LED and 220–330 Ω resistor

## Basic wiring

Build a voltage divider:

- LDR leg 1 → 5V
- LDR leg 2 → A0
- 10 kΩ resistor → A0 to GND

The voltage at A0 changes as the LDR resistance changes.

## Read the LDR

~~~cpp
const int LDR_PIN = A0;

void setup() {
  Serial.begin(9600);
}

void loop() {
  int lightValue = analogRead(LDR_PIN);
  Serial.println(lightValue);
  delay(250);
}
~~~

Open Serial Monitor at 9600 baud and test the sensor in bright and dark conditions.

## Automatic night light

~~~cpp
const int LDR_PIN = A0;
const int LED_PIN = 9;
const int THRESHOLD = 400;

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  int lightValue = analogRead(LDR_PIN);

  if (lightValue < THRESHOLD) {
    digitalWrite(LED_PIN, HIGH);
  } else {
    digitalWrite(LED_PIN, LOW);
  }

  delay(100);
}
~~~

The comparison direction depends on how the voltage divider is arranged. Calibrate the threshold for your actual circuit.

## Calibration

Do not assume that a threshold from another project will work on your circuit. LDR characteristics, resistor value, supply voltage, sensor position, and ambient light all affect the reading.

1. Record the reading in bright light.
2. Record it in the darkest expected condition.
3. Choose a threshold between the two ranges.
4. Test repeatedly.
5. Average several readings if the value fluctuates.

## Troubleshooting

**Reading stays near 0 or 1023:** check the divider and ground.

**Reading changes in the opposite direction:** reverse the comparison or swap the LDR and fixed resistor positions.

**LED flickers near the threshold:** average samples or add hysteresis.

**Values barely change:** expose the LDR to different light levels and check the resistor and wiring.

## Applications

- Automatic night lamps
- Day/night detection
- Light-sensitive alarms
- Display brightness control
- Simple light meters
- Garden lighting
- Robotics experiments

## Safety

Keep the Arduino circuit at low voltage. Never connect an Arduino pin directly to mains voltage. Mains-powered lighting requires appropriate isolated switching hardware and qualified electrical work.

## Next projects

Try adding an LCD, logging readings, controlling LED brightness with PWM, or combining the LDR with a motion sensor.

## License

This project is provided under the repository license.
