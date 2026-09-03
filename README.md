# Virtual Memory, Paging, Page Tables and TLB

## 1. Program, Process and Thread

### Program

A **program** is a passive file containing instructions and data. It is stored on secondary memory, such as an SSD or hard disk.

Examples:

- A compiled C executable
- A web browser application
- A media player executable

### Process

A **process** is a running instance of a program.

A process contains:

- Program instructions
- Global and static data
- Heap
- Stack
- CPU registers
- Program counter
- Open files
- Operating-system resources
- Virtual address space
- Page-table hierarchy

The same program can be executed multiple times, creating multiple processes:

```text
Program executable
      ├── Process A
      ├── Process B
      └── Process C
```

Although these processes execute the same program, they normally have separate virtual address spaces and separate page tables.

### Thread

A process may contain one or more threads.

Threads belonging to the same process normally share:

- Program instructions
- Global data
- Heap
- Virtual address space
- Page tables

Each thread has its own:

- Program counter
- CPU registers
- Stack

Therefore:

> Different processes normally have different virtual address spaces, while threads of the same process share one virtual address space.

---

## 2. Physical Memory

**Physical memory** refers to the actual RAM installed in the system.

For example:

```text
Installed physical memory = 2 GiB
```

Since:

```text
2 GiB = 2³¹ bytes
```

a **31-bit physical address** is sufficient to identify every byte in 2 GiB of physical memory.

Physical addresses range from:

```text
0 to 2³¹ − 1
```

The physical address is normally generated after virtual-to-physical address translation.

---

## 3. Virtual Memory

**Virtual memory** is a memory-management mechanism implemented through cooperation between:

- CPU
- Memory Management Unit
- Operating system
- Page tables
- TLB
- Physical RAM
- Secondary storage

It allows processes to use virtual addresses instead of directly accessing physical addresses.

Virtual memory is not a separate physical memory device. It is an abstraction created using address translation and memory-management mechanisms.

---

## 4. Virtual Address Space

Each process normally receives its own **virtual address space**.

For example:

```text
Process A: Virtual addresses 0 to 32 GiB − 1
Process B: Virtual addresses 0 to 32 GiB − 1
Process C: Virtual addresses 0 to 32 GiB − 1
```

The same virtual address can map to different physical addresses in different processes:

| Process | Virtual address | Physical address |
|---|---:|---:|
| Process A | `0x1000` | `0xA000` |
| Process B | `0x1000` | `0xF000` |

This is possible because each process has its own page-table hierarchy.

### Correct terminology

It is common to informally say:

> Each process has its own virtual memory.

More precisely:

> The system provides the virtual-memory mechanism, while each process has its own virtual address space and page-table mappings.

A process does not own a separate physical virtual-memory device.

---

## 5. Can Virtual Memory Be Larger Than Physical Memory?

Yes. The virtual address space of a process can be larger than the installed physical memory.

For example:

```text
Virtual address space per process = 32 GiB
Installed physical memory         = 2 GiB
```

This does not mean that the process occupies 32 GiB of RAM.

The 32 GiB value represents the range of virtual addresses available to the process. Only the pages currently required by the process need to be present in physical memory.

Some virtual pages may be:

- Present in physical RAM
- Stored on secondary memory
- Not yet allocated
- Shared with another process
- Mapped to files
- Invalid or unused

Therefore:

```text
Virtual address-space size ≠ Amount of physical RAM used
```

---

## 6. Process Isolation

One process normally cannot access another process’s private memory.

Suppose Process A and Process B both generate virtual address `0x1000`.

```text
Process A: VA 0x1000 → Process A page table → PA 0xA000
Process B: VA 0x1000 → Process B page table → PA 0xF000
```

When Process A is running, the MMU uses Process A’s page table. It does not use Process B’s page table.

Therefore, Process A cannot access Process B’s private memory simply by generating the same virtual address.

This isolation is implemented using:

- Separate page tables
- Page-table permission bits
- User and kernel privilege levels
- MMU protection checks

