# SecureCells Architecture: Traps

Traps in SecureCells can originate from features in the base RISC-V 
architecture, or from additions made for SecureCells.
This page documents the relevant trap behaviors.

## Traps due to access control errors

Access control check failures raise traps with SecureCells. These
traps mirror the behavior as with baseline RISC-V architecture.
We describe these traps below, with the corresponding exception
code for `scause` in brackets.

1. Load and store instructions raise "Load access fault" (0x4) and
   "Store/AMO" access fault (0x7), when the executing USID does not
   have the respective permission to the accessed address.
1. Instruction fetch from permissions raise a 
   "Instruction Page Fault" (0xc) when the executing USID does not
   have execute permission for the instruction pointer.
1. Environment calls from userspace (`ecall`) raise the same
   trap number.

Additional traps introduced by SecureCells uses the range `0x18-0x1c`
for exception codes in `scause.

Inst      | Cause                      | `scause`        | `stval`
SCExcl    | Illegal address            | _ILL_ADDR       | `addr` 
SCExcl    | Illegal permissions        | _ILL_PERM       | `(type << 8) | perm` <br> type =  <br> 0 if perm not in RWX <br> 1 if perm is zero <br> 2 if SDcur has insufficient perms 
SCExcl    | Cell description invalid   | _INV_CELL_STATE | 0 if cell is invalid 
SCGrant   | Illegal address            | _ILL_ADDR       | addr 
SCGrant   | Target too high            | _INV_SDID       | SDtgt 
SCGrant   | Illegal permissions        | _ILL_PERM       | `(type << 8) | perm` <br> type =  <br> 0 if perm not in RWX <br> 1 if perm is zero <br> 2 if SDcur has insufficient perms 
SCGrant   | Cell description invalid   | _INV_CELL_STATE | 0 if cell is invalid 
SCRecv    | Illegal address            | _ILL_ADDR       | addr 
SCRecv    | Cell description invalid   | _INV_CELL_STATE | 0 if cell is invalid 
SCRecv    | Illegal permissions        | _ILL_PERM       | `(type << 8) | perm` <br> type =  <br> 0 if perm not in RWX <br> 1 if perm is zero <br> 3 if requested perms not subset of granted perms 
SCRecv    | Invalid SD                 | _INV_SDID       | `(type << 32) | SD` <br> type = <br> 0 if SD source too high, SD=SDsrc <br> 1 if grant SD != SDcur, SD=SDcur 
SCProtect | Illegal address            | _ILL_ADDR       | addr 
SCProtect | Illegal permissions        | _ILL_PERM       | `(type << 8) | perm` <br> type =  <br> 0 if perm not in RWX <br> 2 if SDcur has insufficient perms 
SCProtect | Cell description invalid   | _INV_CELL_STATE | 0 if cell is invalid 
SCTfer    | Illegal address            | _ILL_ADDR       | addr 
SCTfer    | Target too high            | _INV_SDID       | SDtgt 
SCTfer    | Illegal permissions        | _ILL_PERM       | `(type << 8) | perm` <br> type =  <br> 0 if perm not in RWX <br> 1 if perm is zero <br> 2 if SDcur has insufficient perms 
SCTfer    | Cell description invalid   | _INV_CELL_STATE | 0 if cell is invalid 
SCInval   | Illegal address            | _ILL_ADDR       | addr 
SCInval   | Cell description invalid   | _INV_CELL_STATE | 0 if cell is already invalid <br> 2 if other SD has non-zero perms <br> 3 outstanding grants for cell 
SCReval   | Illegal address            | _ILL_ADDR       | addr 
SCReval   | Cell description invalid   | _INV_CELL_STATE | 1 if cell is already valid 
SCReval   | Illegal permissions        | _ILL_PERM       | `(type << 8) | perm` <br> type =  <br> 0 if perm not in RWX <br> 1 if perm is zero 
SDSwitch  | Illegal address            | _ILL_ADDR       | addr 
SDSwitch  | Illegal permissions        | _ILL_PERM       | `(type << 8) | perm` <br> type= <br> 4 for illegal sdswitch 
SDSwitch  | Target too high            | _INV_SDID       | SDtgt 
SDSwitch  | Illegal target instruction | _ILL_TGT        | Target addr 

Numeric values for `scause` shown below.

`scause` cause        | `scause` Exception Code
_ILL_ADDR             | 0x18
_ILL_PERM             | 0x19
_INV_SDID             | 0x1a
_INV_CELL_STATE       | 0x1b
_ILL_TGT              | 0x1c
