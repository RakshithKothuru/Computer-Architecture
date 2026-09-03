# Virtual Memory, Paging, Page Tables and TLB

## 1. Program

A **program** is a passive file containing instructions and data. It is stored on secondary memory, such as an SSD or hard disk.

Examples include:

- A compiled C executable
- A browser application
- A media player

A program becomes active only when the operating system loads and executes it.

---

## 2. Process

A **process** is a running instance of a program.

A process contains:

- Program instructions
- Global and static data
- Heap
- Stack
- CPU registers
- Program counter
- Operating-system resources
- Virtual address space
- Page-table hierarchy

The same program can be executed multiple times, creating multiple processes:

```text
Program
  ├── Process A
  ├── Process B
  └── Process C
```

Even though these processes execute the same program, they normally have separate virtual address spaces and page tables.

---

## 3. Thread

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

Therefore, different processes have separate virtual address spaces, while threads of the same process share one virtual address space.

---

# Virtual and Physical Memory

## 4. Physical Memory

**Physical memory** refers to the actual RAM installed in a system.

For example:

```text
Physical memory = 2 GiB
                = 2³¹ bytes
```

Therefore, a **31-bit physical address** is sufficient to identify every byte in 2 GiB of physical memory.

Physical addresses range from:

```text
0 to 2³¹ − 1
```

---

## 5. Virtual Memory

**Virtual memory** is a memory-management mechanism that allows processes to use virtual addresses instead of directly using physical addresses.

It is implemented using:

- CPU
- Memory Management Unit
- Operating system
- Page tables
- TLB
- Physical memory
- Secondary storage

Virtual memory is not a separate physical memory device. It is an abstraction created using address translation.

Virtual memory provides:

- Process isolation
- Memory protection
- Non-contiguous physical-memory allocation
- Demand paging
- Memory sharing
- A simple address space for programs

---

## 6. Virtual Address Space

Each process normally has its own **virtual address space**.

For example:

```text
Process A: Virtual addresses 0 to 32 GiB − 1
Process B: Virtual addresses 0 to 32 GiB − 1
```

The same virtual address may map to different physical addresses:

| Process | Virtual address | Physical address |
|---|---:|---:|
| Process A | `0x1000` | `0xA000` |
| Process B | `0x1000` | `0xF000` |

This is possible because the two processes have different page-table mappings.

It is common to say that each process has its own virtual memory. More precisely:

> The system provides the virtual-memory mechanism, while each process has its own virtual address space and page-table hierarchy.

---

## 7. Can Virtual Memory Be Larger Than Physical Memory?

Yes. A process’s virtual address space can be larger than the installed physical memory.

For example:

```text
Virtual address space per process = 32 GiB
Physical memory                   = 2 GiB
```

This does not mean that the process occupies 32 GiB of RAM.

The 32 GiB represents the range of virtual addresses available to the process. Only the currently required pages need to be present in physical memory.

A virtual page may be:

- Present in physical memory
- Stored on secondary memory
- Not yet allocated
- Shared with another process
- Mapped to a file
- Invalid or unused

Therefore:

```text
Virtual address-space size ≠ Physical memory occupied
```

---

# Process Isolation and Memory Sharing

## 8. Process Isolation

One process normally cannot access another process’s private memory.

Suppose Process A and Process B both generate virtual address `0x1000`:

```text
Process A: VA 0x1000 → Process A page table → PA 0xA000
Process B: VA 0x1000 → Process B page table → PA 0xF000
```

When Process A is running, the MMU uses Process A’s page table. It cannot use Process B’s private mappings.

Isolation is implemented using:

- Separate page tables
- Page permission bits
- User and kernel privilege levels
- MMU protection checks

---

## 9. Shared Memory

Different processes can intentionally map their virtual pages to the same physical frame:

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

Therefore, processes normally have separate address spaces, but selected pages can intentionally share physical memory.

---

# Paging

## 10. What Is Paging?

Paging divides virtual and physical memory into equal-sized blocks.

- A virtual-memory block is called a **page**.
- A physical-memory block is called a **frame**.
- Page size and frame size are equal.

For example:

```text
Page size = Frame size = 2 KiB
```

Consecutive virtual pages do not need to be stored in consecutive physical frames:

```text
Virtual page 0 → Physical frame 100
Virtual page 1 → Physical frame 25
Virtual page 2 → Physical frame 700
Virtual page 3 → Physical frame 41
```

The process sees continuous virtual memory even though the physical frames are scattered throughout RAM.