---

## 7. Shared Physical Memory

Although processes normally have separate address spaces, they can intentionally share physical memory.

```text
Process A virtual page 5 ─┐
                          ├── Physical frame 100
Process B virtual page 9 ─┘
```

Process A and Process B may use different virtual addresses while accessing the same physical frame.

Shared mappings are used for:

- Shared memory
- Shared libraries
- Inter-process communication
- Memory-mapped files
- Copy-on-write

Therefore:

> Processes have separate virtual address spaces, but selected virtual pages can intentionally map to the same physical frames.

---

## 8. Why Virtual Memory Is Needed

### Process isolation

Each process receives an independent address space, preventing it from directly corrupting another process’s private memory.

### Memory protection

Pages can be marked as:

- Read-only
- Read/write
- Executable
- Non-executable
- User accessible
- Kernel only

### Easier programming

Programs see a simple and continuous virtual address space even when the corresponding physical frames are scattered throughout RAM.

### Execution of large programs

Only the currently required pages must be present in physical memory. Other pages may remain on secondary storage until required.

### Efficient physical-memory allocation

A process does not require one large contiguous physical-memory region.

### Memory sharing

Selected physical pages can be shared between processes without sharing their complete address spaces.

---

# Contiguous Memory Allocation

## 9. Static Partitioning

In **static partitioning**, physical memory is divided into fixed-size partitions before processes are loaded.

Example:

```text
Physical memory
├── Operating-system region
├── Partition 1: 256 MiB
├── Partition 2: 256 MiB
├── Partition 3: 512 MiB
└── Partition 4: 1 GiB
```

Each partition can normally hold one process.

### Advantages

- Simple to implement
- Low allocation overhead
- Predictable partition boundaries

### Disadvantages

- Number of simultaneous processes is limited.
- A process larger than every partition cannot run.
- Unused space inside a partition is wasted.
- It causes internal fragmentation.

### Internal fragmentation example

Suppose a 100 MiB process is loaded into a 256 MiB partition:

```text
Wasted space = 256 MiB − 100 MiB
             = 156 MiB
```

The wasted space is inside the allocated partition, so it is called **internal fragmentation**.

---

## 10. Dynamic Partitioning

In **dynamic partitioning**, partitions are created according to the sizes of processes when they are loaded.

Example:

```text
Process A requires 100 MiB → Allocate 100 MiB
Process B requires 250 MiB → Allocate 250 MiB
Process C requires 80 MiB  → Allocate 80 MiB
```

Common allocation strategies include:

- First fit
- Best fit
- Worst fit
- Next fit

### Advantages

- Partition size closely matches process size.
- Less internal fragmentation occurs.
- Physical memory is initially used more efficiently.

### Disadvantages

- Free memory becomes divided into separated holes.
- It causes external fragmentation.
- Allocation and deallocation are more complex.
- Memory compaction may be required.

### External fragmentation example

Suppose physical memory contains:

```text
100 MiB free hole
60 MiB free hole
80 MiB free hole
```

The total available memory is:

```text
100 MiB + 60 MiB + 80 MiB = 240 MiB
```

However, a 200 MiB process cannot be allocated because there is no single contiguous 200 MiB region.

This is called **external fragmentation**.

### Compaction

The OS can move processes so that scattered free regions combine into one large region.

However, compaction:

- Takes time
- Requires process relocation
- Produces significant memory traffic
- Can interrupt normal execution

Paging removes the requirement that a process occupy one contiguous physical-memory region.

---

# Paging

## 11. What Is Paging?

Paging divides virtual and physical memory into equal-sized fixed blocks.

- A virtual-memory block is called a **page**.
- A physical-memory block is called a **frame** or **page frame**.
- Page size and frame size are equal.

Example:

```text
Page size = Frame size = 2 KiB
```

Consecutive virtual pages do not have to occupy consecutive physical frames:

```text
Virtual page 0 → Physical frame 100
Virtual page 1 → Physical frame 25
Virtual page 2 → Physical frame 700
Virtual page 3 → Physical frame 41
```

To the process, the virtual pages appear continuous. In physical memory, the frames may be scattered.

---

## 12. Advantages of Paging

Paging provides:

- Non-contiguous physical-memory allocation
- Elimination of external fragmentation
- Simpler physical-memory allocation
- Per-page protection
- Page sharing
- Demand paging
- Efficient process isolation

Because every physical frame has the same size, any virtual page can be placed in any available frame.

### Fragmentation in paging

Paging eliminates external fragmentation but may cause some internal fragmentation.

For example, if a process needs 1 byte more than a complete 2 KiB page, another 2 KiB page must be allocated.

The unused portion of the final page is internal fragmentation.

---

# Address Translation

## 13. Example System

Assume:

```text
Number of processes              = 10
Virtual address space per process = 32 GiB
Physical memory                   = 2 GiB
Page size                         = 2 KiB
Page-table entry size             = 4 bytes
```

---

## 14. Virtual Address Size

The virtual address space is:

```text
32 GiB = 2³⁵ bytes
```

Therefore, the virtual address is **35 bits**.

---

## 15. Physical Address Size

The physical memory is:

```text
2 GiB = 2³¹ bytes
```

Therefore, the physical address is **31 bits**.

---

## 16. Page Offset Size

The page size is:

```text
2 KiB = 2¹¹ bytes
```

Therefore, the page offset requires **11 bits**.

---

## 17. Virtual Address Format

The virtual address has 35 bits, of which 11 bits form the offset:

```text
VPN bits = 35 − 11
         = 24 bits
```

Therefore:

```text
35-bit virtual address
┌────────────────────────────┬─────────────┐
│ 24-bit Virtual Page Number │11-bit offset│
└────────────────────────────┴─────────────┘
```

Number of virtual pages per process:

```text
Number of virtual pages = 32 GiB / 2 KiB
                        = 2³⁵ / 2¹¹
                        = 2²⁴
                        = 16,777,216 pages
```

---

## 18. Physical Address Format

The physical address has 31 bits, of which 11 bits form the offset:

```text
PFN bits = 31 − 11
         = 20 bits
```

Therefore:

```text
31-bit physical address
┌─────────────────────────────┬─────────────┐
│20-bit Physical Frame Number │11-bit offset│
└─────────────────────────────┴─────────────┘
```

Number of physical frames:

```text
Number of frames = 2 GiB / 2 KiB
                 = 2³¹ / 2¹¹
                 = 2²⁰
                 = 1,048,576 frames
```

All processes, the operating system and the page tables share these physical frames.

---

## 19. Basic Translation

During address translation:

```text
Virtual Page Number | Offset
          ↓
Physical Frame No.  | Same offset
```

The VPN is replaced by the PFN.

The offset remains unchanged because the page size and frame size are equal.

---

# Memory Management Unit

## 20. What Is the MMU?

The **Memory Management Unit (MMU)** is a hardware unit that translates CPU-generated virtual addresses into physical addresses.

The MMU also performs:

- Permission checking
- User/kernel access checking
- Read/write checking
- Execute-permission checking
- TLB lookup
- Page-table walking
- Memory-protection fault generation

The operating system creates and maintains the page tables. The MMU uses them to perform translation.

### Basic MMU operation

```text
CPU generates virtual address
              ↓
MMU separates VPN and offset
              ↓
MMU searches for VPN translation
              ↓
VPN is translated into PFN
              ↓
PFN is combined with the same offset
              ↓
Physical address is generated
```

---

# Page Tables

## 21. What Is a Page Table?

A **page table** stores mappings from virtual page numbers to physical frame numbers.

Example:

| Virtual page number | Physical frame number |
|---:|---:|
| 0 | 500 |
| 1 | 110 |
| 2 | Not present |
| 3 | 800 |

Each process normally has its own page-table hierarchy.

