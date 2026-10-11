# Opening the Mini Ultra 3 case

No published teardown of the Mini Ultra 3 exists as of October 2026. This is a generic procedure
for its construction: a 25 x 25 x 22 mm cube, aluminium alloy sleeve with a PC-ABS end cap,
integrated USB-A plug. Expect the plastic cap to be press-fit or glued into the metal sleeve
with the PCB attached to the cap and the USB-A plug.

## Before you start

- Opening voids the warranty. Record the serial and buy a second unit if you want a spare.
- Take the `getversion.cgi` capture first (done) and photograph the outside, every label, and
  the USB plug orientation.
- Work on an anti-static mat or at least touch a grounded metal object first. The MFi chip
  and the radio are the parts you cannot replace.
- Tools: plastic spudger or guitar pick, thin flat blade, hair dryer or heat gun on low,
  isopropyl alcohol, tweezers, a phone camera with macro or a loupe, good light.

## Procedure

1. **Find the seam.** Look where the plastic cap meets the metal sleeve on the USB plug end,
   and check the opposite face for a second cap or a label. Check under any label for screws
   or a pry slot. Carlinkit's larger boxes hide screws under rubber feet, so probe soft spots.
2. **Warm it.** 30 to 60 seconds of hair dryer heat on the seam softens adhesive. Do not exceed
   what you can touch comfortably; the radio module and flash are near the shell.
3. **Pry, do not lever against the board.** Work a pick into the seam and walk it around. The
   PCB is likely glued or taped to the cap; if the cap comes off with the board attached,
   stop and photograph before separating anything else.
4. **If it is a USB-A plug on a flex or pins:** the plug is probably soldered to the PCB and the
   board slides out of the sleeve plug-first. Push gently from the far end with a wooden stick.
5. **Antenna.** Small dongles often use a PCB trace antenna or a sprung contact to the metal
   shell. Note how it couples before you pull the board clear; it matters for reassembly.
6. **Photograph both sides** at the highest resolution you can, square on and lit from an angle
   so chip markings read. Markings are tiny and often laser-etched; a loupe helps.

## What to read and record

| Part | Where | What we need |
|---|---|---|
| SoC | largest chip | full marking, expect `X1600E` or similar, or `AK3918`, or `V821B` |
| Radio module | shielded can or small module near the antenna | marking, expect `AIC8800D80` or `RTL8733BS` |
| SPI flash | 8-pin SOIC, marked `25Q128`, `XT25F128`, `W25Q128` or similar | exact part number gives size and the pin 5 and 6 positions for USB boot mode |
| MFi chip | small 10-pin or so package, often near the SoC | marking, and a photo good enough to compare with the dummy packages reported on some units |
| RAM | if a separate chip, marking; the X1600EN has RAM in package so there may be none | |
| Test pads | labelled dots or an unpopulated header | `TX`, `RX`, `GND`, `BOOT`, `RST`; any 4-pad row is a UART candidate |
| PCB silkscreen | board revision string and date | ties the board to reports in catplay and LIVI issues |

Put the photos in `re/captures/` with the date and side in the filename. They are the input to
the next step and to the report we file on catplay-firmware.

## Reassembly

Clean old adhesive with isopropyl alcohol. A thin bead of B7000 or similar flexible adhesive
on the cap lip holds without making the next opening impossible. Check the antenna contact
before closing. Test on the car before gluing.
