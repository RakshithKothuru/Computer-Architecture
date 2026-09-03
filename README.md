# Processes, Virtual Memory, Paging, Page Tables, MMU and TLB

## 1. Program, Process and Thread

### Program

A **program** is a passive file containing instructions and data stored on secondary memory, such as an executable file on an SSD.

Examples include:

- A compiled C executable
- A web browser application
- A media player executable

A program becomes active only when it is loaded and executed.

### Process

A **process** is a running instance of a program. It includes:

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

The same program can be executed multiple times, creating multiple processes.

For example, opening two separate instances of the same application may create:

```text
Program executable
      ├── Process A
      └── Process B
```

Although both processes execute the same program, they normally have separate virtual address spaces and separate page tables.

### Thread

A process may contain one or more threads.

Threads belonging to the same process normally share:

- Code
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
Physical addresses        = 0 to 2 GiB − 1
```

If the system has 2 GiB of byte-addressable physical memory:

\[
2\text{ GiB}=2^{31}\text{ bytes}
\]

Therefore, 31 bits are sufficient to identify every byte in that physical memory.

The physical address is generally generated after virtual-to-physical address translation.

---

## 3. Virtual Memory and Virtual Address Space

### Virtual memory

**Virtual memory** is a memory-management mechanism implemented through cooperation between:

- CPU
- Memory Management Unit
- Operating system
- Page tables
- TLB
- Physical RAM
- Secondary storage

It allows programs to use virtual addresses instead of directly using physical RAM addresses.

Virtual memory is not a separate physical memory chip. It is an abstraction created using address translation and memory-management mechanisms.

### Virtual address space

Each process normally receives its own **virtual address space**.

For example:

```text
Process A: Virtual addresses 0 to 32 GiB − 1
Process B: Virtual addresses 0 to 32 GiB − 1
Process C: Virtual addresses 0 to 32 GiB − 1
```

The same virtual address can represent different physical locations in different processes:

| Process | Virtual address | Physical address |
|---|---:|---:|
| Process A | `0x1000` | `0xA000` |
| Process B | `0x1000` | `0xF000` |

This happens because each process has its own page-table hierarchy.

### Correct terminology

It is common to informally say:

> Each process has its own virtual memory.

More precisely:

> The system provides the virtual-memory mechanism, and each process has its own virtual address space and page-table mappings.

A process does not own a separate physical virtual-memory device.

---

## 4. Process Isolation and Shared Physical Memory

One process normally cannot access another process’s virtual address space.

Suppose Process A tries to access virtual address `0x1000`. The MMU uses Process A’s page table, not Process B’s page table.

```text
Process A virtual address → Process A page table
Process B virtual address → Process B page table
```

Therefore, Process A cannot access Process B’s private memory simply by generating the same virtual address.

However, different processes can intentionally map the same physical frame:

```text
Process A virtual page 5 ─┐
                          ├── Physical frame 100