A process has many page-table entries, generally one entry for each mapped virtual page.

During a context switch, the OS changes a special register containing the root address of the new process’s page table.

Threads of the same process normally share the page tables because they share the same virtual address space.

---

## 22. Page-Table Entry

A **Page-Table Entry (PTE)** commonly contains:

- Physical frame number
- Present or valid bit
- Read/write permission
- User/kernel permission
- Execute-disable permission
- Accessed or reference bit
- Dirty or modified bit
- Cache-control bits
- Other architecture-specific bits

### Present bit

```text
Present = 1 → Page is mapped to a physical frame
Present = 0 → Page is not present or mapping is invalid
```

### Dirty bit

The dirty bit indicates that the page has been modified after being loaded into physical memory.

If a dirty page is evicted, its updated contents normally must be written to secondary storage.

### Accessed bit

The accessed bit indicates that the page has recently been used.

The operating system can use it when selecting a page for replacement.

---

# Page Faults and Demand Paging

## 23. What Happens When a Page Is Not Present?

The correct situation is:

> The page-table entry for the requested virtual page indicates that the page is not currently present in physical memory.

The following sequence occurs:

1. The CPU generates a virtual address.
2. The MMU searches the TLB.
3. If the TLB misses, the page table is examined.
4. The required PTE has `Present = 0`.
5. The MMU raises a page-fault exception.
6. Control transfers from the process to the operating system.
7. The OS checks whether the virtual address is valid.
8. If it is invalid, the OS reports an access violation.
9. If it is valid but not present, the OS locates the page on secondary storage.
10. The OS finds a free physical frame.
11. If no frame is free, another page is selected for eviction.
12. If the selected page is dirty, it is written to secondary storage.
13. The required page is loaded into the selected physical frame.
14. The OS updates the page-table entry.
15. The relevant TLB entry is inserted or updated.
16. The interrupted instruction is restarted.

```text
Virtual address
      ↓
PTE says “not present”
      ↓
Page-fault exception
      ↓
OS checks whether address is valid
      ↓
Find free frame or evict another page
      ↓
Load requested page from storage
      ↓
Update PTE and TLB
      ↓
Restart instruction
```

This mechanism is called **demand paging** when a page is loaded only when it is first accessed.

---

## 24. Page Fault Does Not Always Mean an Error

| Situation | Operating-system response |
|---|---|
| Valid page stored on disk | Load it into RAM |
| First access to a valid unallocated page | Allocate a new page |
| Copy-on-write page is modified | Create a private copy |
| Address is outside valid process memory | Report an access violation |
| Write attempted on a read-only page | Report a protection fault |

A valid page fault is part of normal virtual-memory operation.

An invalid access may terminate the process, for example with a segmentation fault.

---

## 25. Page Replacement

If no free physical frame is available, the operating system must select an existing page for eviction.

Possible page-replacement algorithms include:

- First In First Out
- Least Recently Used
- Clock algorithm
- Second-chance algorithm

If the selected page is clean, it can normally be discarded.

If the selected page is dirty, its updated contents must first be written to secondary storage.

### Thrashing

If a system has too little physical memory for its active processes, pages may be moved continuously between RAM and storage.

This condition is called **thrashing**.

During thrashing, the system spends more time servicing page faults than executing useful instructions.

---

# Flat Page Table

## 26. Flat Page-Table Size

A flat page table contains one PTE for every possible virtual page.

For one process:

```text
Virtual pages = 32 GiB / 2 KiB
              = 2²⁴ pages
```

Assuming each PTE occupies 4 bytes:

```text
Flat page-table size = 2²⁴ × 4 bytes
                     = 2²⁶ bytes
                     = 64 MiB per process
```

For 10 processes:

```text
Total flat page-table memory = 10 × 64 MiB
                             = 640 MiB
```

The complete 64 MiB table is required even when the process maps only a small portion of its 32 GiB virtual address space.

---

# Multilevel Page Table

