# STi7105 hardware boot procedure

## What we have learned
* STi7105 starts execution at address `0x0000_0000`, in little endian mode.
* Boot from SPI flash: the flash is mapped to EMI address space at `0x0000_0000`, determined by hardware pins.
* The initialization process configures a bunch of registers, incl. clock and peripherals.
* SH-4 CPU has two address modes, either 29bits or 32bits. The memory mapping for both modes is different.
* As on-chip RAM is never found in documentation, we can assume that LMI DDR2 SDRAM is initialized before execution starts
* ...which probably initialized by hardware or BootROM.
* The LMI is mapped at `0x0C00_0000` in 29bit mode, `0x4000_0000` in 32bit mode.
* The Github repo contains board support for this specific board(GB620)
* Red LED is `P0_4`, Green LED is `P0_5`

## TODO
* Proper lds for bare metal testing
* Proper assembly startup for bare metal testing

## Supplementary files
[Check Here](https://cloud.imi.moe/s/AFzsnEKNfxQYjjC)