Process B virtual page 9 ─┘
```

This is used for:

- Shared memory
- Shared libraries
- Inter-process communication
- Memory-mapped files
- Copy-on-write

Therefore:

> Processes have separate virtual address spaces, but some of their virtual pages may intentionally map to the same physical frames.

Protection bits in the page-table entries control whether a process can read, write or execute a mapped page.

---

## 5. Why Virtual Memory Is Needed

Virtual memory provides the following benefits.

### 5.1 Process isolation

Each process receives an independent address space. One process normally cannot corrupt another process’s private memory.

### 5.2 Protection

Memory pages can be marked as:

- Read-only
- Read/write
- Executable
- Non-executable
- User accessible
- Kernel only

For example, program instructions can be read-only and executable, while stack memory can be writable but non-executable.

### 5.3 Easier programming

Programs can use a simple, continuous virtual address space even when the corresponding physical frames are scattered throughout RAM.

### 5.4 Programs larger than physical memory

Only the currently needed pages must be present in RAM. Other pages may remain on secondary storage until required.

However, frequent movement between RAM and storage severely reduces performance.

### 5.5 Efficient physical-memory allocation

A process does not require one large contiguous physical-memory region. Its pages can be stored in any available physical frames.

### 5.6 Memory sharing

Different processes can share selected physical pages without sharing their entire address spaces.

---

# Contiguous Memory Allocation

Before paging, consider systems that allocate one contiguous physical-memory region to each process.

## 6. Static Partitioning

In **static partitioning**, physical memory is divided into fixed partitions before processes are loaded.

Example:

```text
Physical memory
├── OS region
├── Partition 1: 256 MiB
├── Partition 2: 256 MiB
├── Partition 3: 512 MiB
└── Partition 4: 1 GiB
```

Each partition can contain one process.

### Advantages

- Simple to implement
- Low allocation overhead
- Easy process placement
- Predictable partition boundaries

### Disadvantages

- Number of simultaneous processes is limited by the number of partitions.
- A process larger than every available partition cannot run.
- Unused space inside a partition is wasted.
- Causes **internal fragmentation**.

### Internal fragmentation

Suppose a 100 MiB process is loaded into a 256 MiB partition:

\[
256-100=156\text{ MiB wasted}
\]

This unused memory is inside the allocated partition, so it is called internal fragmentation.

---

## 7. Dynamic Partitioning

In **dynamic partitioning**, partitions are created according to process sizes when processes are loaded.

Example:

```text
Process A requires 100 MiB → Allocate 100 MiB
Process B requires 250 MiB → Allocate 250 MiB
Process C requires 80 MiB  → Allocate 80 MiB
```

Allocation strategies include:

- First fit
- Best fit
- Worst fit
- Next fit

### Advantages

- Partition size closely matches process size.
- Less internal fragmentation than fixed partitioning.
- Memory is initially used more efficiently.

### Disadvantages

- Free memory becomes divided into small separated holes.
- Causes **external fragmentation**.
- Allocation and deallocation are more complex.
- Compaction may be required.

### External fragmentation

Suppose free memory contains:

```text
100 MiB hole + 60 MiB hole + 80 MiB hole
```

Total free memory is:

\[
100+60+80=240\text{ MiB}
\]

However, a 200 MiB process cannot be allocated because no single contiguous 200 MiB region exists.

The available memory is outside allocated regions but scattered into small holes. This is called external fragmentation.

### Compaction

The OS can move processes so that scattered free spaces combine into one large region.

However, compaction:

- Consumes time
- Requires process relocation
- Creates significant data movement
- Can temporarily disrupt execution

Paging largely removes the requirement that a process occupy contiguous physical memory.

---

# Paging

## 8. What Is Paging?

Paging divides virtual and physical memory into equal-sized fixed blocks.

- A virtual-memory block is called a **page**.
- A physical-memory block is called a **frame** or **page frame**.
- Page size and frame size are equal.

Example:

```text
Page size = Frame size = 2 KiB
```

A process’s consecutive virtual pages can be placed in non-consecutive physical frames:

```text
Virtual page 0 → Physical frame 100
Virtual page 1 → Physical frame 25
Virtual page 2 → Physical frame 700
Virtual page 3 → Physical frame 41
```

To the process, the virtual pages appear continuous. In physical memory, the frames may be scattered.

---

## 9. Why Paging Is Required

Paging provides:

- Non-contiguous physical-memory allocation
- Elimination of external fragmentation
- Easier memory allocation
- Per-page protection
- Page sharing
- Demand paging
- Efficient process isolation

Because every frame has the same size, any virtual page can be placed in any available physical frame.

### Fragmentation in paging

Paging eliminates external fragmentation, but it may cause a small amount of internal fragmentation.

If the page size is 2 KiB and a process requires 1 byte more than a complete page, another 2 KiB page must be allocated.

The unused portion of the final page is internal fragmentation.

---

# Address Translation

## 10. Virtual and Physical Address Formats

Assume:

- Virtual address space per process = 32 GiB
- Physical memory = 2 GiB
- Page size = 2 KiB
- Memory is byte-addressable

### Virtual address size

\[
32\text{ GiB}=2^{35}\text{ bytes}
\]

Therefore, the virtual address is 35 bits.

### Physical address size

\[
2\text{ GiB}=2^{31}\text{ bytes}
\]

Therefore, the physical address is 31 bits.

### Page offset size

\[
2\text{ KiB}=2^{11}\text{ bytes}
\]

Therefore, 11 bits are required for the page offset.

### Virtual address format

\[
35-11=24\text{ VPN bits}
\]

```text
35-bit virtual address
┌────────────────────────────┬─────────────┐
│ 24-bit Virtual Page Number │11-bit offset│
└────────────────────────────┴─────────────┘
```

### Physical address format

\[
31-11=20\text{ PFN bits}
\]

```text
31-bit physical address
┌─────────────────────────────┬─────────────┐
│20-bit Physical Frame Number │11-bit offset│
└─────────────────────────────┴─────────────┘
```

During translation:

```text
VPN | Offset → PFN | Same offset
```

The offset remains unchanged because pages and frames have the same size.

---

## 11. Memory Management Unit

The **Memory Management Unit (MMU)** is a hardware unit responsible for translating CPU-generated virtual addresses into physical addresses.

The MMU also performs:

- Permission checking
- User/kernel access checking
- Read/write checking
- Execute-permission checking
- TLB lookup
- Page-table walking or initiation
- Memory-protection fault generation

The operating system creates and maintains the page tables. The MMU uses those page tables to perform translation.

### Basic translation

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

## 12. What Is a Page Table?

A **page table** stores mappings from virtual page numbers to physical frame numbers.

Conceptually:

| Virtual page number | Physical frame number |
|---:|---:|
| 0 | 500 |
| 1 | 110 |
| 2 | Not present |
| 3 | 800 |

Each process normally has its own page-table hierarchy.

A process does not have one PTE. It has many PTEs—normally one for each mapped virtual page.

During a context switch, the OS changes a special CPU register containing the root address of the new process’s page table.

Threads of the same process normally share the same page tables because they share the same virtual address space.

---

## 13. Page-Table Entry

A **Page-Table Entry (PTE)** normally contains:

- Physical frame number
- Present/valid bit
- Read/write permission
- User/kernel permission
- Execute-disable permission
- Accessed/reference bit
- Dirty/modified bit
- Cache-control bits
- Other architecture-specific control bits

### Present bit

The present bit indicates whether the required page is currently available in physical memory.

```text
Present = 1 → Page is currently mapped to a physical frame
Present = 0 → Page is not currently present or mapping is invalid
```

### Dirty bit

The dirty bit indicates that the page has been modified after being loaded into physical memory.

If a dirty page is evicted, its updated contents generally must be written back to secondary storage.

### Accessed bit

The accessed bit indicates that the page has recently been used. The OS can use it while selecting pages for replacement.

---

# Page Faults and Demand Paging

## 14. What Happens When a Page Is Not Present?

The phrase “page frame is not found in the page table” should be stated more precisely:

> The page-table entry for the requested virtual page indicates that the page is not currently present in physical memory.

The following sequence occurs:

1. The CPU generates a virtual address.
2. The MMU searches the TLB.
3. If the TLB misses, the page table is examined.
4. The required PTE has `Present = 0`.
5. The MMU raises a **page-fault exception**.
6. Control transfers from the process to the operating system.
7. The OS checks whether the virtual address is valid for that process.
8. If the address is invalid, the OS reports a memory-access violation.
9. If the address is valid but the page is not in RAM, the OS locates it on secondary storage.
10. The OS finds a free physical frame.
11. If no free frame exists, the OS selects another page for eviction.
12. If the selected page is dirty, it is written to secondary storage.
13. The required page is read from secondary storage into the selected frame.
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
OS validates address
      ↓
Find free frame or evict a page
      ↓
Load requested page from storage
      ↓
Update PTE and TLB
      ↓
Restart instruction
```

