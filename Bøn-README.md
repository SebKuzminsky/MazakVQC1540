Mazak VQC 15/40

# Mesa interface hardware

    7i80HD-25 (AnyIO FPGA board, 3x50-pin connectors)

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

    Sainsmart 8-channel 5V relay board

        Used to provide pull-downs for the TRS-50B `NC READY` and
        `SERVO ON` signals.  (The TRS-50B pulls those lines to +24V
        internally, the controller must pull them both to ground to
        enable the servo amp.  We can't use the 7i84 for this since it
        has sourcing outputs.)

        The jumper on the board connects VCC to JD-VCC.


# Mazak VQC 15/40 hardware


## NC Connectors

The CND[1234] and CAM1 cable connectors are marked "Honda MR-50W".

The CND5 cable connector is marked "Honda MR-20W".

The CNA[345] cable connectors are marked "Honda MR-20L".

The controller should have the mating connectors for those.


### Pins

Mating pins?  https://www.radwell.co.uk/en-GB/Buy/HONDA/HONDA/MRP-M112

spec sheet here: https://www.onlinecomponents.com/en/datasheet/mrpm112-41909323

wants AWG 24-28 (0.5106-0.3211 mm conductor diameter, insulation OD 1.2-1.5 mm)

We don't have the correct mating connectors or pins, so we're using 0.72
mm² cross-sectional area stranded wire, to which we're attaching some
random pin connectors that we bought.  They friction-fit into the female
connectors ok.


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

The servo amps are labeled: Mitsubishi Meldas TRS-50B BN624A559G52 AXO4D

Each servo amp presents these pins on CAM1:

    Drive Pin Name | Notes
    ---------------+----------------------------------------------------------
    ER             | Analog velocity control voltage, -10 to +10
    ERR            | Ground reference for ER
    ---------------+----------------------------------------------------------
    SC             | FIXME Is this input or output?  What is its function?  It is in a twisted pair with ALM.  Rounding it when ALM is set did nothing.
    ALM            | 0V when amp is happy, 16.5V (!) when amp is alarmed.  Need pullup resistor.  47K resistor worked.
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

X: Negative control voltage moves table left (ie moves tool in
+X direction).  We set pwmgen.00.scale to -1 so that a positive
pwmgen.00.value moves the tool in the +X direction (ie moves the table
in the -X direction).  It is not an alternative to switch the ER and
ERR pins on the Mesa card, as ERR is also ground 0G, this might cause
unwanted currents to flow.  The work table drifts slowly to the right
when the command voltage is zero.  Doing 'sets x-output 0.00048'
almost stops it.

Y: Negative control voltage moves spindle away from operator.  We set
pwmgen.01.scale to -1 so that a positive pwmgen.01.value moves the
tool in the +Y direction.  Spindle drifts towards operator slowly -
badly tuned servo amp?

Z: Spindle drifts upwards slowly - badly tuned Z servo amp?  Negative
control voltage moves spindle down (-0.005 is a nice slow pace).
This is the intuitive direction, so we set pwmgen.02.scale to 1 so that
a positive pwmgen.02.value moves the spindle up, in the +Z direction.
Note that ABRK must be active, so that the Z brake is released, before
trying to move the Z axis.


## Spindle drive

spindle drive: Mitsubishi Freqrol FR-SE-2-11K

When enabling Spindle and trying to run forward or backwards, by
activating relays SSET and SMR using 'sets spindle-run 1', 'sets
spindle-set 1' and 'sets spindle-cw 1', the spindle controller trip
and report 'Inverter I O C'.  According to page 32 in
Mitsubishi-Freqrol-FR-SE-Maintenance-Manual.pdf, this can be caused by
faulty power transistors or diods in the controller.  Conducting the
tests listed there did not show any issues:

Waited until machine had been turned off for 4 days

Tested 0+20 voltage beteen P and N is 0.00

Tested 0-200 ohm resistance between P and R3, S3 and T3 is infinite
Tested 0-200 ohm resistance between N and R3, S3 and T3 is infinite
Tested 0-2000 ohm resistance between P and R3, S3 and T3 is around 400 ohm
Tested 0-2000 ohm resistance between N and R3, S3 and T3 is around 400 ohm and rising

Tested 0-200 ohm resistance between P and U, V and W is infinite
Tested 0-200 ohm resistance between N and U, V and W is infinite
Tested 0-2000 ohm resistance between P and U, V and W is around 500 ohm
Tested 0-2000 ohm resistance between N and U, V and W is around 500 ohm and rising

New measurement with Caltek CM100 multimeter after machine had been
turned off for 10 hours:

Tested 0-500 voltage between P and N is 0.00.
Tested 0-2000m voltage between P and N is 000 - 001.

Tested 0-2000 ohm resistance between P and R3, S3 and T3 is 366, 367, 367 ohm.
Tested 0-2000 ohm resistance between N and R3, S3 and T3 start below 100 and raises during measurement.

