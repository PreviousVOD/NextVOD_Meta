# FDMA

The STi7105 SoC has 2 FDMA instances, each FDMA instance has a small CPU called SLIM.
According to the specs, each FDMA supports 16 concurrent transfers.
The firmware of the SLIM processor is required to be loaded into the IMEM and DMEM mapped in
the FDMA address space. The firmware files can be acquired from STLinux distribution.

## The CPU
The `SLIM` CPU is also noted as `STxP70`, which is a 32-bit RISC processor used in some STMicroelectronic products.

> Apart from this FDMA, it is also found in VL53L5Cx ToF ranging sensors[[1](#References)].

### Firmware
The firmware is an ELF file which has an `e_machine` of `EM_MAX(102)`, this is reserved 
during the time this processor is developed.

<details>
    <summary>ELF headers of the firmware</summary>
    <pre>
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           MAX Processor
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          52 (bytes into file)
  Start of section headers:          116 (bytes into file)
  Flags:                             0x2
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         2
  Size of section headers:           40 (bytes)
  Number of section headers:         4
  Section header string table index: 1

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .shstrtab         STRTAB          00000000 000114 000017 00      0   0  0
  [ 2] .data             PROGBITS        fe228000 0036c0 002000 00   A  0   0 32
  [ 3] .text             PROGBITS        fe22c000 000180 00353c 00  AX  0   0 32
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), p (processor specific)

There are no section groups in this file.

Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  LOAD           0x000180 0xfe22c000 0xfe22c000 0x0353c 0x0353c R E 0x20
  LOAD           0x0036c0 0xfe228000 0xfe228000 0x02000 0x02000 R   0x20

 Section to Segment mapping:
  Segment Sections...
   00     .text 
   01     .data 

There is no dynamic section in this file.

There are no relocations in this file.

The decoding of unwind sections for machine type MAX Processor is not currently supported.

No version information found in this file.
    </pre>
</details>

## Memories
The SLIM CPU has its dedicated instruction memory(IMEM) and data memory(DMEM).

### Instruction Memory(IMEM)
The instruction memory mapped in the FDMA space is unique, the upper 8 bits of each 32bit word is
always zero. Values written to the MSB will read out as 0, resulting the actual IMEM size is 12kB.

### Data Memory(DMEM)
The 8kB data memory is mapped at 0x8000, which can be accessed by both SLIM and the SH4-300 core.
The data memory also serves as "firmware registers" to the FDMA channel descriptors.

## Address spaces
The address space of the FDMA can be categorized as follows:

SLIM control registers
| Offset | Size | Name | Remarks |
|--|--|--|--|
| 0x0000 | 4 | ID | The ID of the SLIM processor, 0 for STi7105 |
| 0x0004 | 4 | VER | The version of the SLIM processor, 0 for STi7105 |
| 0x0008 | 4 | EN | Enable control of the SLIM processor |
| 0x000C | 4 | CLK_GATE | Clock gate of the SLIM processor |

Memories
| Offset | Size | Name | Remarks |
|--|--|--|--|
| 0x8000 | 0x2000 | DMEM | Data memory and firmware registers |
| 0xC000 | 0x4000 | IMEM | Instruction memory, see above. |

Peripheral control regisers/Mailboxes
| Offset | Size | Name | Remarks |
|--|--|--|--|
| 0xBF88 | 4 | SYNC | STBus sync register |
| 0xBFC0 | 4 | CMD_STA | Command mailbox |
| 0xBFC4 | 4 | CMD_SET | See above, set bits |
| 0xBFC8 | 4 | CMD_CLR | See above, clear bits |
| 0xBFCC | 4 | CMD_MASK | See above, mask bits |
| 0xBFD0 | 4 | INT_STA | Interrupt mailbox |
| 0xBFD4 | 4 | INT_SET | See above, set bits |
| 0xBFD8 | 4 | INT_CLR | See above, clear bits |
| 0xBFDC | 4 | INT_MASK | See above, mask bits |


Firmware registers (shared memory)
> Address relative to DMEM(BASE + 0x8000)

| Offset | Size | Name | Remarks |
|--|--|--|--|
| 0x0000 | 4 | REVID | firmware revision, major: bit [23:16], minor: bit[15:8]. E.g. 0x00020300 v2.3 |
| 0x1140 + 4 * n | 4 | CMD_STAT[n] ||
| 0x1180 + 4 * n | 4 | REQ_CTL[n] ||
| 0x1580 + 64 * n | 64 | CHANNEL_DESC[n] | Channel descriptors, see below |

Channel descriptor
| Offset | Size | Name | Remarks |
|--|--|--|--|
| 0x0000 | 4 | PTR ||
| 0x0008 | 4 | CNT ||
| 0x000C | 4 | SADDR ||
| 0x0010 | 4 | DADDR ||

## References
\[1\]: https://the6p4c.github.io/2022/06/13/stxp70-sthorm-p2012.html