## 27. Why Multilevel Page Tables Are Used

A multilevel page table divides a large flat page table into smaller page-table structures.

Lower-level page tables are allocated only for virtual-address regions that are actually mapped.

This saves memory when the virtual address space is sparse.

---

## 28. Number of Entries in One Page-Table Page

Assume:

```text
Page-table page size = 2 KiB
PTE size             = 4 bytes
```

Therefore:

```text
Entries per page-table page = 2 KiB / 4 bytes
                             = 2048 / 4
                             = 512 entries
                             = 2⁹ entries
```

Therefore, one full page-table level can be indexed using 9 bits.

The 24-bit VPN can be divided as:

```text
24 VPN bits = 6 L1 bits + 9 L2 bits + 9 L3 bits
```

The complete virtual address becomes:

```text
35-bit virtual address
┌──────────┬──────────┬──────────┬─────────────┐
│ L1 index │ L2 index │ L3 index │ Page offset │
│  6 bits  │  9 bits  │  9 bits  │   11 bits   │
└──────────┴──────────┴──────────┴─────────────┘
```

---

## 29. Multilevel Page-Table Structure

```text
L1 entry → Address of selected L2 table
L2 entry → Address of selected L3 table
L3 entry → Physical frame number
```

Each process has its own L1 root table.

A special CPU register stores the physical address of the currently running process’s L1 table.

---

## 30. Coverage of Each Level

### L3 table

One L3 table contains 512 entries.

Each entry maps one 2 KiB page:

```text
L3 table coverage = 512 × 2 KiB
                  = 1 MiB
```

Therefore:

```text
One L3 table occupies 2 KiB and maps 1 MiB.
```

### L2 table

One L2 table contains 512 entries.

Each entry points to an L3 table that maps 1 MiB:

```text
L2 table coverage = 512 × 1 MiB
                  = 512 MiB
```

Therefore:

```text
One L2 table occupies 2 KiB and covers 512 MiB.
```

### L1 table

Each L1 entry points to an L2 table covering 512 MiB.

The 6-bit L1 index can select:

```text
2⁶ = 64 entries
```

Therefore:

```text
L1 coverage = 64 × 512 MiB
            = 32 GiB
```

This covers the complete virtual address space.

---

## 31. Multilevel Translation Example

Suppose Process A generates:

```text
Virtual address = 0x252345678
```

Splitting it into `6 + 9 + 9 + 11` bits produces:

| Field | Value |
|---|---:|
| L1 index | 18 |
| L2 index | 291 |
| L3 index | 138 |
| Page offset | `0x678` |

Assume the page-table-root register contains:

```text
L1 base address = 0x00100000
```

### Step 1: L1 lookup

Each PTE occupies 4 bytes:

```text
L1 PTE address = L1 base + L1 index × PTE size
               = 0x00100000 + 18 × 4
               = 0x00100048
```

Suppose the L1 entry points to:

```text
L2 table base address = 0x00200000
```

### Step 2: L2 lookup

```text
L2 PTE address = L2 base + L2 index × PTE size
               = 0x00200000 + 291 × 4
               = 0x0020048C
```

Suppose the L2 entry points to:

```text
L3 table base address = 0x00300000
```

### Step 3: L3 lookup

```text
L3 PTE address = L3 base + L3 index × PTE size
               = 0x00300000 + 138 × 4
               = 0x00300228
```

Suppose the final PTE contains:

```text
Physical frame number = 0x34567
Present                = 1
Read/write             = 1
Execute                = 0
```

### Step 4: Construct the physical address

The physical address is formed by combining the PFN with the unchanged offset:

```text
Physical address = PFN × page size + offset
                 = 0x34567 × 0x800 + 0x678
                 = 0x1A2B3E78
```

Therefore:

```text
Virtual address: 0x252345678
        ↓
L1 index: 18
        ↓
L2 index: 291
        ↓
L3 index: 138
        ↓
PFN: 0x34567
        +
Offset: 0x678
        ↓
Physical address: 0x1A2B3E78
```