Tested 0-2000 ohm resistance between P and U, V and W is 350, 351, 356 ohm.
Tested 0-2000 ohm resistance between N and U, V and W is start around 200 and raises during measurement.

The transistors are marked QM150DY-H and BKO-C2144H01.

Some ideas:

 * https://forum.linuxcnc.org/30-cnc-machines/32162-mitsubishi-freqrol-fr-se-replacement-recommendations
 * https://www.practicalmachinist.com/forum/threads/spindle-drive-issues-leadwell-mcv-0p-3-axis-vmc-with-mitsubish-freqrol-fr-sf-spindl.280843/
 * https://carelabz.com/megger-test-performed/


## Tool Magazine

The tool magazine runs on hydraulic pressure using solenoids.

The door between the magazine and the work space, as well as the tool
height measuring arm run on air pressure.

The tool change button at the back is connected to the control computer,
not directly to the tool magazine.

To unclamp the tool in the spindle, the spindle-set signal must be
true, spindle-run signal must be false and the spindle-zero-speed
signal must be true.  Then the tool-unclamp signal can be set to true
to release the tool, while watching the tool-clamped and
tool-unclamped signals for feedback.


## Home and Limit switches

### X

X has a negative limit switch (`-LX`, "-X Overtravel") and a combined
Home and positive limit switch (`*DECX`, "X-Axis Zero Return Decel").

### Y

Y has a negative limit switch (`-LY`, "-Y Over Travel"), and a combined
Home and positive limit switch (`*DECY`, "Y Axis Zero Return Decelerate").

### Z

Z has a negative limit switch (`-LZ`, "-Z Over Travel") at the bottom
(very close to the tablle), and a combined Home and positive limit
switch (`*DECZ`, "Z Axis Zero Return Decelerate") at the top.  The +LZ
(CND1-49) switch is not installed

# To bring machine up

Apply pressurized air (what pressure?).

Open the manual air valve at the machine's air intake.

Turn on "+24V"/"0G" power supply (currently hard wired to come on when
the main machine power comes on).

Release all E-stop buttons (1 by operator console, 1 by tool magazine
access door, 1 optional on "handy controller").

Then, using "halcmd -kf":

Power on MAR relay ("sets power-on 1").

Power on the hydraulic pump ("sets hydraulic-lube-pump-on 1").

Activate the tool magazine main solenoid ("sets magazine-run 1").

'sets servo-on-1 1': (Note: this name follows Mazak's servo numbering
scheme which starts at 1.)  This net does a couple of things:

    * Enable pwmgen.00.  This does two things:

        * It turns on all 6 of the Enable outputs on the 7i49.  The
          Enables for channels 0, 1, and 2 turn on external relays 1,
          2, and 3, which asserts NC READY 1, 2, and 3, which turns on
          servo power on the X, Y, and Z servo amps.

        * Enable the analog control voltage from the 7i49 to the X servo.

    * Set SVON1 true, which activates external relay 4, which enables
      servo control of X.

'sets servo-on-2 1':

    * Enable pwmgen.01.  This enables the analog control voltage from
      the 7i49 to the Y servo.

    * Sets SVON2 true, which activates external relay 5, which enables
      servo control of Y.

'sets servo-on-3 1':

    * Enable pwmgen.02.  This enables the analog control voltage from
      the 7i49 to the Z servo.

    * Sets SVON3 true, which activates external relay 6, which enables
      servo control of Z.

    * Assert SA, to activate the ABRK relay, which releases the Z brake.
      Note: the Z servo amp and the Z brake can not be active at the same
      time, or the Z servo will fault with an overcurrent alarm as the
      servo fights against the brake.  If you release the Z brake without
      Z servo control active, the Z axis will slowly drop downwards due
      to gravity.


# Axis brakes

## Z axis brake

sheet 334A:
* solenoid 60 opens Z brake, controlled by line #60

sheet 406B:
* line #60 (Z brake open) activates when ABRK relay closed
* ABRK relay activates when EMS (e-stop ok) and SE (servos enable) are closed

## Table Unclamp (for NC Table, aka rotary 4th axis)

sheet 346A:
* "Table Unclamp" solenoid 20 between #16 and #420 (AC, i think)
* #420 is on TB-41, mentioned on sheet 338 2/5
* #420 is on CA-4, sheet 338 3/5

sheet 371A
* "Table Unclamp" solenoid 20


## MMS Renishaw probe

The probe tool is connected to a control box in the lower right of the
right cabinet door.

https://www.practicalmachinist.com/forum/threads/mazak-probing.173948/
https://www.forum.linuxcnc.org/40-subroutines-and-ngcgui/33034-probe-calibration-renishaw-mp12

https://forum.linuxcnc.org/27-driver-boards/45636-interfacing-with-m8-renishaw-probe
