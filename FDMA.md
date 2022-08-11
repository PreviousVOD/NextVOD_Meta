# FDMA

The STi7105 SoC has two FDMA instances, each FDMA instance has a small CPU called SLIM.
The firmware of the SLIM processor is required to be loaded into the IMEM and DMEM mapped in
the FDMA address space. The firmware files can be acquired from STLinux distribution.

## The CPU


### Firmware
The firmware is an ELF file which has an `e_machine` of `EM_MAX(102)`. However the actual
processor architecture is unknown since the ELF machine ID might be reserved when this architecture
is developed.

### Instruction memory
The instruction memory mapped in the FDMA space is unique, the upper 8 bits of each 32bit word is
always zero. Values written to the MSB will read out as 0.
