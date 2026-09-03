# Virtual Memory, Paging, Page Tables and TLB

## 1. Program and Process

A **program** is a passive file containing instructions and data stored on secondary memory. A **process** is a running instance of that program.

The same program can be executed multiple times, creating multiple processes:

```text
Program
  ├── Process A
  ├── Process B
  └── Process C
```

Each process normally has:

- Its own virtual address space
- Its own page-table hierarchy
- Program instructions and data
- Heap and stack
- CPU execution state
- Operating-system resources

Although Process A and Process B may execute the same program, they normally have separate virtual-to-physical address mappings.

---

## 2. Virtual and Physical Memory

### Physical Memory

**Physical memory** is the actual RAM installed in the system.

For example:

```text
Physical memory = 2 GiB
                = 2³¹ bytes
```

Therefore, a **31-bit physical address** is sufficient to identify every byte in 2 GiB of physical memory.

### Virtual Memory

**Virtual memory** is a memory-management mechanism that allows processes to use virtual addresses instead of directly using physical addresses.

It is implemented using:

- CPU
- Memory ManagementUnit
- Operating system
- Page tables
- TLB
- Physical memory
- Secondary storage

Virtual memory provides:

- Process isolation
- Memory protection
- Non-contiguous physical-memory allocation
- Demand paging
- Memory sharing
- A simple address space for programs

Virtual memory is not a separate physical memory device. It is an abstraction created through address translation.

### Virtual Address Space

Each process normally receives its own **virtual address space**.

For example:

```text
Process A: Virtual addresses 0 to 32 GiB − 1
Process B: Virtual addresses 0 to 32 GiB − 1
```

The same virtual address can map to different physical addresses:

| Process | Virtual address | Physical address |
|---|---:|---:|
| Process A | `0x1000` | `0xA000` |
| Process B | `0x1000` | `0xF000` |

This is possible because the processes have different page-table mappings.

It is common to say that each process has its own virtual memory. More precisely:

> The system provides the virtual-memory mechanism, while each process has its own virtual address space and page-table hierarchy.

### Virtual vs Physical Memory Size

A process’s virtual address space can be larger than the installed physical memory.

For example:

```text
Virtual address space per process = 32 GiB
Physical memory                   = 2 GiB
```

This does not mean that the process occupies 32 GiB of RAM. The 32 GiB only represents the range of virtual addresses available to the process.

A virtual page may be:

- Present in physical memory
- Stored in secondary memory
- Not yet allocated
- Shared with another process
- Invalid or unused

Therefore:

```text
Virtual address-space size ≠ Physical memory occupied
```

---

## 3. Process Isolation and Shared Memory

### Process Isolation

One process normally cannot access another process’s private memory.

Suppose Process A and Process B both generate virtual address `0x1000`:

```text
Process A: VA 0x1000 → Process A page table → PA 0xA000
Process B: VA 0x1000 → Process B page table → PA 0xF000
```

When Process A is running, the MMU uses Process A’s page table. It does not use Process B’s page table.

Isolation is provided through:

- Separate page tables
- Page permission bits
- User and kernel privilege levels
- MMU protection checks

### Shared Memory

Different processes can intentionally map virtual pages to the same physical frame:

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

Therefore, processes normally have separate address spaces, but selected pages may intentionally share physical memory.

---

## 4. Paging

**Paging** divides virtual and physical memory into equal-sized fixed blocks:

- A virtual-memory block is called a **page**.
- A physical-memory block is called a **frame**.
- Page size and frame size are equal.

For example:

```text
Page size = Frame size = 2 KiB
```

Consecutive virtual pages do not need to occupy consecutive physical frames:

```text
Virtual page 0 → Physical frame 100
Virtual page 1 → Physical frame 25
Virtual page 2 → Physical frame 700
Virtual page 3 → Physical frame 41
```

The process sees a continuous virtual address space even though the physical frames are scattered throughout RAM.

Paging provides:

- Non-contiguous physical-memory allocation
- Elimination of external fragmentation
- Easier physical-memory allocation
- Per-page protection
- Page sharing
- Demand paging

