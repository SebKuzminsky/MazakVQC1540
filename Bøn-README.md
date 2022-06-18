Mazak VQC 15/40

TDS used:

    7i80HD anyio (3x50-pin)

    7i49 resolver servo interface, 50-pin

    7i44 (sserial breakout

    2*7i84 (the regular "not-D" version) (32 input/16 output each,
    sserial)
        7i84-A
            in: 19
                TB2 15: 1-8 10-16
                TB3  4: 1-4
            out: 13
        7i84-B
            in: 22
                TB2 12: 1-9 13 (14-16?)
                TB3 10: 1-6 9-12
            out: 16
        total:
            41 in
            29 out

    7i73 smart serial remote control panel interface

we're using:

    7i80HD-25 (AnyIO FPGA board)

        MAC 00:60:1B:11:81:AA

        W1, W2 in the DOWN position (default), IP address is 192.168.1.121

        W3 in the UP position (default), pull-up IO pins on power-up
        and reset

        W4 in the UP position (default), 5V tolerance mode enabled

        W5 in the UP position (default), boot from primary flash

        W6, W7, W8 all in the UP position (default), 5V on all 3 50-pin
        connectors

        P1 is connected to +5 & Ground (from an external power supply)

        Default firmware doesn't support resolvers (like the 7i49 has).
        Download 7i80.zip (Mesa's 7i80HD "Support Software" zip file)

    7i49 (6 channel resolver/servo interface)

        W1 is in the LEFT position (NOT default), accept external 5V power via P1

        W2 is in the UP position (default), standard resolver signal
        levels on all channels

        P1 is connected to +5 & Ground (from an external power supply)

    7i44 (8 channel RS-422/RS-485 interface)

        W1 is in the BOTTOM position (NOT default), accept external 5V power via P1

        P1 is connected to +5 & Ground (from an external power supply)

        It expects Cat6 568B pinout.  Is Cat5e ok?

        RS-422 SSLBP "smart serial"

    2x7i84 (32 in/16 out each, sserial)

        this is the sourcing output version, not the sinking output
        "D" version

        W1 is in the LEFT position (default), operate logic on field
        power, no connection to TB1 pin 5.

        W2 is in the LEFT position (default), 2.5 Mbaud

        TB1:
            pin 1-4 are connected to P24 on the Mazak (+24V DC)
            pin 5 is not connected (internally connected to VFIELDB via W1 in the left position)
            pin 6-8 are connected to G24 on the Mazak (ground for P24 power rail)


# NC Connectors

Mating pins?  https://www.radwell.co.uk/en-GB/Buy/HONDA/HONDA/MRP-M112

spec sheet here: https://www.onlinecomponents.com/en/datasheet/mrpm112-41909323

wants AWG 24-28 (0.5106-0.3211 mm conductor diameter, insulation OD 1.2-1.5 mm)

We don't have the correct mating connectors or pins, so we're using 0.72
mm² cross-sectional area stranded wire, to which we're attaching some
random pin connectors that we bought.  They friction-fit into the female
connectors ok.

The CND[1234] cable connectors are marked "Honda MR-50W".  The controller
should have the mating connector.

connectors between control computer and machine cabling:

    MR-50L Honda connectors?

    pins: "MRP-M1( )( )"?


# Mazak VQC 15/40 hardware

spindle drive: Mitsubishi Freqrol FR-SE

servo amps: Mitsubishi Meldas TRS50B BN624A559G52 AXO4D

The tool changer magazine run on hydraulic pressure using solenoids.
The door between the magazine and the work space, as well as the tool
height measuring arm run on air pressure.  The tool change button at
the back signal the computer and is not directly connected to the
magazine system.


## Power supplies

There are two 24V DC power systems:

1. P24 (+24 VDC) and G24 (ground) is always on, powered by the EC-11
power supply.

2. "+24" (+24 VDC) and "0G" (ground) can normally be turned on and off
using the green power-on button on the control panel.  Supplied by the
PD14C-1 power supply that's part of the NC.  Powers the computer and
a bunch of other stuff.  0G is connected to G24 when this power supply
is on.

We hardwired "0G" to "G24" with a jumper wire.

We disconnected the PD14C-1 power supply by the NC and instead wired a
small off-the shelf 24V DC (Mean Well HDR-60-24), 1.5A (100 VAC input)
power supply to the "+24"/"0G" power rail.  It's powered by the 100V
AC from the machine.

We're powering the 5V Mesa boards using a stand-alone ATX power supply.
This power supply's Ground is connected to 0G/G24.  The power supply
also provides the +12/-12 VDC for the resolvers.  FIXME: The eventual
final controller will likely run the LinuxCNC control computer off this
same power supply.


## Servos

The X and Y servos (and I think Z) are labeled:

    HA-83C-S
    Permanent Magnet AC Servo Motor
    Mitsubishi Electric Corp

The back of each servo has a "detector", aka "pickup unit" labeled:

    PICKUP UNIT
    TYPE: ATT-A-11
    SPEC NO: BKO-NC6198
    PARTS NO: RS2034N2E3
    TAMAGAWA SEIKI CO LTD

    A bundle of 7 wire/wire-pairs from this pickup unit go to the
    servo amp.  This is the resolver used by the servo amp.

Each ball screw has a "resolver", aka pickup unit mounted on the end
near the servo.  The one on the Y ball screw is labeled:

    Sanyo Denki
    P/N: 101
    SPEC NO: BKO-NC 6062
    TYPE: RT XC-11 (maybe "RT-4XC-11"?)
    PICKUP UNIT

    The bundle of 3 wire-pairs from each of these pickup units goes to
    the NC, via CNA-3, -4, and -5.  This is the resolver used by the NC
    for position control.

    The machine's resolver connector (CNA3 etc) needs +12, -12V, and
    Ground.  We used +12/-12 from the ATX power supply that powers the
    Mesa boards.  Its +12 goes to CNA{3,4,5} pin "P12", its -12V goes to
    "M12", and two "AG" pins are connected to 0G.


## Servo amps

This machine has TRS-50B servo amps.  Each servo amp presents these pins
on CAM1:

Drive Pin Name | Notes
---------------+----------------------------------------------------------
ER             |
ERR            |
---------------+----------------------------------------------------------
SC             |
ALM            |
---------------+----------------------------------------------------------
READY          | Controller pulls this to ground to enable servo power.
SVON           | Controller pulls this to ground to enable servo control.
---------------+----------------------------------------------------------
+24V           | Connect to +24V power
RG             | Connect to Ground
AG             | Connect to Ground
---------------+----------------------------------------------------------

Give the servo amp power and ground.  When the controller wants axis
motion, pull READY to ground, then pull SVON to ground.  Mesa 7i84
boards have current sourcing outputs and can only pull up, so use the
7i84 output to activate a relay that connects the servo amp pin to ground.

X: Negative control voltage moves table left (ie moves machine in +X
direction).

Y: Negative control voltage moves spindle away from operator.  Spindle
drifts towards operator slowly - badly tuned servo amp?

Z: Spindle drifts upwards slowly - badly tuned Z servo amp?  Negative
control voltage moves spindle down (-0.005 is a nice slow pace).
Positive control voltage does not appear to move spindle up.  Enable Z
servo amp and release ABRK at the same time.


# To bring machine up

Apply pressurized air (what pressure?).

Open the manual air valve at the machine's air intake.

Turn on "+24V"/"0G" power supply.

Release all E-stop buttons (1 by operator console, 1 by tool magazine
access door, 1 optional on "handy controller").

Power on MAR relay ("power-on" net).

Power on the hydraulic pump ("hydraulic-lube-pump-on" net).

Activate the tool magazine main solenoid ("magazine-run" net).

Enable power to the three axis servos by setting pwmgen.00.enable true
(turns on power on all three servo amps).

Enable servo control by setting servo-on-{1,2,3} to 1.  Be careful,
when setting servo-on-3 true, the Z servo amp will take control of the
Z axis, so the ABRK relay needs to be released at the same time (via the
"Servos Ready"/"SA" line).

Powering on the SE relay (via "Servos Ready"/"SA") will release the
Z brake, and if the Z servo is not running then the Z axis will drop
towards the table.
