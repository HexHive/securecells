---
title: Architecture Description
---

Other stuff to be added.

## PTable-GTable layout

SecureCells replaces traditional page tables for storing virtual memory
translations and permissions with a unified PTable-GTable.
These tables describe the memory **Cell**s, permissions for each **SecDiv**
to these cells, and hold outstanding permissions between grant and receive.

<div>
<embed src="/assets/svg/ptable_layout.svg" width="100%" height=700px>
</div>

The unified PTable-GTable layout is shown above. 
This table contains three sections:

1. Sorted cell descriptor list, including metadata cell
1. PTable matrix
1. GTable matrix

The metadata cell describes the table with four parameters.
The implementation assumes a cache line size of 64B, but can be amended for
any other cache line size.
Two parameters relate to the sizing of the tables, limiting the maximum
number of cells and SecDivs accomodated by the table.
If either limit is exceeded, the table must be resized to accomodate extra
cells and SecDivs.

1. N: the number of cells
1. M: the number of SecDivs
1. T: for table sizing, allowing up to (64T - 1)[^1] cells
1. R: for table sizing, allowing up to R cells


### Cell descriptors

Each cell has a descriptor in the sorted list at the beginning of the table.
The 128-bit descriptor holds the following fields.

1. 36-bit VA base, holding the virtual page number for the base of the cell
2. 36-bit VA bound, holding the virtual page number for the last page of the 
   cell
3. 44-bit PA base, holding the physical frame number for the base of the cell
4. 1-bit tracking the cell's current validity
5. 11-bit reserved region

The virtual address space size matches Intel's 4-level paging and RISC-V's 
SV-48 mode.
The physical address space size matches RISC-V's 56-bit physical address space.
Cells are aligned to 4kiB pages, mapping the 48-bit virtual addresses to 56-bit
physical addresses.
Larger virtual and physical address spaces can be supported, with 
correspondingly larger cell descriptors.
Compression of cell descriptors (incl. naturally aligned powers-of-2 spaces)
are possible, but eschewed in favour of flexibility.

### PTable permissions

Each PTable entry holds three permissions, corresponding to read, write 
and execute.
Each entry is one byte large, with rwx permissions at bits 1, 2 and 3
respectively.
Bits 0, 4-7 are reserved for future use.
The reserved bits can be later purposed for tracking dirty and accessed cells,
similarly to page table entries.

The table reserves T cache lines[^1] for each SecDiv's permissions.
Permissions for SecDivs are held in linear order of their SDID.
To keep the PTable size manageable, SecDivs must be allocated SDIDs linearly.

Clustering permissions for each SecDiv aims to improve spatial locality for
MMU accesses to the PTable.
Since the same cache line holds permissions for nearby cells, reading 
permissions for one cell brings in permissions for nearby cells into the cache.

> The PTable starts at offset `(16 * 64T)` bytes

> The offset of the permission byte PT(SD_cur, cell_i) is `(64T * SD_cur) + i` from the beginning of the PTable

### GTable permissions

Each GTable entry is 4bytes, holding a target 29-bit SDID and 3-bit 
permissions.

> The GTable starts at offset `(16 * 64T) + (R * 64T)` bytes

> The offsest of grant entry GT(SD_cur, cell_i) is `4 * ((64T * SD_cur) + i)` from the beginning of the GTable

[^1]: Related to the cache line size of 64B