Paging may cause some **internal fragmentation** in the final allocated page.

For example, if a process requires one byte more than a complete 2 KiB page, another complete 2 KiB page must be allocated.

---

## 5. Address Translation Example

Assume:

```text
Number of processes               = 10
Virtual address space per process = 32 GiB
Physical memory                   = 2 GiB
Page size                         = 2 KiB
Page-table entry size             = 4 bytes
```

### Virtual Address

```text
32 GiB = 2³⁵ bytes
```

Therefore:

```text
Virtual address size = 35 bits
```

### Physical Address

```text
2 GiB = 2³¹ bytes
```

Therefore:

```text
Physical address size = 31 bits
```

### Page Offset

```text
Page size = 2 KiB
          = 2¹¹ bytes
```

Therefore:

```text
Page offset = 11 bits
```

### Virtual Address Format

```text
VPN bits = 35 − 11
         = 24 bits
```

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

### Physical Address Format

```text
PFN bits = 31 − 11
         = 20 bits
```

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

### Translation

The page table replaces the Virtual Page Number with a Physical Frame Number:

```text
Virtual address:  VPN | Offset
                         ↓
Physical address: PFN | Same offset
```

The page offset remains unchanged because pages and frames have the same size.

---

## 6. MMU and Page Table

### Memory Management Unit

The **Memory Management Unit (MMU)** is hardware that translates virtual addresses into physical addresses.

The MMU also performs:

- TLB lookup
- Page-table lookup
- Read/write permission checking
- Execute-permission checking
- User/kernel access checking
- Protection-fault generation

The operating system creates and maintains the page tables, while the MMU uses them during translation.

### Page Table

A **page table** stores mappings from Virtual Page Numbers to Physical Frame Numbers.

| Virtual page number | Physical frame number |
|---:|---:|
| 0 | 500 |
| 1 | 110 |
| 2 | Not present |
| 3 | 800 |

Each process normally has its own page-table hierarchy.

During a process context switch, the operating system changes a special CPU register that points to the new process’s page-table root.

### Page-Table Entry

A **Page-Table Entry (PTE)** generally contains:

- Physical frame number
- Present or valid bit
- Read/write permission
- User/kernel permission
- Execute-disable bit
- Accessed bit
- Dirty bit

Important control bits:

| Bit | Meaning |
|---|---|
| Present | Indicates whether the page is currently in physical memory |
| Read/write | Controls whether the page can be modified |
| Execute | Controls whether instructions can be executed from the page |
| Accessed | Indicates that the page was recently accessed |
| Dirty | Indicates that the page was modified |

A dirty page must normally be written to secondary storage before it is removed from physical memory.

---

## 7. Flat and Multilevel Page Tables

### Flat Page Table

A flat page table contains one PTE for every possible virtual page.

```text
Virtual pages per process = 32 GiB / 2 KiB
                          = 2²⁴ pages
```

Assuming each PTE occupies 4 bytes:

```text
Flat table size = 2²⁴ × 4 bytes
                = 64 MiB per process
```

For 10 processes:

```text
Total size = 10 × 64 MiB
           = 640 MiB
```

The complete table is required even when a process uses only a small part of its virtual address space.

### Multilevel Page Table

A multilevel page table divides the flat table into smaller tables. Lower-level tables are created only for virtual-address regions that are actually mapped.

A 2 KiB page-table page with 4-byte entries contains:

```text
Entries per table = 2 KiB / 4 bytes
                  = 512 entries
                  = 2⁹ entries
```

Therefore, the 24-bit VPN can be divided as:

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

Translation proceeds as follows:

```text
L1 entry → Address of an L2 table
L2 entry → Address of an L3 table
L3 entry → Physical frame number
```

For example:

```text
L1 index = 18
L2 index = 291
L3 index = 138
Offset   = 0x678
```

The page-table walk is:

```text
Page-table root
      ↓ Select L1 entry 18
Selected L2 table
      ↓ Select L2 entry 291
Selected L3 table
      ↓ Select L3 entry 138
Physical frame number
      ↓
Combine PFN with offset 0x678
      ↓
Physical address
```

