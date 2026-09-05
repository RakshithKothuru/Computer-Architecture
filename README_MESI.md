# Cache Coherency and MESI Protocol

## Index

1. [Need for Cache Coherency](#1-need-for-cache-coherency)
2. [Cache Coherency Requirements](#2-cache-coherency-requirements)
3. [Write-Invalidate and Write-Update](#3-write-invalidate-and-write-update)
4. [Snooping and Directory-Based Coherence](#4-snooping-and-directory-based-coherence)
5. [MESI Protocol](#5-mesi-protocol)
6. [MESI State Transitions](#6-mesi-state-transitions)
7. [Complete MESI Example](#7-complete-mesi-example)
8. [False Sharing](#8-false-sharing)
9. [Cache Coherency vs Memory Consistency](#9-cache-coherency-vs-memory-consistency)
10. [Summary](#10-summary)

---

## 1. Need for Cache Coherency

In a multicore processor, each core generally has a **private cache**, while the main memory is shared.

Consider a variable `X` whose initial value is `10`.

```text
Core 0 Cache        Core 1 Cache
X = 10              X = 10
       \            /
          Memory
          X = 10
```

If Core 0 changes `X` to `20` only in its cache:

```text
Core 0 Cache        Core 1 Cache
X = 20              X = 10  ← Stale copy

Memory
X = 10
```

Core 1 may continue reading the old value. Therefore, a mechanism is required to coordinate the copies stored in different caches.

> **Cache coherency ensures that multiple cores do not continue using inconsistent cached copies of the same memory location.**

Cache coherency is normally maintained at the granularity of a **cache line**, not an individual byte or variable.

---

## 2. Cache Coherency Requirements

A coherent memory system must provide two main properties.

### Write Propagation

A write performed by one core must eventually become visible to the other cores.

If Core 0 writes `X = 20`, Core 1 must not continue observing `X = 10` indefinitely.

### Write Serialization

All cores must observe writes to the same memory location in the same order.

For example:

```text
Core 0 writes X = 20
Core 1 writes X = 30
```

All other cores must agree on the order of these writes.

---

## 3. Write-Invalidate and Write-Update

Two basic approaches can be used to maintain cache coherence.

### Write-Invalidate

Before modifying a shared cache line, the writing core invalidates all other cached copies.

```text
Initially:

Core 0: X = 10, Valid
Core 1: X = 10, Valid

Core 0 writes X = 20:

Core 0: X = 20, Valid
Core 1: X = 10, Invalid
```

When Core 1 accesses `X` again, it experiences a cache miss and obtains the latest value.

Write-invalidate is commonly used because after obtaining ownership, a core can perform multiple writes without repeatedly communicating with the other caches.

### Write-Update

Whenever a core modifies a cache line, the new value is sent to every other cache holding that line.

```text
Core 0 writes X = 20
          ↓
Core 1 copy is updated to X = 20
```

This keeps all copies updated but may generate large amounts of coherence traffic when writes occur frequently.

> MESI is primarily a **write-invalidate protocol**.

---

## 4. Snooping and Directory-Based Coherence

### Snooping-Based Coherence

In a snooping system, every cache controller monitors, or **snoops**, coherence transactions on a shared interconnect.

When one core requests or modifies a cache line:

1. It broadcasts the request.
2. Other caches snoop the request.
3. Caches containing the requested line respond or change their states.
4. The requesting core obtains the data or permission to modify it.

```text
Core 0 ─┐
Core 1 ─┼── Shared Interconnect ── Main Memory
Core 2 ─┘
```

Snooping is relatively simple and effective for systems with a small number of cores. However, broadcast traffic becomes expensive as the number of cores increases.

### Directory-Based Coherence

A **directory** maintains information about which caches contain each memory block.

```text
Block X:
Sharers = {Core 0, Core 2}
Owner   = Core 0
```

If Core 1 wants to write Block `X`:

1. Core 1 sends an ownership request to the directory.
2. The directory identifies the caches holding the block.
3. Invalidations are sent only to Core 0 and Core 2.
4. These caches invalidate their copies and acknowledge.
5. Core 1 receives exclusive ownership.

Directory-based coherence scales better than broadcast snooping but requires additional storage and more complex control logic.

---

## 5. MESI Protocol

MESI is a write-invalidate cache-coherence protocol. Every cache line is assigned one of four states:

- **M — Modified**
- **E — Exclusive**
- **S — Shared**
- **I — Invalid**

### Modified State

A cache line in the Modified state:

- Is valid and present in only one cache.
- Contains the latest data.
- Is different from the copy in main memory.
- Can be read or modified locally without a coherence transaction.

```text
Core 0: X = 20, M
Core 1: X, I
Memory: X = 10
```

Since memory is stale, the modified data must be written back or transferred if another core requests it.

### Exclusive State

A cache line in the Exclusive state:

- Is valid and present in only one cache.
- Contains the same data as main memory.
- Can be read locally.
- Can transition directly from `E` to `M` when written.

```text
Core 0: X = 10, E
Core 1: X, I
Memory: X = 10
```

The `E` state is useful because the core can modify the block without sending invalidations, as no other cache holds a copy.

### Shared State

A cache line in the Shared state:

- Is valid.
- May exist in multiple caches.
- Contains the same data as main memory.
- Can be read locally.
- Cannot be modified until the other copies are invalidated.

```text
Core 0: X = 10, S
Core 1: X = 10, S
Memory: X = 10
```

If Core 0 wants to write, it must first obtain exclusive ownership by invalidating Core 1's copy.

### Invalid State

A cache line in the Invalid state does not contain valid data.

```text
Core 0: X, I
```

Any processor read or write to this line results in a cache miss or ownership request.

### MESI State Summary

| State | Valid? | Other caches may have a copy? | Same as memory? | Can write directly? |
|---|---:|---:|---:|---:|
| **Modified** | Yes | No | No | Yes |
| **Exclusive** | Yes | No | Yes | Yes, followed by `E → M` |
| **Shared** | Yes | Yes | Yes | No, invalidate others first |
| **Invalid** | No | Possibly | Not relevant | No, obtain the line first |

---

## 6. MESI State Transitions

MESI transitions are caused by:

- Processor reads
- Processor writes
- Snooped read requests from other cores
- Snooped ownership or invalidation requests

### Processor Read Hit

If the line is in the `M`, `E`, or `S` state, the cache supplies the data locally.

```text
M → M
E → E
S → S
```

No state transition is normally required.

---

### Processor Read Miss

A read miss occurs when the local cache line is in the `I` state.

The core sends a read request, commonly called `BusRd`.

#### No Other Cache Has the Line

The data is fetched from memory. Since the requesting cache is the only cache holding it:

```text
Requesting cache: I → E
```

#### Another Cache Has the Line in Exclusive State

The other cache no longer has the only copy.

```text
Other cache:      E → S
Requesting cache: I → S
```

#### Another Cache Has the Line in Shared State

The requesting cache receives a shared copy.

```text
Other cache:      S → S
Requesting cache: I → S
```

#### Another Cache Has the Line in Modified State

The modified cache holds the latest data, while memory may contain an old value. It must supply or write back the latest value.

```text
Other cache:      M → S
Requesting cache: I → S
```

Both caches now hold shared copies. The exact data-transfer path depends on the hardware implementation.

---

### Processor Write Hit

#### Write in Modified State

The cache already has exclusive ownership.

```text
M → M
```

The data is updated locally without additional coherence traffic.

#### Write in Exclusive State

No other cache has a copy, so the cache can modify the data locally.

```text
E → M
```

No invalidation request is required.

#### Write in Shared State

Other caches may contain copies. The writing cache sends an upgrade or invalidate request.

```text
Writing cache: S → M
Other caches:  S → I
```

The writing cache does not need to fetch the block again because it already has the data. It only needs ownership permission.

---

### Processor Write Miss

A write miss occurs when the local cache line is in the `I` state.

The processor sends a **Read for Ownership**, commonly called `BusRdX` or `RFO`.

This request:

1. Obtains the cache line.
2. Invalidates all copies in the other caches.
3. Gives the requesting core exclusive write permission.

```text
Requesting cache: I → M
Other caches:     M/E/S → I
```

If another cache holds the line in the `M` state, it must supply or write back the latest data before invalidating its copy.

---

### Common MESI Transitions

| Current state | Event | Next state | Required action |
|---|---|---|---|
| `I` | Processor read, no other copy | `E` | Fetch from memory |
| `I` | Processor read, another copy exists | `S` | Fetch shared data |
| `I` | Processor write | `M` | Read for ownership |
| `E` | Processor read | `E` | Read locally |
| `E` | Processor write | `M` | Modify locally |
| `E` | Another core reads | `S` | Allow shared copy |
| `E` | Another core requests ownership | `I` | Invalidate local copy |
| `S` | Processor read | `S` | Read locally |
| `S` | Processor write | `M` | Invalidate other copies |
| `S` | Another core requests ownership | `I` | Invalidate local copy |
| `M` | Processor read or write | `M` | Access locally |
| `M` | Another core reads | `S` | Supply or write back data |
| `M` | Another core requests ownership | `I` | Supply data and invalidate |

---

## 7. Complete MESI Example

Assume `X = 10` in main memory, and both cache lines are initially invalid.

### Step 1: Core 0 Reads X

No other cache contains `X`, so Core 0 receives the line in the Exclusive state.

```text
Core 0: X = 10, E
Core 1: X, I
Memory: X = 10
```

Transition:

```text
Core 0: I → E
```

### Step 2: Core 1 Reads X

Core 0 snoops the request. Since another cache now receives a copy:

```text
Core 0: E → S
Core 1: I → S
```

Result:

```text
Core 0: X = 10, S
Core 1: X = 10, S
Memory: X = 10
```

### Step 3: Core 0 Writes X = 20

Core 0 already has the data but must obtain exclusive ownership. It sends an invalidation request because Core 1 has a shared copy.

```text
Core 0: S → M
Core 1: S → I
```

Result:

```text
Core 0: X = 20, M
Core 1: X, I
Memory: X = 10
```

Core 0 contains the latest value, while memory is temporarily stale.

### Step 4: Core 1 Reads X

Core 1 experiences a read miss. Core 0 supplies or writes back the modified value.

```text
Core 0: M → S
Core 1: I → S
```

Result:

```text
Core 0: X = 20, S
Core 1: X = 20, S
Memory: X = 20
```

Both cores now observe the latest value.

### Step 5: Core 1 Writes X = 30

Core 1 invalidates the copy in Core 0 and obtains exclusive ownership.

```text
Core 0: S → I
Core 1: S → M
```

Result:

```text
Core 0: X, I
Core 1: X = 30, M
Memory: X = 20
```

The most recent copy of `X` is now present only in Core 1's cache.

---

## 8. False Sharing

Cache coherence operates on an entire cache line.

Suppose two different variables, `A` and `B`, are stored in the same cache line:

```text
Cache line: [ A | B ]
```

If Core 0 repeatedly writes `A` and Core 1 repeatedly writes `B`, the entire cache line keeps moving between their caches.

```text
Core 0 writes A → Core 1's cache line is invalidated
Core 1 writes B → Core 0's cache line is invalidated
```

Although the cores access different variables, they generate coherence traffic because the variables occupy the same cache line.

This is called **false sharing**.

False sharing does not produce incorrect results, but it can severely reduce performance. It can be reduced by placing frequently modified variables in separate cache lines using padding or alignment.

---

## 9. Cache Coherency vs Memory Consistency

These concepts are related but different.

### Cache Coherency

Cache coherency controls how different cores observe accesses to the **same memory location**.

```text
How do all cores observe writes to X?
```

### Memory Consistency

Memory consistency defines the order in which accesses to **different memory locations** may become visible.

```text
In what order are writes to X and Y observed?
```

A processor can have coherent caches while still allowing memory operations to different addresses to be reordered.

---

## 10. Summary

- Private caches may contain multiple copies of the same memory block.
- Cache coherency prevents cores from continuously using stale copies.
- Write propagation and write serialization are the main coherence requirements.
- Write-invalidate is more commonly used than write-update.
- MESI is a write-invalidate protocol operating at the cache-line level.
- `M` means modified and exclusively owned.
- `E` means clean and exclusively owned.
- `S` means clean and possibly present in multiple caches.
- `I` means invalid.
- `E → M` requires no coherence transaction.
- `S → M` requires invalidating the other shared copies.
- `I → M` requires a Read for Ownership.
- Snooping is suitable for smaller systems.
- Directory-based coherence scales better for larger systems.
- False sharing occurs when independent variables in the same cache line cause unnecessary invalidations.
- Cache coherency and memory consistency are different concepts.