---

## 11. Why Paging Is Used

Paging provides:

- Non-contiguous physical-memory allocation
- Elimination of external fragmentation
- Easier physical-memory allocation
- Per-page protection
- Page sharing
- Demand paging

Paging can produce a small amount of **internal fragmentation** in the final allocated page.

For example, if a process needs one byte more than a complete 2 KiB page, another complete 2 KiB page must be allocated.

---

# Address Translation Example

## 12. Given System

Assume:

```text
Number of processes               = 10
Virtual address space per process = 32 GiB
Physical memory                   = 2 GiB
Page size                         = 2 KiB
Page-table entry size             = 4 bytes
```

---

## 13. Virtual Address Size

```text
32 GiB = 2³⁵ bytes
```

Therefore:

```text
Virtual address size = 35 bits
```

---

## 14. Physical Address Size

```text
2 GiB = 2³¹ bytes
```

Therefore:

```text
Physical address size = 31 bits
```

---

## 15. Page Offset

```text
Page size = 2 KiB
          = 2¹¹ bytes
```

Therefore:

```text
Page offset = 11 bits
```

The page offset identifies a byte within a page.

---

## 16. Virtual Address Format

```text
VPN bits = Virtual address bits − Offset bits
         = 35 − 11
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
Virtual pages = 32 GiB / 2 KiB
              = 2³⁵ / 2¹¹
              = 2²⁴
              = 16,777,216 pages
```

---

## 17. Physical Address Format

```text
PFN bits = Physical address bits − Offset bits
         = 31 − 11
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
Physical frames = 2 GiB / 2 KiB
                = 2³¹ / 2¹¹
                = 2²⁰
                = 1,048,576 frames
```

All processes, the operating system and the page tables share these physical frames.

---

## 18. Basic Address Translation

The virtual address is divided into:

```text
Virtual Page Number | Page offset
```

The page table translates the VPN into a PFN:

```text
VPN → Page table → PFN
```

The physical address becomes:

```text
Physical Frame Number | Same page offset
```

Therefore:

```text
Virtual address:  VPN | Offset
                         ↓
Physical address: PFN | Same offset
```

The page offset remains unchanged because the page and frame sizes are equal.

---

# MMU and Page Table

## 19. Memory Management Unit

The **Memory Management Unit (MMU)** is the hardware responsible for translating virtual addresses into physical addresses.

The MMU also performs:

- TLB lookup
- Page-table lookup
- Read/write permission checking
- Execute-permission checking
- User/kernel access checking
- Protection-fault generation

The operating system creates and maintains the page tables. The MMU uses those tables during address translation.

---

## 20. Page Table

A **page table** stores mappings from virtual page numbers to physical frame numbers.

Example:

| Virtual page number | Physical frame number |
|---:|---:|
| 0 | 500 |
| 1 | 110 |
| 2 | Not present |
| 3 | 800 |

Each process normally has its own page-table hierarchy.

During a process context switch, the operating system changes a CPU register that points to the new process’s page-table root.

Threads belonging to the same process normally share the same page table.

---

## 21. Page-Table Entry

A **Page-Table Entry (PTE)** commonly contains:

- Physical frame number
- Present or valid bit
- Read/write permission
- User/kernel permission
- Execute-disable bit
- Accessed bit
- Dirty bit

### Present bit

```text
Present = 1 → Page is mapped to a physical frame
Present = 0 → Page is absent or the mapping is invalid
```

### Dirty bit

The dirty bit indicates that the page has been modified.

If a dirty page is removed from physical memory, its updated contents must normally be written to secondary storage.

### Accessed bit

The accessed bit indicates that the page was recently used. The operating system may use it while selecting a page for replacement.

---

# Flat Page Table

## 22. Flat Page-Table Size

A flat page table contains one PTE for every possible virtual page.

Number of virtual pages per process:

```text
32 GiB / 2 KiB = 2²⁴ pages
```

Assuming each PTE occupies 4 bytes:

```text
Flat table size = 2²⁴ × 4 bytes
                = 2²⁶ bytes
                = 64 MiB per process
```

For 10 processes:

```text
Total size = 10 × 64 MiB
           = 640 MiB
```

The problem is that the complete 64 MiB table is required even when a process uses only a small portion of its 32 GiB virtual address space.

---

# Multilevel Page Table

## 23. Why Multilevel Page Tables Are Used

A multilevel page table divides a large flat page table into smaller tables.

