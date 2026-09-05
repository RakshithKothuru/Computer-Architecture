# Cache Memory

## Index

1. [Cache Basics](#1-cache-basics)
2. [Cache Organization](#2-cache-organization)
3. [Cache Design Questions](#3-cache-design-questions)
4. [Mapping Techniques](#4-mapping-techniques)
5. [Block Replacement](#5-block-replacement)
6. [Write Strategies](#6-write-strategies)
7. [Cache Performance](#7-cache-performance)
8. [Types of Cache Misses](#8-types-of-cache-misses)
9. [Basic Cache Optimizations](#9-basic-cache-optimizations)
10. [Final Summary](#10-final-summary)

---

# 1. Cache Basics

The processor is much faster than main memory. If every instruction and operand is accessed directly from main memory, the processor spends many cycles waiting.

Cache is a small and fast memory placed between the processor and main memory.

```text
Processor → L1 Cache → L2 Cache → L3 Cache → Main Memory
Fast, small                                  Slow, large
```

When the processor requests data:

- If it is present in the cache, it is a **cache hit**.
- If it is absent, it is a **cache miss**.
- On a miss, a complete block is fetched from the next memory level.

## Locality of Reference

Cache works because programs exhibit locality.

### Temporal Locality

Recently accessed data is likely to be accessed again.

```c
for (i = 0; i < 100; i++)
    sum += A[i];
```

Variables such as `i` and `sum` are accessed repeatedly.

### Spatial Locality

If one address is accessed, nearby addresses are likely to be accessed soon.

After accessing `A[0]`, a program will probably access `A[1]`, `A[2]`, and so on. Therefore, cache transfers a complete block rather than only one word.

---

# 2. Cache Organization

## Memory Block

Main memory is divided into equal-sized blocks.

If each block contains four words:

```text
Block 0 → Words 0–3
Block 1 → Words 4–7
Block 2 → Words 8–11
```

## Cache Line

A cache is divided into lines or entries of the same size as a memory block.

A **cache line** is a location in the cache capable of storing one memory block.

A typical cache line contains:

```text
| Valid Bit | Dirty Bit | Tag | Data Block |
```

- **Data block:** Actual instructions or data
- **Tag:** Identifies the memory block stored in the line
- **Valid bit:** Indicates whether the cache line contains valid data
- **Dirty bit:** Indicates whether the block has been modified

## Cache Address Fields

A memory address is divided into:

```text
| Tag | Index/Set Number | Block Offset |
```

### Block Offset

The block offset selects the required byte or word within the cache block.

For a block size of `B` bytes:

```text
Number of block-offset bits = log₂(B)
```

### Index

The index selects a cache line or set.

```text
Number of index bits = log₂(Number of sets)
```

### Tag

The tag identifies which memory block is stored in the selected cache line or set.

```text
Tag bits = Address bits − Index bits − Block-offset bits
```

## Number of Cache Lines

```text
Number of cache lines = Cache size / Block size
```

## Number of Sets

For an `N`-way set-associative cache:

```text
Number of sets = Cache size / (Block size × Number of ways)
```

---

# 3. Cache Design Questions

A cache design must answer four fundamental questions:

1. **Block placement:** Where can a memory block be placed?
2. **Block identification:** How is the requested block located?
3. **Block replacement:** Which existing block should be removed?
4. **Write strategy:** What happens when the processor writes?

---

# 4. Mapping Techniques (Block Placement and Identification)

Cache mapping determines:

- **Block placement:** The set in which a memory block can be placed.
- **Block identification:** Whether the requested block is present in that set.

For all mapping techniques:

```text
Set index = Memory block number mod Number of sets
```

When we divide the memory block number by the number of sets:

- **Remainder = Set index**
- **Quotient contributes to the tag**

In actual hardware, division is unnecessary because the number of sets is normally a power of two. The required lower bits of the block number directly form the set index.

---

## 4.1 Direct-Mapped Cache

A direct-mapped cache has only one way in each set. Therefore, every memory block has exactly one possible cache location.

```text
Number of sets = Number of cache lines
```

### Block Placement

```text
Set index = Memory block number mod Number of sets
```

Suppose the cache contains eight sets and the requested memory block is 26:

```text
26 ÷ 8 = 3 remainder 2
```

Therefore:

```text
Set index = 2
Tag       = 3
```

Blocks such as `2`, `10`, `18` and `26` all have the same set index and therefore map to Set 2.

### Block Identification

1. Use the set-index bits to select the required set.
2. Because the set contains only one way, read its tag and valid bit.
3. Compare the requested tag with the stored tag.
4. If the tag matches and `Valid = 1`, it is a cache hit.
5. Use the block offset to select the required byte or word.

```text
Hit = Tag match AND Valid bit
```

```text
Address
   ↓
Select set using set index
   ↓
Compare the single stored tag
   ↓
Tag match AND Valid = 1?
   ↓
Use block offset to select data
```

---

## 4.2 Set-Associative Cache

In an `N`-way set-associative cache, every set contains `N` ways.

A memory block maps to exactly one set but can be placed in any way of that set.

```text
Number of sets =
Cache size / (Block size × Number of ways)
```

### Block Placement

```text
Set index = Memory block number mod Number of sets
```

Suppose the cache has four sets and the requested memory block is 26:

```text
26 ÷ 4 = 6 remainder 2
```

Therefore:

```text
Set index = 2
Tag       = 6
```

Block 26 must be placed in Set 2, but it can occupy any available way within Set 2.

```text
Set 2:
Way 0 → Valid | Tag | Data block
Way 1 → Valid | Tag | Data block
```

### Block Identification

1. Use the set-index bits to select the required set.
2. Access all the ways in that set.
3. Compare the requested tag with the stored tag of every way.
4. Check the valid bit of each way.
5. If any way has a matching tag and `Valid = 1`, it is a cache hit.
6. Select the matching way.
7. Use the block offset to select the required byte or word.

For every way:

```text
WayHit[i] = TagMatch[i] AND Valid[i]
```

```text
Address
   ↓
Select set using set index
   ↓
Compare tags of all ways in that set
   ↓
Find a way with Tag match AND Valid = 1
   ↓
Select the matching way
   ↓
Use block offset to select data
```

The important difference is:

- **Direct mapped:** One tag is checked because the selected set has one way.
- **Set associative:** All tags in the selected set are checked because the set has multiple ways.

---

## 4.3 Fully Associative Cache

A fully associative cache has only one set containing all cache lines.

Therefore:

```text
Number of sets = 1
```

Every memory block maps to this single set and can be placed in any cache line.

Since there is only one set:

```text
Set index = Memory block number mod 1 = 0
```

Therefore, no set-index bits are required.

### Block Identification

1. Compare the requested tag with the tags of all cache lines.
2. Check the corresponding valid bits.
3. If any line has a matching tag and `Valid = 1`, it is a hit.
4. Select the matching line.
5. Use the block offset to select the required data.

```text
Address
   ↓
Compare tag with every cache line
   ↓
Find Tag match AND Valid = 1
   ↓
Use block offset to select data
```

---

## Mapping Summary

| Mapping | Number of sets | Block placement | Tags checked |
|---|---:|---|---:|
| Direct mapped | Number of cache lines | One set, one way | One |
| Set associative | Cache lines ÷ ways | One set, any way | All ways in selected set |
| Fully associative | One | Any line in the cache | All cache lines |

> In direct and set-associative caches, dividing the memory block number by the number of sets gives the **set index as the remainder**. In hardware, the set-index bits are directly extracted from the address rather than performing division.

---

# 5. Block Replacement

Block replacement answers:

> When all possible locations are occupied, which existing block should be removed?

Replacement is required when:

- A cache miss occurs.
- All ways in the selected set are valid.

An invalid way should always be used before replacing a valid block.

A direct-mapped cache does not require a replacement algorithm because the destination line is fixed.

---

## 5.1 Random Replacement

A block is selected randomly for replacement.

### Advantages

- Simple implementation
- Very low hardware overhead

### Disadvantage

A recently or frequently accessed block may be removed.

---

## 5.2 FIFO

**First In, First Out** replaces the block that entered the set first.

### Advantage

It can be implemented using a pointer or insertion-order information.

### Disadvantage

It does not consider how recently a block was accessed.

A block that entered the cache long ago but is still frequently used may be replaced.

---

## 5.3 LRU

**Least Recently Used** replaces the block that has not been accessed for the longest time.

### Advantage

It exploits temporal locality because recently used blocks are more likely to be used again.

### Disadvantage

Exact LRU becomes expensive as associativity increases because the cache must track the relative access order of all ways.

For a two-way cache, LRU can be maintained with one bit per set. For highly associative caches, simpler approximations are generally preferred.

---

## 5.4 Optimal Replacement

Optimal replacement removes the block whose next access is farthest in the future.

### Advantage

It produces the minimum possible number of cache misses.

### Disadvantage

It cannot be implemented in real hardware because future memory accesses are unknown.

It is mainly used as a theoretical reference.

---

# 6. Write Strategies

Cache write policies answer two independent questions:

1. What happens on a write hit?
2. What happens on a write miss?

---

## 6.1 Write-Through

On every write hit, both the cache and the next memory level are updated.

```text
Processor write → Cache + Next memory level
```

## Advantages

- Lower-level memory always contains updated data
- Dirty bit is not required
- Block replacement is simpler

## Disadvantages

- Every write generates lower-level memory traffic
- The processor may stall for slow memory writes
- Higher memory bandwidth and energy consumption

---

## 6.2 Write Buffer

A write-through cache commonly uses a write buffer.

```text
Processor → Cache → Write Buffer → Lower memory
```

The processor places the write in the buffer and continues execution. The buffer updates the lower memory in the background.

If the write buffer becomes full, the processor must stall.

If a later read requests an address present in the write buffer, the processor must obtain the newest value from the buffer or wait for the write to complete.

---

## 6.3 Write-Back

On a write hit, only the cache copy is updated.

The cache line is marked dirty:

```text
Dirty = 1
```

The modified block is written to the next memory level only when it is replaced.

## Advantages

- Lower memory traffic
- Multiple writes to the same block require only one final write-back
- Better performance for frequently written data

## Disadvantages

- A dirty bit is required
- The cache controller is more complex
- Replacing a dirty block takes additional time
- Lower-level memory may temporarily contain old data

## Replacement of a Write-Back Block

When a victim block is selected:

```text
Dirty = 0 → Replace it directly
Dirty = 1 → Write it back first, then replace it
```

---

## 6.4 Write Allocate

On a write miss:

1. Fetch the complete block into the cache.
2. Place it in an appropriate cache line.
3. Update the required byte or word.

Write allocate is useful when the block is likely to be accessed again.

It is commonly combined with write-back.

---

## 6.5 No-Write Allocate

On a write miss:

1. Do not fetch the block into the cache.
2. Send the write directly to the next memory level.

No-write allocate avoids filling the cache with data that may not be reused.

It is commonly combined with write-through.

---

## 6.6 Write-Strategy Summary

### Write-Hit Policies

| Policy | Operation |
|---|---|
| Write-through | Update cache and lower memory |
| Write-back | Update cache and set the dirty bit |

### Write-Miss Policies

| Policy | Operation |
|---|---|
| Write allocate | Fetch the block and then modify it |
| No-write allocate | Write directly to lower memory |

Common combinations are:

- **Write-back + Write allocate**
- **Write-through + No-write allocate**

> Write-back and write allocate are not the same thing. Write-back handles a write hit, while write allocate handles a write miss.

---

# 7. Cache Performance

## Hit Rate

```text
Hit rate = Number of cache hits / Total memory accesses
```

## Miss Rate

```text
Miss rate = 1 − Hit rate
```

## Hit Time

The time required to:

1. Access the cache
2. Compare the tag
3. Select and return the data

## Miss Penalty

The additional time required to:

1. Access the next memory level
2. Fetch the missing block
3. Install it in the cache
4. Return the requested data

## Average Memory Access Time

```text
AMAT = Hit time + (Miss rate × Miss penalty)
```

### Example

Suppose:

```text
Hit time    = 1 cycle
Miss rate   = 5%
Miss penalty = 50 cycles
```

Then:

```text
AMAT = 1 + (0.05 × 50)
     = 3.5 cycles
```

Therefore, cache optimizations attempt to:

- Reduce hit time
- Reduce miss rate
- Reduce miss penalty
- Increase cache bandwidth

---

# 8. Types of Cache Misses

The three basic cache misses are called the **3Cs**.

---

## 8.1 Compulsory Miss

A compulsory miss occurs on the first access to a memory block.

It is also called a:

- Cold miss
- First-reference miss

It can be reduced using:

- Larger cache blocks
- Hardware or software prefetching

---

## 8.2 Capacity Miss

A capacity miss occurs when the cache is too small to hold the program's active working set.

It would occur even in a fully associative cache of the same capacity.

It can be reduced using:

- A larger cache
- Loop blocking
- Better program data organization

---

## 8.3 Conflict Miss

A conflict miss occurs when multiple memory blocks compete for the same cache set, even though free space may exist in other sets.

It can be reduced using:

- Higher associativity
- Victim cache
- Better address or data placement

A fully associative cache does not have conflict misses.

---

# 9. Basic Cache Optimizations

Basic cache optimizations change the cache organization to reduce hit time, miss rate or miss penalty.

---

## 9.1 Small and Simple Cache

A smaller cache generally has:

- Faster decoding
- Shorter wires
- Lower hit time
- Lower area
- Lower power consumption

A direct-mapped cache is simpler because only one cache line and one tag are checked.

### Trade-Off

A small or direct-mapped cache may have a higher miss rate.

This is especially important for L1 cache because its hit time directly affects the processor pipeline.

---

## 9.2 Larger Block Size

A larger block exploits spatial locality by fetching more neighboring data.

## Advantage

- Reduces compulsory misses when nearby data is used

## Disadvantages

- Higher miss penalty
- More data transferred per miss
- Fewer cache lines for the same cache capacity
- Possible increase in capacity and conflict misses
- Cache pollution if the additional data is not used

Therefore, increasing block size helps only up to a certain point.

---

## 9.3 Larger Cache

A larger cache can hold a larger working set.

## Advantages

- Reduces capacity misses
- May also reduce some conflict misses

## Disadvantages

- Longer hit time
- Larger chip area
- Higher leakage power
- More complex physical implementation

---

## 9.4 Higher Associativity

Higher associativity gives each memory block more possible locations.

## Advantage

- Reduces conflict misses

## Disadvantages

- More tag comparisons
- Higher dynamic power
- Larger way-selection multiplexer
- More complex replacement logic
- Possibly longer hit time

A set-associative cache is therefore a balance between the speed of direct mapping and the low conflict-miss rate of fully associative mapping.

---

## 9.5 Multilevel Cache

Modern processors use multiple cache levels.

```text
Processor → L1 → L2 → L3 → Main Memory
```

- **L1:** Small and fast
- **L2:** Larger but slower
- **L3:** Larger and often shared by multiple cores

A small L1 provides a low hit time, while larger lower-level caches reduce expensive main-memory accesses.

For a two-level cache:

```text
AMAT = L1 hit time
     + L1 miss rate ×
       (L2 hit time + L2 miss rate × Main-memory penalty)
```

---

# 10. Final Summary

The complete cache-access process is:

```text
Processor generates an address
              ↓
Address is divided into tag, index and offset
              ↓
Index selects a cache line or set
              ↓
Requested tag is compared with stored tag(s)
              ↓
Is there a tag match with Valid = 1?
          ↙                              ↘
        Hit                              Miss
         ↓                                ↓
Select matching way                Select victim
         ↓                                ↓
Use block offset              Write back if dirty
         ↓                                ↓
Return requested data             Fetch new block
                                          ↓
                                  Update tag and data
                                          ↓
                                  Return requested data
```

The four fundamental cache-design decisions are:

1. **Block placement:** Where can a block be placed?
2. **Block identification:** How is a block located?
3. **Block replacement:** Which block is removed?
4. **Write strategy:** How are writes handled?

The main cache-design trade-off is:

```text
Hit Time vs Miss Rate vs Miss Penalty
```

- Direct mapping provides low hit time but more conflict misses.
- Set associativity reduces conflict misses but increases comparison hardware.
- Fully associative mapping eliminates conflict misses but has high hardware cost.
- Larger caches reduce capacity misses but increase area, power and possibly hit time.
- Larger blocks exploit spatial locality but increase refill cost and cache pollution.
- Advanced techniques increase bandwidth or hide miss penalties at the cost of additional hardware complexity.