The VPN is replaced by the PFN, while the 11-bit page offset remains unchanged.

---

## 32. Minimum Multilevel Page-Table Memory

For the first mapped virtual page, the following tables are required:

```text
One L1 table = 2 KiB
One L2 table = 2 KiB
One L3 table = 2 KiB
```

Therefore:

```text
Minimum page-table memory = 2 KiB + 2 KiB + 2 KiB
                          = 6 KiB per process
```

However, this does not mean 6 KiB is required for every mapped page.

The same L3 table can map 512 pages:

```text
512 pages × 2 KiB per page = 1 MiB
```

Therefore, the initial 6 KiB of page-table structures can map up to 1 MiB of virtual memory within the corresponding region.

---

## 33. Mapping the Entire 2 GiB Physical Memory

Assume one process maps the complete 2 GiB physical memory into a contiguous part of its virtual address space.

### Number of mapped pages

```text
Mapped pages = 2 GiB / 2 KiB
             = 2³¹ / 2¹¹
             = 2²⁰
             = 1,048,576 pages
```

### Number of L3 tables

Each L3 table maps 1 MiB:

```text
Required L3 tables = 2 GiB / 1 MiB
                   = 2048 tables
```

Memory required:

```text
L3 memory = 2048 × 2 KiB
          = 4 MiB
```

### Number of L2 tables

Each L2 table covers 512 MiB:

```text
Required L2 tables = 2 GiB / 512 MiB
                   = 4 tables
```

Memory required:

```text
L2 memory = 4 × 2 KiB
          = 8 KiB
```

### Number of L1 tables

Only one L1 table is required:

```text
L1 memory = 2 KiB
```

### Total multilevel page-table memory

```text
Total = L3 memory + L2 memory + L1 memory
      = 4 MiB + 8 KiB + 2 KiB
      = 4 MiB + 10 KiB per process
```

---

## 34. Flat vs Multilevel Page-Table Memory

### Per process

| Page-table organization | Memory required |
|---|---:|
| Flat table for complete 32 GiB virtual space | 64 MiB |
| Three-level table mapping 2 GiB | 4 MiB + 10 KiB |
| Three-level table with minimum mapping | 6 KiB |

### For 10 processes

Assume every process maps 2 GiB:

```text
Flat page tables = 10 × 64 MiB
                 = 640 MiB
```

```text
Multilevel page tables = 10 × (4 MiB + 10 KiB)
                       = 40 MiB + 100 KiB
```

| Page-table organization | Per process | For 10 processes |
|---|---:|---:|
| Flat page table | 64 MiB | 640 MiB |
| Multilevel table mapping 2 GiB | 4 MiB + 10 KiB | 40 MiB + 100 KiB |
| Multilevel table, minimum mapping | 6 KiB | 60 KiB |

### Important observation

The multilevel table saves memory because only page-table structures corresponding to mapped virtual regions are created.

If the entire 32 GiB virtual address space is mapped, the leaf PTEs alone require 64 MiB. Additional upper-level tables are also required.

Therefore, when the complete virtual address space is mapped:

```text
Multilevel table size > Flat table size
```

Multilevel page tables are most beneficial when the virtual address space is sparse.

---

# Translation Lookaside Buffer

## 35. What Is a TLB?

The **Translation Lookaside Buffer (TLB)** is a small and fast hardware cache that stores recently used virtual-to-physical page translations.

A TLB entry contains information such as:

```text
Virtual Page Number → Physical Frame Number + Permissions
```

Page tables are stored in memory. Walking a three-level page table may require three memory accesses before the actual data or instruction is accessed.

The TLB avoids repeated page-table walks for recently used pages.

---

## 36. TLB Hit

A TLB hit occurs when the required translation is present in the TLB.

Steps:

1. The CPU generates a virtual address.
2. The MMU extracts the VPN.
3. The MMU searches the TLB.
4. The VPN is found.
5. The corresponding PFN is obtained.
6. The PFN is combined with the offset.
7. The cache or physical memory is accessed.