Lower-level tables are allocated only for virtual-address regions that are actually mapped.

This saves memory when the virtual address space is sparse.

---

## 24. Dividing the Virtual Page Number

Assume:

```text
Page-table page size = 2 KiB
PTE size             = 4 bytes
```

Number of entries in one page-table page:

```text
Entries = 2 KiB / 4 bytes
        = 2048 / 4
        = 512 entries
        = 2⁹ entries
```

Therefore, 9 bits are required to select an entry from a full page-table page.

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

## 25. Multilevel Translation

The three levels work as follows:

```text
L1 entry → Address of an L2 table
L2 entry → Address of an L3 table
L3 entry → Physical frame number
```

For a virtual address:

```text
L1 index = 18
L2 index = 291
L3 index = 138
Offset   = 0x678
```

Translation occurs as follows:

```text
Page-table root
      ↓ Use L1 index 18
Selected L2 table
      ↓ Use L2 index 291
Selected L3 table
      ↓ Use L3 index 138
Obtain the physical frame number
      ↓
Combine PFN with offset 0x678
      ↓
Generate physical address
```

The L1 and L2 entries point to lower-level page tables. The final L3 entry contains the physical frame number and permission bits.

---

## 26. Coverage of Each Level

### L3 table

One L3 table contains 512 entries, and each entry maps one 2 KiB page:

```text
L3 coverage = 512 × 2 KiB
            = 1 MiB
```

Therefore:

```text
One 2 KiB L3 table maps 1 MiB of virtual memory.
```

### L2 table

One L2 table contains 512 entries. Each entry points to an L3 table that maps 1 MiB:

```text
L2 coverage = 512 × 1 MiB
            = 512 MiB
```

Therefore:

```text
One 2 KiB L2 table covers 512 MiB.
```

### L1 table

Each L1 entry points to an L2 table covering 512 MiB.

The 6-bit L1 index provides 64 possible entries:

```text
L1 coverage = 64 × 512 MiB
            = 32 GiB
```

Therefore, the L1 table covers the complete 32 GiB virtual address space.

---

## 27. Minimum Multilevel Page-Table Size

For the first mapped page, the following structures are required:

```text
One L1 table = 2 KiB
One L2 table = 2 KiB
One L3 table = 2 KiB
```

Therefore:

```text
Minimum size = 2 KiB + 2 KiB + 2 KiB
             = 6 KiB per process
```

This does not mean that every page requires another 6 KiB.

One L3 table contains 512 entries and maps:

```text
512 × 2 KiB = 1 MiB
```

Therefore, the same initial table path can map as much as 1 MiB within that virtual-address region.

---

## 28. Mapping 2 GiB Using a Multilevel Page Table

Assume one process maps a contiguous 2 GiB region.

Number of mapped pages:

```text
2 GiB / 2 KiB = 2²⁰
              = 1,048,576 pages
```

### L3 tables

Each L3 table maps 1 MiB:

```text
Required L3 tables = 2 GiB / 1 MiB
                   = 2048
```

Memory required:

```text
2048 × 2 KiB = 4 MiB
```

### L2 tables

Each L2 table covers 512 MiB:

```text
Required L2 tables = 2 GiB / 512 MiB
                   = 4
```

Memory required:

```text
4 × 2 KiB = 8 KiB
```

### L1 table

One L1 table is required:

```text
L1 memory = 2 KiB
```

### Total multilevel page-table size

```text
Total = 4 MiB + 8 KiB + 2 KiB
      = 4 MiB + 10 KiB per process
```

---

## 29. Flat vs Multilevel Page Tables

| Page-table organization | Per process | For 10 processes |
|---|---:|---:|
| Flat table | 64 MiB | 640 MiB |
| Multilevel table mapping 2 GiB | 4 MiB + 10 KiB | 40 MiB + 100 KiB |
| Multilevel table with minimum mapping | 6 KiB | 60 KiB |

A multilevel page table saves memory when the virtual address space is sparsely mapped.

If the entire 32 GiB virtual address space is mapped, the leaf PTEs alone require 64 MiB. The upper-level tables require additional memory.

Therefore, for a completely mapped address space:

```text
Multilevel page-table size > Flat page-table size
```

---

# Page Faults

## 30. What Happens When a Page Is Not Present?

Suppose the CPU accesses a valid virtual page that is currently absent from physical memory.

The following sequence occurs:

1. The CPU generates a virtual address.
2. The MMU searches the TLB.
3. The translation is not found in the TLB.
4. The MMU or hardware page walker checks the page table.
5. The PTE indicates `Present = 0`.
6. The CPU raises a page-fault exception.
7. Control transfers to the operating system.
8. The OS checks whether the virtual address is valid.
9. The OS locates the page on secondary storage.
10. The OS finds a free physical frame.
11. If no frame is free, another page is selected for eviction.
12. A dirty evicted page is written back to storage.
13. The required page is loaded into the selected frame.
14. The OS updates the PTE.
15. The TLB is updated or invalidated as required.
16. The interrupted instruction is restarted.

The simplified sequence is:

```text
Page not present
      ↓
Page fault
      ↓
OS validates the virtual address
      ↓
Find free frame or evict another page
      ↓
Load the page from secondary storage
      ↓
Update the PTE and TLB
      ↓
Restart the instruction
```

Loading a page only when it is first required is called **demand paging**.

---

## 31. Valid and Invalid Page Faults

A page fault is not always an error.

| Situation | Operating-system action |
|---|---|
| Valid page is stored on disk | Load it into RAM |
| Valid page has not yet been allocated | Allocate a new page |
| Copy-on-write page is modified | Create a private copy |
| Address is outside valid process memory | Report an access violation |
| Write is attempted on a read-only page | Report a protection fault |

An invalid access may terminate the process with an error such as a segmentation fault.

---

# Translation Lookaside Buffer

## 32. What Is a TLB?

The **Translation Lookaside Buffer (TLB)** is a small and fast hardware cache that stores recently used virtual-to-physical address translations.

A TLB entry stores information similar to:

```text
Virtual Page Number → Physical Frame Number + Permissions
```

Without a TLB, a three-level page table may require three memory accesses for translation before the actual data is accessed.

The TLB improves performance by avoiding repeated page-table walks.

---

## 33. TLB Hit

A TLB hit occurs when the required translation is already present in the TLB.

```text
CPU generates virtual address
          ↓
MMU searches TLB
          ↓
TLB hit
          ↓
Obtain PFN
          ↓
Combine PFN with offset
          ↓
Generate physical address
```

No page-table walk is required.

---

## 34. TLB Miss

A TLB miss occurs when the translation is absent from the TLB.

```text
CPU generates virtual address
          ↓
MMU searches TLB
          ↓
TLB miss
          ↓
Walk the page table
          ↓
Page is present
          ↓
Insert translation into TLB
          ↓
Continue memory access
```

A TLB miss does not necessarily mean that the page is absent from physical memory.

---

## 35. TLB Miss vs Page Fault

| Situation | Result |
|---|---|
| Translation is present in TLB | TLB hit |
| Translation is absent from TLB, but page is in RAM | TLB miss and page-table walk |
| PTE indicates that the page is absent | Page fault |
| PTE denies the requested operation | Protection fault |

Therefore:

> Every page fault normally involves a failed translation, but every TLB miss does not cause a page fault.

---

# Complete Memory-Access Flow

## 36. Overall Flow

```text
CPU generates virtual address
          ↓
Separate VPN and offset
          ↓
Search TLB
     ┌────┴─────┐
  TLB hit    TLB miss
     │             ↓
     │       Walk page table
     │        ┌────┴─────┐
     │     Present    Not present
     │        │             ↓
     │        │         Page fault
     │        │             ↓
     │        │      OS loads page
     │        │      into a frame
     └────────┴─────────────┘
                  ↓
             Obtain PFN
                  ↓
        Combine PFN with offset
                  ↓
        Generate physical address
                  ↓
           Access cache or RAM
```

---

# Key Points

- A program is a passive executable file.
- A process is a running instance of a program.
- A process may contain multiple threads.
- Each process normally has one virtual address space and its own page-table hierarchy.
- Threads of the same process normally share the virtual address space and page tables.
- Virtual memory is an abstraction, not a separate physical-memory device.
- The MMU translates virtual addresses into physical addresses.
- Paging divides virtual memory into pages and physical memory into equal-sized frames.
- A page table maps virtual page numbers to physical frame numbers.
- The page offset remains unchanged during translation.
- A multilevel page table allocates lower-level tables only for mapped regions.
- A page fault occurs when operating-system intervention is required for an address translation.
- The TLB caches recent translations and avoids repeated page-table walks.
- A TLB miss is not necessarily a page fault.
- In the example, a flat table requires 64 MiB per process.
- A three-level table mapping 2 GiB requires approximately 4 MiB plus 10 KiB per process.


