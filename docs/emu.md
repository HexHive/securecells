# SecureCells Emulation

The SecureCells architecture can be emulated entirely in QEMU, or with a
combination of QEMU+OpenSBI. In both cases, QEMU performs PTable-based 
access control as described in the paper, with a range-based TLB and 
PTable walks. In both cases, QEMU also implements the `SDSwitch` and
`SDEntry` instructions. The main difference is in the emulation of 
`SCxxx` instructions.

In QEMU, we define two types of machines:

1. `virt_sc` which emulates `SCxxx` instructions
1. `virt` which does not emulate these instructions, raising traps for 
   illegal opcode. Thereafter, OpenSBI handles the traps and emulates
   the instructions.

## Additional CSRs

SecureCells adds two CSRs (Control and Status Registers) as defined in
the paper.

1. `USID` tracks the currently executing SecDiv. The supervisor is assigned
   the special value of `0`. All user SecDivs are assigned non-zero values.
   This field is not writable by userspace, and is only modified by 
   the `SDSwitch` instruction, `ecall` instructions for supervisor/firmware
   calls, traps and by `sret` instructions.
2. `URID` holds the ID of the SecDiv before an explicit (`SDSwitch`, `ecall`)
   or implicit (traps) compartment switch.

To enable the supervisor to transparently handle traps (for example, page faults
originating from paged-out Cells), the `URID` field must be protected across
traps.
We introduce a supervisor-only CSR `UXID` to store the value of the userspace
`URID` CSR on encountering a trap.
The `UXID` register is an artifact of fitting SecureCells onto the RISC-V
architecture, and can be omitted in a generic implementation.
Therefore, this register is not included in the paper.
In a section below, we describe how `UXID` can be omitted in the future.

## `VM_SECCELL` Virtual Memory System

SecureCells introduces a new virtual memory mode to RISC-V, complementing
the existing SV-39 and SV-48 modes. A core enters the `VM_SECCELL` mode by
writing `0xf` to the MODE field in the `satp` register.

In this mode, supervisor and user virtual addresses are translated into
physical addresses by traversing the PTable, using the SecDiv identifier
stored in the `USID` register.

## Modifications in QEMU

The major modifications to QEMU are listed below.

1. Modifications to `get_physical_address` in `target/riscv/cpu_helper.c`
1. Modifications to trap handling in `target/riscv/op_helper.c`
1. Additional file `target/riscv/seccells.c` implements SecureCells-specific
   helpers and instructions.
1. Implementing general instructions for accelerating trap-and-emulate.

## Accelerating `SCxxxx` instructions in OpenSBI

We have modified OpenSBI to emulate `SCxxx` instructions, following RISC-V's
trap-and-emulate philosophy.
Towards this end, we have made three major modifications:

- adding a fast path to handle illegal `SCxxx` instructions, 
- adding machine-mode helpers to help read arguments and find PTable/GTable
addresses, and
- adding machine-mode instructions to access PTable/GTable entries
  and addressses from the MMU.

### Fast Emulation Path

On illegal instruction traps, we have reduced the list of stored/restored
registers to only include callee-saved registers as per the System-V
calling convention.
We also shortcut dispatch handlers to directly jump to SecureCells'
emulation handlers.

Other traps and instruction emulation handlers are delegated to the
existing slow path.

### Accessing Arguments

SecureCells' instructions get their arguments from general purpose registers
and instruction immediate fields.
To emulate these instructions, trap handlers need to read the correct
registers.
The standard approach adopted by OpenSBI is to save all registers into a
context structure, and then index this structure to read out the correct
arguments.
This approach is inefficient, storing and restoring unneccessary register
state on traps.
Our approach accelerates instruction emulation by storing argument registers
to special hardware registers on a trap, and using added instructions to
directly read arguments during emulation.

First, we introduce machine-mode CSRs: `mtirs1`, `mtirs2`, `mtiimm`, 
`mtird` and `mtirdval`.
On an instruction which traps, the corresponding operand registers `rs1`,
`rs2`, `imm` and `rd` (when relevant) are stored to the CSRs.
When the illegal instruction reaches the decode stage, the  operand registers
have already been decoded.
When the trapping instruction reaches the commit stage, the trap is raised, and
the correct argument values are copied from the operand registers to the CSRs.
Since the commit stage is in-order (even on out-of-order cores), the correct
argument values are read.
When `mret` is committed, the value of `mtirdval` is copied into `mtird`.

Within OpenSBI, the instruction emulation handlers can use the instructions
described below.

1. `csrr` to read `mtirs1`, `mtirs2` and `mtiimm`.
2. `csrw` to write `mtirdval`.

The trap handlers can directly read arguments, perform their emulation, and
when relevant, output values directly.

### Interactions with the MMU

SecureCells leverages instructions similar to ARM's `at` isntructions to read
permissions from the MMU, and to quickly compute PTable/GTable addresses
which rely on the metadata cell (see ![PTable layout](/docs/arch.md) for more
details on table layout).
These instructions map directly to common operations for SecureCells'
instructions (described in Figure 4 in the paper).

The firmware can use the instructions described below.

1. `sccck` queries the MMU/TLB for whether an address is valid or not.
1. `scca` computes the address of cell descriptor for cell `C_i`
1. `scpa` computes the address of PTable entry `PT(SD_j, c_i)`
1. `scga` computes the address of GTable entry `GT(SD_j, c_i)`

The first instruction queries the TLB for an address. When the TLB holds an
entry containing the address to a cell, the instruction completes immediately,
otherwise invoking the PTable walker.
The remaining address computation instructions require trivial arithmetic
operations involving parameters of the PTable, derived from the metadata
cell.

## Vision for the future

For a purely compartmentalized architecture, SecureCells can eliminate some
of the heirarchical nature of RISC-V's instruction set.
For example, `ecall` and `sret` instuctions can be eliminated in favour of
`SDSwitch` instructions to and from SecDiv_0, which is reserved for a
privileged compartment, typically the supervisor.

With a non-hierarchical architecture, traps also need not be entirely invisible to
userspace.
We can eliminate the `UXID` register if the supervisor is not responsible for
preserving the `URID` register across traps.

The interactions with the MMU (described in a section above) fit neatly into
a microcode implementation of SecureCells' instructions.
