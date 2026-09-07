# Multicore Processors

## Index

1. [Multiprocessor Basics](#1-multiprocessor-basics)
2. [Cache Coherence](#2-cache-coherence)
3. [Coherence Protocols](#3-coherence-protocols)
4. [MESI Protocol](#4-mesi-protocol)

---

# 1. Multiprocessor Basics

## What Is a Multicore Processor?

A **multicore processor** contains two or more processing cores on the same chip. Each core can execute its own instruction stream, allowing multiple tasks or threads to run simultaneously.

A typical multicore system contains:

- Multiple processor cores
- Private registers and execution units for each core
- Private or shared caches
- Shared main memory
- An interconnect connecting the cores, caches and memory

## Why Are Multicore Processors Used?

Earlier processors improved performance mainly by increasing clock frequency. However, increasing frequency causes:

- Higher power consumption
- Greater heat generation
- Increased leakage power
- More difficult timing closure
- Diminishing benefits from instruction-level parallelism

Multicore processors improve performance by executing multiple threads simultaneously without requiring a large increase in clock frequency.

The actual performance improvement depends on how much of the program can execute in parallel.

## Shared-Memory Multiprocessor

In a shared-memory multiprocessor, all processors access a common address space.

If Core 0 and Core 1 access address `0x1000`, both refer to the same physical memory location.

```text
Core 0 → Private Cache ┐
                       ├── Shared Memory
Core 1 → Private Cache ┘
```

Each core may store a separate cached copy of the same memory location. This creates the **cache-coherence problem**.

## Distributed-Memory Multiprocessor

In a distributed-memory system:

- Each processor has its own local memory.
- A processor cannot directly access another processor’s local memory.
- Processors communicate by exchanging messages.

| Shared-Memory System | Distributed-Memory System |
|---|---|
| Common address space | Separate address spaces |
| Communication through shared variables | Communication through messages |
| Easier to program | Generally more scalable |
| Requires cache coherence | No coherence needed between private memories |

## Symmetric Multiprocessing

In a **Symmetric Multiprocessing (SMP)** system:

- All processors are treated equally.
- All processors execute the same operating system.
- All processors can access the same main memory and I/O devices.
- Any process or thread can generally execute on any processor.

```text
Core 0 ─┐
Core 1 ─┼── Shared Memory
Core 2 ─┤
Core 3 ─┘
```

Most small and medium shared-memory multicore systems use an SMP-style organization.

> SMP describes how processors are treated, whereas UMA and NUMA describe memory-access latency.

## Uniform Memory Access

In a **Uniform Memory Access (UMA)** system, every processor experiences approximately the same latency when accessing any location in main memory.

```text
Core 0 ─┐
Core 1 ─┼── Shared Interconnect ── Shared Memory
Core 2 ─┤
Core 3 ─┘
```

### Advantages

- Simple memory organization
- Easier operating-system scheduling
- Predictable memory-access latency

### Limitations

- Shared memory may become a bottleneck.
- The shared interconnect may become congested.
- Scalability is limited as the number of cores increases.

## Non-Uniform Memory Access

In a **Non-Uniform Memory Access (NUMA)** system, memory is physically distributed among processor nodes, but the system provides a shared address space.

```text
Node 0: Cores + Local Memory
Node 1: Cores + Local Memory
Node 2: Cores + Local Memory
```

A core can access:

- **Local memory:** Lower access latency
- **Remote memory:** Higher access latency through the interconnect

NUMA improves scalability because memory bandwidth is distributed across multiple nodes. However, performance depends on placing data close to the cores that frequently access it.

A cache-coherent NUMA system is called **ccNUMA**.

## UMA vs NUMA

| Feature | UMA | NUMA |
|---|---|---|
| Memory latency | Approximately uniform | Depends on memory location |
| Physical memory | Usually centralized | Distributed among nodes |
| Address space | Shared | Shared |
| Scalability | Lower | Higher |
| Programming complexity | Lower | Higher |
| Importance of data placement | Lower | High |

## Typical Multicore Cache Hierarchy

A multicore processor may contain:

- Private L1 instruction cache for each core
- Private L1 data cache for each core
- Private or shared L2 cache
- Shared Last-Level Cache (LLC)
- Shared main memory

```text
Core 0 ── Private L1 ─┐
                      ├── Shared LLC ── Main Memory
Core 1 ── Private L1 ─┘
```

### Private Caches

Advantages:

- Lower access latency
- Less contention
- Higher bandwidth for each core

Disadvantages:

- Multiple caches may store copies of the same memory block.
- A cache-coherence protocol is required.

### Shared Caches

Advantages:

- Easier data sharing
- Less duplication of shared data
- Flexible distribution of cache capacity

Disadvantages:

- Higher access latency
- Contention between cores

---

# 2. Cache Coherence

## Why Is Cache Coherence Required?

Suppose two cores initially read the same variable:

```text
Memory:       X = 10

Core 0 cache: X = 10
Core 1 cache: X = 10
```

Now Core 0 changes `X` to `20`:

```text
Core 0 cache: X = 20
Core 1 cache: X = 10
```

Core 1 now contains a **stale copy**. If it reads `X`, it may receive the old value instead of the latest value.

Cache coherence ensures that processors maintain a consistent view of each shared memory location.

## Coherence Is Maintained per Cache Line

Coherence protocols normally track data at the granularity of a **cache line**, not an individual byte or variable.

For example, if the cache-line size is 64 bytes, writing one variable can affect the coherence state of the entire 64-byte line.

This is also the reason **false sharing** can occur.

## Properties of Cache Coherence

A coherent system should provide two important properties:

1. Write propagation
2. Write serialization

### Write Propagation

When one processor writes to a memory location, the new value must eventually become visible to other processors.

For example:

```text
Core 0 writes X = 20
```

A later read of `X` by Core 1 must eventually return `20`, rather than an indefinitely stale value.

### Write Serialization

All processors must observe writes to the same memory location in the same order.

Suppose:

```text
Core 0 writes X = 10
Core 1 writes X = 20
```

All processors must agree on one global order:

```text
X = 10 followed by X = 20
```

or:

```text
X = 20 followed by X = 10
```

Different processors must not observe conflicting orders for writes to the same memory location.

## Cache Coherence vs Memory Consistency

Cache coherence and memory consistency are related but different concepts.

- **Cache coherence** controls the ordering and visibility of accesses to the same memory location.
- **Memory consistency** controls the ordering and visibility of accesses to different memory locations.

Consider:

```text
Core 0:
X = 1
Y = 1

Core 1:
Read Y
Read X
```

Coherence ensures that writes to `X` are consistently ordered and writes to `Y` are consistently ordered.

However, coherence alone does not guarantee that Core 1 observes the write to `X` before the write to `Y`. That behavior is defined by the system’s memory-consistency model.

## Write-Invalidate Protocol

Before a core writes to a shared cache line, all other cached copies are invalidated.

```text
Initial state:

Core 0: X = 10
Core 1: X = 10
```

When Core 0 writes `X = 20`:

```text
Core 0: X = 20
Core 1: Invalid
```

Protocols such as MSI and MESI use write invalidation.

### Advantages

- Multiple writes by the same core do not generate repeated updates.
- It usually produces less interconnect traffic.

## Write-Update Protocol

Whenever a core writes a new value, the updated value is sent to all other caches that hold the line.

```text
Core 0 writes X = 20

Core 0: X = 20
Core 1: X = 20
```

### Advantages

- Other caches retain an updated copy.
- A later read by another core may be a cache hit.

### Disadvantages

- Frequent writes generate a large amount of interconnect traffic.

Modern processors generally prefer **write-invalidate protocols**.

---

# 3. Coherence Protocols

## Snooping-Based Coherence

In snooping systems, coherence transactions are broadcast over a shared interconnect.

Every cache monitors, or **snoops**, the transactions generated by other caches and changes the state of its local cache lines when necessary.

### Advantages

- Relatively simple
- Fast for a small number of cores

### Limitation

Broadcasting every coherence transaction to every cache does not scale efficiently to a large number of cores.

## Directory-Based Coherence

A directory stores information about which caches currently hold each memory block.

Instead of broadcasting a request to every cache, the system sends messages only to the caches that hold the requested block.

### Advantages

- More scalable
- Avoids system-wide broadcasts
- Suitable for large multicore systems

### Limitations

- Requires additional directory storage
- More complex than snooping
- Directory lookup may add latency

## MSI Protocol

MSI uses three states:

- **Modified:** Only this cache has the line, and the line is newer than memory.
- **Shared:** One or more caches may have the line, and memory is up to date.
- **Invalid:** The cached copy cannot be used.

The limitation of MSI is that even when only one cache contains a clean line, the line is placed in the `Shared` state.

Therefore, the first write requires an additional coherence transaction:

```text
Shared → Modified
```

MESI reduces this transaction by adding the `Exclusive` state.

## MOESI Protocol

MOESI extends MESI by adding the **Owned** state.

A cache holding a line in the Owned state:

- Contains the latest modified value
- May share the line with other caches
- Is responsible for supplying the latest data
- Does not have to immediately update main memory

Therefore, memory may remain stale while the owner cache supplies the latest data to other caches.

---

# 4. MESI Protocol

MESI is a **write-invalidate cache-coherence protocol**.

Every cache line can exist in one of four states:

1. Modified
2. Exclusive
3. Shared
4. Invalid

## MESI States

### Modified State

In the **Modified (M)** state:

- The cache contains the only valid copy.
- The line has been modified by the local processor.
- Main memory contains an old value.
- No other cache may contain a valid copy.
- The line must be written back before it is evicted or transferred.

```text
Core 0 cache: X = 20
Memory:       X = 10
```

Here, the latest value exists only in Core 0’s cache.

### Exclusive State

In the **Exclusive (E)** state:

- The cache contains the only cached copy.
- The line has not been modified.
- Main memory is up to date.
- No other cache contains the line.

```text
Core 0 cache: X = 10
Memory:       X = 10
```

A processor can write to an Exclusive line without generating an invalidation transaction:

```text
E → M
```

This is called a **silent transition**.

### Shared State

In the **Shared (S)** state:

- Multiple caches may contain the line.
- The cache line is clean.
- Main memory is up to date.
- A processor must invalidate other copies before modifying the line.

```text
Core 0 cache: X = 10
Core 1 cache: X = 10
Memory:       X = 10
```

### Invalid State

In the **Invalid (I)** state:

- The cache line does not contain valid data.
- The processor cannot directly use the cached copy.
- The cache must obtain the line before reading or writing it.

## MESI State Summary

| State | Valid? | Modified? | Other caches may contain it? | Memory up to date? |
|---|---:|---:|---:|---:|
| Modified | Yes | Yes | No | No |
| Exclusive | Yes | No | No | Yes |
| Shared | Yes | No | Yes | Yes |
| Invalid | No | Not applicable | Yes or no | Not applicable |

## Important Coherence Transactions

The exact names vary between implementations, but the following conceptual transactions are commonly used.

### BusRd

A cache issues `BusRd` when the processor wants to read a cache line that is currently Invalid.

- If no other cache has the line, it is loaded in `Exclusive`.
- If another cache has the line, it is loaded in `Shared`.

### BusRdX

A cache issues `BusRdX` when it wants to write a line but does not currently have a valid copy.

`BusRdX` performs two operations:

1. Fetches the cache line
2. Invalidates all other cached copies

The requesting cache obtains the line in the `Modified` state.

`BusRdX` is also called **Read For Ownership (RFO)**.

### BusUpgr

A cache issues `BusUpgr` when it already contains the line in the `Shared` state and wants to write it.

- Other shared copies are invalidated.
- The requesting cache does not fetch the data again.
- The requesting cache changes from `Shared` to `Modified`.

```text
Requesting cache: S → M
Other caches:     S → I
```

### Flush or Write-Back

A cache in the `Modified` state contains the latest value.

It must provide this value when:

- Another processor requests the line
- The modified line is evicted

Depending on the implementation, the data may be sent to:

- Main memory
- The requesting cache
- Both the requesting cache and main memory

## Processor-Initiated State Transitions

These transitions occur because of a read or write by the local processor.

### Processor Read Hit

If the requested line is already in `M`, `E` or `S`, the read is a cache hit.

```text
M → M
E → E
S → S
```

No coherence transaction is required.

### Processor Read Miss

If the line is in `Invalid`, the cache generates `BusRd`.

#### No Other Cache Has the Line

```text
I → E
```

The requesting cache receives the line in the Exclusive state.

#### Another Cache Has the Line

```text
I → S
```

The requesting cache and existing caches hold the line in the Shared state.

### Processor Write Hit in Modified

The processor already has exclusive ownership of the modified line:

```text
M → M
```

No coherence transaction is required.

### Processor Write Hit in Exclusive

No other cache has a valid copy:

```text
E → M
```

No invalidation transaction is required.

### Processor Write Hit in Shared

Other caches may contain copies of the line. Therefore, the cache generates `BusUpgr`.

```text
Requesting cache: S → M
Other caches:     S → I
```

### Processor Write Miss in Invalid

The cache generates `BusRdX`.

```text
Requesting cache: I → M
Other valid copies → I
```

The requesting cache obtains both the cache line and exclusive ownership.

## Snooped State Transitions

A cache must monitor transactions generated by other processors.

### Line in Modified

If another core generates `BusRd`:

```text
M → S
```

The Modified cache supplies or writes back the latest data. The requesting cache receives the line in `Shared`.

If another core generates `BusRdX`:

```text
M → I
```

The Modified cache supplies or writes back the latest data, and the requesting cache obtains ownership.

### Line in Exclusive

If another core generates `BusRd`:

```text
E → S
```

Both caches now contain clean shared copies.

If another core generates `BusRdX`:

```text
E → I
```

The requesting cache obtains the line in `Modified`.

### Line in Shared

If another core generates `BusRd`:

```text
S → S
```

The line remains Shared.

If another core generates `BusRdX` or `BusUpgr`:

```text
S → I
```

The local shared copy is invalidated.

## MESI Transition Table

| Current state | Event | Transaction | Next state |
|---|---|---|---|
| I | Processor read; no sharer exists | `BusRd` | E |
| I | Processor read; sharer exists | `BusRd` | S |
| I | Processor write | `BusRdX` | M |
| S | Processor read | None | S |
| S | Processor write | `BusUpgr` | M |
| E | Processor read | None | E |
| E | Processor write | None | M |
| M | Processor read or write | None | M |
| M | Snooped `BusRd` | Flush or supply data | S |
| M | Snooped `BusRdX` | Flush or supply data | I |
| E | Snooped `BusRd` | None | S |
| E | Snooped `BusRdX` | None | I |
| S | Snooped `BusRd` | None | S |
| S | Snooped `BusRdX` or `BusUpgr` | Invalidate | I |

## MESI Example

Initially:

```text
Memory: X = 10

Core 0: I
Core 1: I
```

### Step 1: Core 0 Reads X

Core 0 generates `BusRd`.

No other cache has `X`, so Core 0 receives the line in the Exclusive state.

```text
Core 0: E, X = 10
Core 1: I
Memory: X = 10
```

### Step 2: Core 1 Reads X

Core 1 generates `BusRd`. Core 0 snoops the request.

```text
Core 0: E → S
Core 1: I → S
Memory: X = 10
```

Both caches now contain clean shared copies.

### Step 3: Core 0 Writes X = 20

Core 0 has the line in the Shared state, so it generates `BusUpgr`.

```text
Core 0: S → M, X = 20
Core 1: S → I
Memory: X = 10
```

Memory is stale because the latest value exists only in Core 0’s Modified line.

### Step 4: Core 1 Reads X

Core 1 has the line in the Invalid state, so it generates `BusRd`.

Core 0 has the latest value in the Modified state. Therefore, Core 0 supplies or writes back the latest data.

```text
Core 0: M → S
Core 1: I → S
Memory: X = 20
```

Both cores now observe the latest value.

## Why Is the Exclusive State Useful?

Suppose a processor reads a cache line that no other cache contains.

In MSI:

```text
I → S
```

A later write requires an upgrade transaction:

```text
S → M
```

In MESI:

```text
I → E
```

A later write uses a silent transition:

```text
E → M
```

Therefore, the Exclusive state reduces coherence traffic when a cache line is accessed by only one core.