This process is called **demand paging** when a page is loaded only when it is first required.

### Invalid access versus normal page fault

A page fault is not always an error.

| Situation | OS response |
|---|---|
| Valid page stored on disk | Load it into RAM |
| First access to a valid unallocated page | Allocate a new page |
| Copy-on-write page is modified | Create a private copy |
| Address is outside the process’s valid space | Report an access violation |
| Write attempted on read-only page | Report a protection fault |

---

# Flat and Multilevel Page Tables

## 15. Flat Page Table

A flat page table contains one PTE for every possible virtual page.

For a 32 GiB virtual address space with 2 KiB pages:

\[
\text{Number of virtual pages}
=
\frac{32\text{ GiB}}{2\text{ KiB}}
=
2^{24}
\]

Assuming each PTE occupies 4 bytes:

\[
\text{Flat page-table size}
=
2^{24}\times4
=
64\text{ MiB per process}
\]

For 10 processes:

\[
10\times64\text{ MiB}=640\text{ MiB}
\]

This memory is required even when a process maps only a small part of its 32 GiB virtual address space.

---

## 16. Multilevel Page Table

A multilevel page table divides the flat page table into multiple levels.

Assume:

- Page-table page size = 2 KiB
- PTE size = 4 bytes

Each page-table page contains:

\[
\frac{2\text{ KiB}}{4\text{ bytes}}
=
512
=
2^9\text{ entries}
\]

The 24-bit virtual page number can be divided as:

```text
35-bit virtual address
┌──────────┬──────────┬──────────┬─────────────┐
│ L1 index │ L2 index │ L3 index │ Page offset │
│  6 bits  │  9 bits  │  9 bits  │   11 bits   │
└──────────┴──────────┴──────────┴─────────────┘
```

The levels operate as follows:

```text
L1 entry → Address of selected L2 table
L2 entry → Address of selected L3 table
L3 entry → Physical frame number
```

Only the required lower-level tables are allocated.

If a large virtual-address region is unused, the corresponding L2 and L3 tables do not need to exist.

---

## 17. Multilevel Translation Example

Suppose Process A generates:

```text
Virtual address = 0x252345678
```

Splitting it according to `6 + 9 + 9 + 11` bits gives:

| Field | Value |
|---|---:|
| L1 index | 18 |
| L2 index | 291 |
| L3 index | 138 |
| Offset | `0x678` |

Assume the page-table-root register contains:

```text
L1 base address = 0x00100000
```

### L1 lookup

\[
\text{L1 PTE address}
=
0x00100000+(18\times4)
=
0x00100048
\]

Suppose this L1 entry points to:

```text
L2 table base address = 0x00200000
```

### L2 lookup

\[
\text{L2 PTE address}
=
0x00200000+(291\times4)
=
0x0020048C
\]

Suppose this entry points to:

```text
L3 table base address = 0x00300000
```

### L3 lookup

\[
\text{L3 PTE address}
=
0x00300000+(138\times4)
=
0x00300228
\]

Suppose the final PTE contains:

```text
Physical frame number = 0x34567
Present                = 1
Read/write             = 1
```

The physical address is:

\[
PA=(PFN\ll11)\;|\;\text{offset}
\]

\[
PA=(0x34567\ll11)\;|\;0x678
\]

\[
\boxed{PA=0x1A2B3E78}
\]

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

---

## 18. How Multilevel Page Tables Save Memory

One L3 table contains 512 entries, and each entry maps one 2 KiB page:

\[
512\times2\text{ KiB}=1\text{ MiB}
\]

Therefore:

- One 2 KiB L3 table maps 1 MiB of virtual memory.
- One L2 entry covers 1 MiB.
- One 2 KiB L2 table contains 512 entries and covers 512 MiB.
- One L1 entry covers 512 MiB.
- The L1 table can cover the complete 32 GiB virtual address space.

For the first mapped page, the minimum allocation in this simplified example is:

\[
2\text{ KiB L1}
+
2\text{ KiB L2}
+
2\text{ KiB L3}
=
6\text{ KiB}
\]

The same L3 table can map as many as 512 pages within its 1 MiB virtual region. Therefore, 6 KiB is not required for every translated page.

---

## 19. Example: Entire 2 GiB Physical Memory Is Mapped

Assume one process maps 2 GiB of virtual address space using 2 KiB pages.

### Number of mapped pages

\[
\frac{2\text{ GiB}}{2\text{ KiB}}
=
2^{20}
=
1{,}048{,}576\text{ pages}
\]

### Required L3 tables

Each L3 table maps 1 MiB:

\[
\frac{2\text{ GiB}}{1\text{ MiB}}
=
2048\text{ L3 tables}
\]

Memory required:

\[
2048\times2\text{ KiB}
=
4\text{ MiB}
\]

### Required L2 tables

Each L2 table covers 512 MiB:

\[
\frac{2\text{ GiB}}{512\text{ MiB}}
=
4\text{ L2 tables}
\]

Memory required:

\[
4\times2\text{ KiB}
=
8\text{ KiB}
\]

### Required L1 table

One L1 table is required:

\[
2\text{ KiB}
\]

### Total multilevel page-table size

\[
4\text{ MiB}+8\text{ KiB}+2\text{ KiB}
\]