```text
Virtual address
      ↓
TLB hit
      ↓
Obtain PFN
      ↓
Generate physical address
      ↓
Access cache or RAM
```

---

## 37. TLB Miss

A TLB miss occurs when the required translation is absent from the TLB.

Steps:

1. The CPU generates a virtual address.
2. The MMU searches the TLB.
3. The translation is not found.
4. The page table is walked.
5. If the page is present, the translation is inserted into the TLB.
6. The physical address is generated.
7. The original memory access continues.

```text
Virtual address
      ↓
TLB miss
      ↓
Page-table walk
      ↓
PTE found and present
      ↓
Insert translation into TLB
      ↓
Generate physical address
```

---

## 38. TLB Miss vs Page Fault

A TLB miss does not necessarily cause a page fault.

| Situation | Result |
|---|---|
| Translation is found in the TLB | TLB hit |
| Translation is absent from TLB, but page is in RAM | Page-table walk |
| Translation is absent and PTE says not present | Page fault |
| PTE denies the requested access | Protection fault |

---

## 39. TLB and Multiple Processes

Different processes can use the same virtual page numbers with different mappings.

Therefore, the TLB must distinguish between process address spaces.

Two common techniques are:

- Flush TLB entries during a process switch.
- Tag TLB entries using an Address Space Identifier.

Example:

```text
ASID 5, VPN 100 → PFN 400
ASID 8, VPN 100 → PFN 900
```

The VPN is the same, but the processes and physical-frame mappings are different.

---

# Complete Memory-Access Flow

## 40. Overall Operation

```text
Program is executed
        ↓
OS creates a process
        ↓
Process receives a virtual address space
        ↓
CPU generates a virtual address
        ↓
MMU divides it into VPN and offset
        ↓
TLB lookup
   ┌────┴─────┐
 TLB hit    TLB miss
   │            ↓
   │      Page-table walk
   │       ┌────┴─────┐
   │    Present    Not present
   │       │            ↓
   │       │        Page fault
   │       │            ↓
   │       │     OS obtains page
   │       │     from secondary
   │       │        storage
   └───────┴────────────┘
              ↓
       Obtain physical frame
              ↓
Combine PFN with unchanged offset
              ↓
      Generate physical address
              ↓
       Access cache or RAM
```

---

# Final Summary

1. A **program** is a passive executable file.

2. A **process** is a running instance of a program.

3. The same program can be executed as multiple processes.

4. A process can contain multiple threads.

5. Each process normally has one virtual address space and its own page-table hierarchy.

6. Threads of the same process normally share the virtual address space and page tables.

7. Virtual memory is the complete mechanism that provides address translation, protection, isolation, demand paging and memory sharing.

8. Virtual memory is not a separate physical-memory device.

9. Different processes may use the same virtual addresses without interference because their page tables contain different mappings.

10. Different processes may intentionally map selected virtual pages to the same physical frames.

11. Paging divides virtual memory into pages and physical memory into equal-sized frames.

12. Paging eliminates external fragmentation but may produce limited internal fragmentation.

13. The MMU translates virtual addresses into physical addresses and checks access permissions.

14. A page-table entry contains a physical-frame number and control bits such as present, writable, executable, dirty and accessed bits.

15. A page fault occurs when operating-system intervention is required, commonly because a valid page is not currently present in RAM.

16. The OS can load a missing page from secondary storage, update the page table and restart the interrupted instruction.

17. A multilevel page table saves memory by allocating lower-level tables only for mapped virtual-address regions.

18. A TLB caches recently used address translations and avoids repeated page-table walks.

19. A TLB miss is not the same as a page fault.

20. In the example system, a flat page table requires 64 MiB per process.

21. A three-level page table mapping 2 GiB requires approximately 4 MiB plus 10 KiB per process.

22. Multilevel page tables save memory when the virtual address space is sparse.


