We use the "8 Channel 5V Relay Module" from Sainsmart (or similar)
to actuate signals that need more current than the Mesa 7i84 IO boards
can supply/sink.

<https://www.sainsmart.com/products/8-channel-5v-relay-module>

Each relay control input (IN1-IN8) needs to be grounded and sink less
than 1 mA to activate the relay (per '8 Relay Module.pdf').


8 channel relay board    |
10-pin control connector | Connected to
-------------------------+-------------------------
GND                      | GND on ATX power supply
IN1                      | 7i49 P4-18 ENA0+ (ENA0- is GND)
IN2                      | 7i49 P4-22 ENA1+ (ENA1- is GND)
IN3                      | 7i49 P3-22 ENA2+ (ENA2- is GND)
IN4                      |
IN5                      |
IN6                      |
IN7                      |
IN8                      |
VCC                      | +5V on ATX power supply


8 channel relay board    |
relay output pins        | Connected to
-------------------------+-------------------------
Ch1 common               | 0G
Ch1 NC                   |
Ch1 NO                   | CAM1-6 "READY1"
-------------------------+-------------------------
Ch2 common               | 0G
Ch2 NC                   |
Ch2 NO                   | CAM1-16 "READY2"
-------------------------+-------------------------
Ch3 common               | 0G
Ch3 NC                   |
Ch3 NO                   | CAM1-38 "READY3"
-------------------------+-------------------------
Channels 4-8 are currently unused.