\[
\boxed{4\text{ MiB}+10\text{ KiB per process}}
\]

### Comparison

| Page-table organization | Memory per process | Memory for 10 processes |
|---|---:|---:|
| Flat table | 64 MiB | 640 MiB |
| Three-level table mapping 2 GiB | 4 MiB + 10 KiB | 40 MiB + 100 KiB |
| Three-level table, minimum mapping | 6 KiB | 60 KiB |

The multilevel table saves memory because only page-table structures corresponding to mapped virtual regions are created.

If the entire 32 GiB virtual address space is mapped, the multilevel page table requires slightly more memory than a flat table because the leaf entries still require 64 MiB and additional upper-level tables are also needed.

---

# Translation Lookaside Buffer

## 20. What Is a TLB?

The **Translation Lookaside Buffer (TLB)** is a small and fast hardware cache that stores recently used virtual-to-physical page translations.

It stores information similar to:

```text
Virtual page number → Physical frame number + permissions
```

Page tables are stored in memory. Walking a multilevel page table can require several memory accesses before the actual instruction or data is accessed.

The TLB reduces this translation delay.

---

## 21. TLB Working

### TLB hit

1. The CPU generates a virtual address.
2. The MMU extracts the VPN.
3. The MMU searches the TLB.
4. A matching translation is found.
5. The PFN is obtained directly.
6. The PFN is combined with the offset.
7. Physical memory or cache is accessed.

```text
Virtual address
      ↓
TLB hit
      ↓
Obtain PFN
      ↓
Generate physical address
```

### TLB miss

1. The CPU generates a virtual address.
2. The translation is not found in the TLB.
3. A page-table walk is performed.
4. If the page is present, its translation is inserted into the TLB.
5. The memory access continues.

```text
Virtual address
      ↓
TLB miss
      ↓
Page-table walk
      ↓
PTE found and present
      ↓
Update TLB
      ↓
Generate physical address
```

### TLB miss versus page fault

A TLB miss does not necessarily cause a page fault.

| Condition | Result |
|---|---|
| Translation found in TLB | TLB hit |
| Translation absent from TLB, page present in RAM | Page-table walk |
| Translation absent and PTE says not present | Page fault |
| PTE denies requested operation | Protection fault |

---

## 22. TLB and Multiple Processes

Because different processes can use the same virtual addresses, the TLB must distinguish between their translations.

Two common approaches are:

- Flush relevant TLB entries during a process switch.
- Tag entries with an Address Space Identifier (ASID) or Process Context Identifier (PCID).

Example:

```text
ASID 5, VPN 100 → PFN 400
ASID 8, VPN 100 → PFN 900
```

The VPN is identical, but the process identifiers and physical-frame mappings are different.

---

# Complete Memory-Access Flow

## 23. Overall Operation

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
   │       │     OS loads the page
   │       │       from storage
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

# Important Summary

1. A program is a passive executable file.

2. A process is a running instance of a program.

3. The same program can create multiple processes.

4. A process can contain multiple threads.

5. Each process normally has one virtual address space and its own page-table hierarchy.

6. Threads of the same process normally share the virtual address space and page tables.

7. Virtual memory is the complete mechanism that provides address translation, isolation, protection, demand paging and sharing.

8. Virtual memory is not a separate physical-memory device.

9. Different processes may use identical virtual addresses without interference because their page tables map those addresses independently.

10. Different processes can intentionally map some virtual pages to the same physical frames.

11. Paging divides virtual memory into pages and physical memory into equal-sized frames.

12. Paging eliminates external fragmentation but may create limited internal fragmentation.

13. The MMU translates virtual addresses into physical addresses and checks permissions.

14. A PTE contains a physical-frame number and control information such as present, writable, executable, dirty and accessed bits.

15. A page fault occurs when OS intervention is required, commonly because a valid page is not currently present in RAM.

16. The OS can load the missing page from secondary storage, update the page table and restart the interrupted instruction.

17. A multilevel page table reduces page-table memory by allocating lower-level tables only for mapped virtual-address regions.

18. A TLB caches recent address translations and avoids repeated page-table walks.

19. A TLB miss is not the same as a page fault.

20. In the example system, a flat page table requires 64 MiB per process, while a three-level table mapping 2 GiB requires approximately 4 MiB plus 10 KiB per process.
