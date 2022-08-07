# STi7105 hardware boot procedure

## What we have learned
* STi7105 starts execution at address `0xA000_0000`, in little endian mode.
* Boot from SPI flash: the flash is mapped to EMI address space at `0xA000_0000`, determined by hardware pins.
* In 29-bit mode, these ranges are the same EMI address space as the processor simply discarded the higher 3 address bits.
* The EMI can only be accessed in 32 bits, so XIP cannot be achieved.
* MMU is forced active in 32bit(SE) mode, TLB or PMB has to be configured before mode switching
* Most address space which PMB can configure is in P2/P3, which cannot be accessed when in user mode (U0)
