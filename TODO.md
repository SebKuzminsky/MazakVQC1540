finish wiring (we currently do not have a list of which wires are
present and which are still to do).

make a wiring diagram in kicad (started in bon-wiring/).

figure out ALM and SC on TRS-50B - pull up to +24V and read as inputs?

figure out spindle drive

figure out limit & home switches and use them in LinuxCNC to stop
machine before hitting hard stops.

figure out estop.  prelimiary setup in place, need to also support
pressing estop from tui.

tune servo amps with pots?

tune servo position pid loops in linuxcnc, see
https://gnipsel.com/linuxcnc/tuning/index.html

make finished Mesa controller and electronics mounting plate with
MR-20 and MR-50 connectors

figure out and hook up Renishaw probe tool (MMS)

figure out tool change procedyre.  Partly done, T# work, M06 remains.

integrate tool length probe arm into linuxcnc
