# LiquorBot

# Parts

## Printing
Parts are named using Voron-style conventions with [a] prefixes to denote accent color parts and quantity needed at the suffix. You'll need all parts from the main STLs directory plus the pump labels & octopus board mount from the single_color _or_ multi_color directories.

There are two options for the dispenser & drip tray. The basic version is what it says on the tin, a mount arm to attach the hoses to the leg and a simple tray to catch the drips. The led_sensor versions have provisions for a ring of 16 LEDs in the dispenser and a cup sensor switch in the drip tray (to prevent pouring a drink without a cup present).

All parts were printed at 0.20 layer height, 15% infill except for the pump pieces (see Pumps section below)

## Top plate and shelf
It's possible to make the top plate and shelf by hand using brackets as templates but it's _tedious_, I've done it a couple times on the different iterations of LiquorBot. That's why I've updated the CAD design from just laying out the parts and holes on the board to a set of SVGs suitable to be cut with a CNC.

Holes for the shelf and top plate leg brackets can be optionally counter-sunk for a smoother surface (hardware list assumes counter sunk screws for the shelf).

Holes for mounting the Raspberry Pi & screen will need to be hand-drilled.

## Hardware and parts

| Item                      | Qty | Location                            |
|---------------------------|-----|-------------------------------------|
| M3 nut                    | 18  | Pump bracket & power plug           |
| M3S nut                   | 10  | Leg, dispenser & cup brackets       |
| M3-8 countersunk          | 20  | Plate/shelf to leg brackets         |
| M3-16                     | 16  | Pump bracket                        |
| M3-12                     | 10  | Leg, dispenser & cup brackets       |
| M3 heat insert            | 4   | Controller board mount              |
| M4-12 countersunk	        | 6   | Pump tags/hose clips                |
| M4-16 countersunk	        | 2   | Controller board mount/hose clips   |
| M2-8 countersunk          | 2   | Cup sensor & LED dispenser head     |
| M2x4x3.5 heat insert      | 2   | Cup sensor & LED dispenser head     |
| M2-10 self-tapping        | 4   | Cup sensor switches                 |
| M4 washers                | 48  | Pump rotors                         |
| M4 nuts                   | 56  | Pump rotors & housing               |
| M4x20                     | 24  | Pump rotors                         |
| M4x25                     | 32  | Pump housing                        |

| Item                                     | Qty | Source |
|------------------------------------------|-----|--------|
| 18"/460mm 3/4"/19mm aluminum tubing      | 4   |  |
| 5"/130mm 3/4"/19mm aluminum tubing       | 2   |  |
| furniture leg caps                       | 6   | https://amazon.com/uxcell-Furniture-Protector-Prevent-Scratches/dp/B07PGS84Y6 |
| micro switches                           | 2   | https://www.amazon.com/gp/product/B07FJ77HNV |
| Big Tree Tech Octopus board              | 1   | https://biqu.equipment/products/bigtreetech-octopus-v1-1?variant=39812758765666 |
| 16 LED neopixel ring light               | 1   | https://amazon.com/dp/B0B2D5QXG5 |
| 3 & 4 pin JST connectors                 | 1ea | https://aliexpress.us/item/2251832574216984.html	|
| 2.5mm power socket                       | 1   | https://www.amazon.com/gp/product/B078YNW3JZ |
| 24V power supply                         | 1   | https://amazon.com/gp/product/B01GC6VS8I |
| Nema17 Stepper motors, 24mm shaft        | 8   | https://www.amazon.com/dp/B088B756MY/?coliid=I3CAV1IQ1ES0T8 |
| 13x4x5 (ODBoreW) bearings for pump rotor | 48  | https://www.amazon.com/dp/B094D6C3ZN |
| 6x4mm silicone tubing                    | 16m | https://www.amazon.com/dp/B08PTXZ51Q |

Raspberry pi + touchscreen and case, I used https://smarticase.com/collections/all-products/products/smartipi-touch-2

Tubing lengths can be adjusted for shorter/taller bottles

## Pumps
You'll also need to print & build 8 sets of pumps and 2 brackets from Ryan Dibisch's excellent nema17-based peristalic pump model: https://www.thingiverse.com/thing:1134817 There is some variability of inner diameter and squishiness of silicone hoses, so you may need to print thicker or thinner pressure plates from the pump_mods folder. Plates are labeled as how many millimeters thicker or thinner the space for the hose is relative to the stock design.

# Assembly

TODO: pictures are worth 1000 words

# Configure
LiquorBot is effectively a 3d printer that extrudes drink ingredients and configuration is the same as any other [Klipper](https://www.klipper3d.org/)-based 3d printer. [KIAUH](https://github.com/th33xitus/kiauh) is an easy path to get Klipper installed and running. Follow Big Tree Tech's [instructions](https://github.com/bigtreetech/BIGTREETECH-OCTOPUS-V1.0/tree/master/Firmware/Klipper) for installing klipper firmware on the Octopus board and connecting it to the pi.

Once Klipper is installed, be sure to change the `serial` parameter in [printer.cfg](config/printer.cfg) to match your board (see BTT instructions above).

# Tuning
Using the PUMP macro with a relatively low SPEED parameter (about 100) for each pump, start with a relatively loose pressure plate and switch to increasingly tighter ones until you find one that will both pump water and not leak/siphon/drip when the pump is off. Once you have the rigth pressure plate, test with increasing SPEED parameters until you find the fastest speed that will reliably run without stalling the stepper. On my machine, most pumps run at about 250, but some need to be backed off to 210. Set the default speed for each pump in the DISPENSE_NO_CHECK macro.

The PUMP macro is also where volumetric units are converted into mm of rotation. You'll want to adjust the multiplier in the MANUAL_STEPPER MOVE parameter so that the volume dispensed matches the volume requested. My machine needs 1862mm for 1oz or 63mm per milliliter.

# API
Moonraker provides a HTTP interface to run gcode, see "Run a gcode" on https://moonraker.readthedocs.io/en/latest/web_api/ for details. 

LiquorBot's main API macros are:
 * DISPENSE - takes a volume for each pump (p1 through p8), waits until the cup sensor reports a cup present and then calls DISPENSE_NO_CHECK
 * DISPENSE_NO_CHECK - takes same parameters as DISPENSE but doens't check if a cup is present before calling PUMP macro for each pump. Use this if you're not using the cup sensor. SPEED parameter can be used to override the default speed for all pumps
 * PRIME - pumps slightly more than the volume contained in the tubing for each pump. Call this when connecting all bottle for the first time
 * UNLOAD - opposite of PRIME, reverses the pumps to put any product still in the hoses back into the bottles
 * CLEAN - runs several oz through each pump, useful to flush the hoses with water and/or vodka for cleaning