### Coverage of the Tables

One L3 table contains 512 entries, with each entry mapping one 2 KiB page:

```text
One L3 table maps = 512 × 2 KiB
                  = 1 MiB
```

One L2 table points to 512 L3 tables:

```text
One L2 table covers = 512 × 1 MiB
                    = 512 MiB
```

The L1 table covers the complete 32 GiB virtual address space.

### Minimum Memory Requirement

For the first mapped page:

```text
One L1 table = 2 KiB
One L2 table = 2 KiB
One L3 table = 2 KiB

Minimum total = 6 KiB
```

The same L3 table can map as much as 1 MiB, so 6 KiB is not required for every mapped page.

### Mapping a 2 GiB Region

If one process maps a contiguous 2 GiB region:

```text
Mapped pages = 2 GiB / 2 KiB
             = 2²⁰ pages
```

Each L3 table maps 1 MiB:

```text
L3 tables required = 2 GiB / 1 MiB
                   = 2048

L3 memory = 2048 × 2 KiB
          = 4 MiB
```

Each L2 table covers 512 MiB:

```text
L2 tables required = 2 GiB / 512 MiB
                   = 4

L2 memory = 4 × 2 KiB
          = 8 KiB
```

Including one 2 KiB L1 table:

```text
Total multilevel size = 4 MiB + 8 KiB + 2 KiB
                      = 4 MiB + 10 KiB per process
```

### Comparison

| Page-table organization | Per process | For 10 processes |
|---|---:|---:|
| Flat table | 64 MiB | 640 MiB |
| Multilevel table mapping 2 GiB | 4 MiB + 10 KiB | 40 MiB + 100 KiB |
| Multilevel table with minimum mapping | 6 KiB | 60 KiB |

A multilevel page table saves memory when the virtual address space is sparsely mapped.

---

## 8. Page Fault

A **page fault** occurs when an address translation requires operating-system intervention.

A common case is when the requested page is valid but is not currently present in physical memory.

The sequence is:

1. The CPU generates a virtual address.
2. The MMU searches the TLB.
3. On a TLB miss, the page table is checked.
4. The PTE indicates `Present = 0`.
5. The CPU raises a page-fault exception.
6. The OS checks whether the virtual address is valid.
7. The OS obtains a free frame or evicts another page.
8. A dirty evicted page is written to secondary storage.
9. The required page is loaded from secondary storage.
10. The PTE and TLB are updated.
11. The interrupted instruction is restarted.

```text
Page not present
      ↓
Page fault
      ↓
OS validates the address
      ↓
Find a free frame or evict another page
      ↓
Load the page from secondary storage
      ↓
Update the PTE and TLB
      ↓
Restart the instruction
```

Loading a page only when it is required is called **demand paging**.

A page fault is not always an error:

| Situation | OS action |
|---|---|
| Valid page stored on disk | Load it into RAM |
| Valid page not yet allocated | Allocate a new page |
| Invalid virtual address | Report an access violation |
| Write to a read-only page | Report a protection fault |

---

## 9. Translation Lookaside Buffer

The **Translation Lookaside Buffer (TLB)** is a small and fast hardware cache that stores recently used address translations.

A TLB entry stores:

```text
Virtual Page Number → Physical Frame Number + Permissions
```

Without a TLB, a three-level page table may require three memory accesses for translation before the actual data is accessed.

### TLB Hit

A TLB hit occurs when the translation is already present:

```text
Virtual address
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

### TLB Miss

A TLB miss occurs when the translation is absent:

```text
Virtual address
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

### TLB Miss vs Page Fault

| Situation | Result |
|---|---|
| Translation is present in TLB | TLB hit |
| Translation is absent, but page is in RAM | TLB miss and page-table walk |
| PTE indicates that the page is absent | Page fault |
| PTE denies the requested operation | Protection fault |

Therefore:

> A TLB miss does not necessarily cause a page fault.

---

## 10. Complete Memory-Access Flow

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

The complete path can be summarized as:

```text
Virtual address
      ↓
TLB or page table
      ↓
Physical address
      ↓
Cache or physical memory
```


