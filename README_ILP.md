# Instruction-Level Parallelism

> These notes follow Lectures 9–14 of the NPTEL **Advanced Computer Architecture** course by Prof. John Jose, IIT Guwahati.

## Index

1. [Introduction to Instruction-Level Parallelism](#1-introduction-to-instruction-level-parallelism)
2. [Instruction Dependencies](#2-instruction-dependencies)
3. [Compiler Techniques for Exploiting ILP](#3-compiler-techniques-for-exploiting-ilp)
4. [Dynamic Scheduling](#4-dynamic-scheduling)
5. [Tomasulo’s Algorithm](#5-tomasulos-algorithm)
6. [Speculative Execution and Reorder Buffer](#6-speculative-execution-and-reorder-buffer)
7. [Advanced Pipelining](#7-advanced-pipelining)
8. [Superscalar Processors](#8-superscalar-processors)
9. [Limits of Instruction-Level Parallelism](#9-limits-of-instruction-level-parallelism)
10. [Complete Execution Picture](#10-complete-execution-picture)
11. [Final Summary](#11-final-summary)

---

# 1. Introduction to Instruction-Level Parallelism

**Instruction-Level Parallelism (ILP)** is the ability to execute multiple independent instructions simultaneously or with overlapping execution.

Consider:

```assembly
ADD R1, R2, R3
SUB R4, R5, R6
```

These instructions use different operands and are independent. Therefore, they may execute simultaneously if the processor has sufficient hardware.

Now consider:

```assembly
ADD R1, R2, R3
SUB R4, R1, R5
```

The `SUB` instruction requires the value of `R1` produced by the `ADD`. Therefore, the instructions cannot execute independently.

## 1.1 Approaches for Exploiting ILP

There are two main approaches.

### Static Scheduling

- Performed by the compiler.
- Instructions are rearranged before execution.
- The processor follows the compiler-generated schedule.
- The compiler must ensure that program behaviour remains unchanged.

### Dynamic Scheduling

- Performed by processor hardware during execution.
- The processor detects which instructions are ready.
- Independent instructions may execute out of program order.
- It can respond to runtime events such as variable memory latency.

---

# 2. Instruction Dependencies

Before rearranging or executing instructions out of order, the compiler or processor must identify the dependencies among them.

## 2.1 True Data Dependence — RAW

A **Read After Write (RAW)** dependence occurs when an instruction requires a value produced by an earlier instruction.

```assembly
ADD R1, R2, R3
SUB R4, R1, R5
```

The first instruction writes `R1`, while the second instruction reads `R1`.

Therefore, the `SUB` must read `R1` only after the `ADD` produces it.

RAW represents the actual flow of data between instructions. It cannot be eliminated using register renaming.

---

## 2.2 Name Dependencies

Name dependencies occur when instructions use the same register or memory location even though there may be no actual transfer of data between them.

There are two types of name dependencies:

- WAR
- WAW

### Write After Read — WAR

```assembly
SUB R4, R1, R5
ADD R1, R2, R3
```

The first instruction must read the old value of `R1` before the second instruction writes a new value into it.

If the second instruction writes first, the first instruction reads an incorrect value.

### Write After Write — WAW

```assembly
MUL R1, R2, R3
ADD R1, R4, R5
```

Both instructions write into `R1`.

The writes must occur in program order so that the final value of `R1` comes from the second instruction.

WAR and WAW are not true data dependencies. They occur only because the same register name is reused.

Therefore, they can be eliminated using **register renaming**.

---

## 2.3 Control Dependence

An instruction is control-dependent on a branch when its execution depends on the outcome of that branch.

```assembly
BEQ R1, R2, LABEL
ADD R3, R4, R5
```

Whether the `ADD` instruction should execute depends on the result of the `BEQ`.

Moving instructions across branches may:

- Produce an incorrect result.
- Modify registers or memory incorrectly.
- Generate an exception that should not have occurred.
- Change the original program behaviour.

---

## 2.4 Summary of Dependencies

| Dependence | Meaning | Example relationship | Can renaming remove it? |
|---|---|---|---|
| RAW | Read After Write | Consumer reads producer’s result | No |
| WAR | Write After Read | Later instruction overwrites an earlier operand | Yes |
| WAW | Write After Write | Two instructions write the same destination | Yes |
| Control | Execution depends on a branch | Instruction after a conditional branch | No |

---

# 3. Compiler Techniques for Exploiting ILP

Static scheduling uses the compiler to locate independent instructions and rearrange them to reduce pipeline stalls.

The important compiler techniques are:

- Instruction scheduling
- Loop unrolling
- Register renaming
- Static branch prediction

---

## 3.1 Instruction Scheduling

Consider the following loop:

```assembly
LOOP:
    L.D   F0, 0(R1)
    ADD.D F4, F0, F2
    S.D   0(R1), F4
    DADDI R1, R1, -8
    BNEZ  R1, LOOP
```

The `ADD.D` instruction immediately uses the value loaded into `F0`.

Therefore, a load-use stall may occur.

The compiler can move an independent instruction between the load and its consumer:

```assembly
LOOP:
    L.D   F0, 0(R1)
    DADDI R1, R1, -8
    ADD.D F4, F0, F2
    S.D   8(R1), F4
    BNEZ  R1, LOOP
```

The store offset changes from `0(R1)` to `8(R1)` because `R1` is now decremented before the store.

### Objective of Instruction Scheduling

Instruction scheduling attempts to:

- Separate dependent instructions.
- Fill stall cycles with useful instructions.
- Reduce pipeline bubbles.
- Improve functional-unit utilization.
- Preserve the original program behaviour.

---

## 3.2 Loop Unrolling

Loops generally contain useful computation along with loop-control instructions.

Consider:

```c
for (i = 1000; i > 0; i--)
    x[i] = x[i] + s;
```

```assembly
LOOP:
    L.D    F0, 0(R1)       # F0 = x[i]
    ADD.D  F4, F0, F2      # F4 = x[i] + s
    S.D    F4, 0(R1)       # x[i] = F4
    DADDI  R1, R1, -8      # Move to x[i-1]
    DADDI  R2, R2, -1      # Decrement loop counter
    BNEZ   R2, LOOP        # Repeat until R2 = 0
```

Every iteration requires:

- Loading an element.
- Performing the addition.
- Storing the result.
- Updating the pointer.
- Executing a branch.

The loop can be unrolled to process multiple elements during one iteration:

```assembly
LOOP:
    L.D   F0,    0(R1)
    L.D   F6,   -8(R1)
    L.D   F10, -16(R1)
    L.D   F14, -24(R1)

    ADD.D F4,  F0,  F2
    ADD.D F8,  F6,  F2
    ADD.D F12, F10, F2
    ADD.D F16, F14, F2

    S.D    0(R1), F4
    S.D   -8(R1), F8
    S.D  -16(R1), F12
    S.D  -24(R1), F16

    DADDI R1, R1, -32
    BNEZ  R1, LOOP
```

Four original loop iterations are now combined into one larger iteration.

### Advantages of Loop Unrolling

- Reduces the number of branch instructions.
- Reduces loop-counter and pointer updates.
- Exposes independent instructions from different iterations.
- Allows instructions from different iterations to overlap.
- Provides more scheduling opportunities.
- Improves functional-unit utilization.

### Requirements for Loop Unrolling

The compiler must:

- Check whether different loop iterations are independent.
- Use different registers for different iterations.
- Modify memory offsets correctly.
- Change the loop-counter update according to the unrolling factor.
- Schedule the resulting instructions to minimize stalls.
- Handle cases where the number of iterations is not divisible by the unrolling factor.

### Limitations of Loop Unrolling

Excessive unrolling can:

- Increase program code size.
- Increase register requirements.
- Cause register spilling.
- Increase instruction-cache pressure.
- Provide diminishing performance improvement.

---

## 3.3 Register Renaming by the Compiler

## Register Renaming

Register renaming is a hardware technique used in **out-of-order processors** to eliminate **false data dependencies** between instructions.

Consider:

```assembly
I1: R1 = R2 + R3
I2: R4 = R1 + R5
I3: R1 = R6 + R7
I4: R8 = R1 + R9
```

Dependencies:

- `I1 → I2`: **RAW dependency** — true dependency.
- `I1 → I3`: **WAW dependency** — false dependency.
- `I2 → I3`: **WAR dependency** — false dependency.
- `I3 → I4`: **RAW dependency** — true dependency.

`I1` and `I3` both write to architectural register `R1`, but their results are logically different values.

### Renaming

The processor maps architectural registers to a larger set of physical registers:

```assembly
I1: P10 = P2 + P3
I2: P11 = P10 + P5
I3: P12 = P6 + P7
I4: P13 = P12 + P9
```

Now:

- `I1` writes to `P10`.
- `I3` writes to `P12`.
- The WAW and WAR dependencies disappear.
- Only the true RAW dependencies remain.

Therefore, `I3` does not need to wait for `I1` or `I2` and may execute earlier if its operands are ready.

## How Hardware Performs Renaming

The processor typically maintains:

- **Architectural Register File (ARF):** Registers visible to the program, such as `R0–R31`.
- **Physical Register File (PRF):** A larger collection of internal registers.
- **Register Alias Table (RAT):** Maps each architectural register to its latest physical register.
- **Free List:** Tracks unused physical registers.
- **Reorder Buffer (ROB):** Preserves program order during retirement and supports precise exceptions.

Suppose initially:

```text
R1 → P1
R2 → P2
R3 → P3
```

For the instruction:

```assembly
ADD R1, R2, R3
```

During renaming:

1. Source registers are looked up in the RAT:

   ```text
   R2 → P2
   R3 → P3
   ```

2. A free physical register, say `P10`, is allocated for destination `R1`.

3. The instruction becomes:

   ```assembly
   ADD P10, P2, P3
   ```

4. The RAT is updated:

   ```text
   R1 → P10
   ```

A later instruction reading `R1` will therefore read its latest value from `P10`.

## Important Ordering Rule

For an instruction such as:

```assembly
ADD R1, R1, R2
```

The processor must rename the **source registers before updating the destination mapping**.

Initially:

```text
R1 → P5
R2 → P2
```

Correct renaming:

```assembly
ADD P10, P5, P2
```

Then, the RAT is updated:

```text
R1 → P10
```

The old value of `R1` comes from `P5`, while the new value is written to `P10`.

## What Register Renaming Removes

| Dependency | Meaning | Removed by renaming? |
|---|---|---|
| RAW | Read After Write | No |
| WAR | Write After Read | Yes |
| WAW | Write After Write | Yes |

RAW cannot be removed because it represents an actual flow of data between instructions.

---

## 3.4 Static Scheduling Limitations

A compiler-generated schedule is fixed before the program executes.

The compiler may not know:

- Whether a cache access will hit or miss.
- The actual latency of a memory operation.
- The outcome of a branch.
- Whether two computed memory addresses are identical.
- Whether an execution unit will be available.
- The exact runtime behaviour of the program.

Dynamic scheduling is used to respond to these runtime conditions.

---

# 4. Dynamic Scheduling

In a statically scheduled pipeline, a stalled instruction may prevent later instructions from proceeding even when those instructions are independent.

Consider:

```assembly
DIV.D F0, F2, F4
ADD.D F6, F8, F10
SUB.D F12, F6, F14
```

`DIV.D` may require many clock cycles.

However, `ADD.D` does not depend on `DIV.D`. A dynamically scheduled processor can allow `ADD.D` to execute without waiting for the division to complete.

## 4.1 Main Idea

Dynamic scheduling allows:

- Instructions to be issued in program order.
- Instructions to wait until their operands become available.
- Independent instructions to bypass stalled instructions.
- Instructions to execute out of program order.
- Instructions to complete out of program order.

The processor separates:

1. Checking whether an instruction is ready.
2. Sending the instruction to an execution unit.

---

## 4.2 Advantages

Dynamic scheduling:

- Handles dependencies discovered during execution.
- Tolerates unpredictable execution and memory delays.
- Allows independent instructions to proceed.
- Reduces unnecessary pipeline stalls.
- Extracts ILP without depending completely on the compiler.

---

## 4.3 Problems Introduced by Out-of-Order Execution

Out-of-order execution can introduce:

- WAR hazards.
- WAW hazards.
- Imprecise exceptions.
- Difficulty recovering from incorrect branch predictions.
- Incorrect architectural state if speculative instructions update registers or memory.

Tomasulo’s algorithm handles dynamic scheduling and name dependencies.

The reorder buffer extends Tomasulo’s algorithm to support:

- In-order commit
- Precise exceptions
- Speculative execution
- Branch-misprediction recovery

---

# 5. Tomasulo’s Algorithm

Tomasulo’s algorithm is a hardware-based dynamic scheduling technique.

Its important features are:

- Distributed hazard detection.
- Reservation stations.
- Register renaming.
- Operand forwarding.
- Common Data Bus.
- Out-of-order execution.
- Out-of-order completion.

---

## 5.1 Main Components

### Reservation Stations

Reservation stations hold instructions waiting to execute.

Each reservation station contains information such as:

- Operation to perform.
- Available operand values.
- Tags identifying unavailable operands.
- Busy status.

A reservation station may be represented using fields such as:

| Field | Meaning |
|---|---|
| Busy | Indicates whether the reservation station is occupied |
| Op | Operation to be performed |
| Vj | Available value of the first operand |
| Vk | Available value of the second operand |
| Qj | Tag of the unit producing the first operand |
| Qk | Tag of the unit producing the second operand |

If an operand is available, its value is stored in `Vj` or `Vk`.

If an operand is unavailable, the tag of its producer is stored in `Qj` or `Qk`.

An instruction begins execution when:

- All its operands are available.
- The required functional unit is available.

---

### Register Status Indicator

The register-status information indicates whether a register value is:

- Available in the register file, or
- Expected from a reservation station.

If the value is unavailable, the register status stores the tag of the reservation station that will produce it.

---

### Functional Units

Functional units perform the actual operations.

Examples include:

- Integer ALU
- Floating-point adder
- Floating-point multiplier
- Floating-point divider
- Load/store unit

Different functional units may have different execution latencies.

---

### Common Data Bus

The **Common Data Bus (CDB)** broadcasts:

- A completed result.
- The tag of the reservation station that produced it.

All reservation stations monitor the CDB.

If a reservation station is waiting for the broadcast tag:

- It captures the value.
- The corresponding operand becomes ready.
- The producer tag is cleared.

The register file also captures the value when the broadcast tag matches its current producer.

This allows a result to be forwarded directly to waiting instructions without requiring the consumer to wait for a separate register-file access.

---

## 5.2 Tomasulo Execution Stages

Tomasulo’s algorithm contains three main stages:

1. Issue
2. Execute
3. Write result

---

### Stage 1: Issue

Instructions are examined in program order.

If a suitable reservation station is available:

- The instruction is placed in the reservation station.
- Available operand values are copied.
- Tags are stored for unavailable operands.
- The destination register is associated with the reservation station.

If no suitable reservation station is available, instruction issue stalls.

Although execution may occur out of order, instruction issue generally occurs in order.

---

### Stage 2: Execute

The instruction waits in its reservation station until all operands become available.

Once the operands and functional unit are available:

- Execution begins.
- The operation occupies the functional unit for the required number of cycles.

Different instructions may start and finish execution out of program order.

For load and store instructions, the effective address must also be calculated.

---

### Stage 3: Write Result

After execution, the functional unit broadcasts:

- The result.
- Its reservation-station tag.

Waiting reservation stations compare the broadcast tag with `Qj` and `Qk`.

If a tag matches:

- The result is copied into `Vj` or `Vk`.
- The corresponding tag is cleared.

The register file accepts the result only if its current producer tag matches the broadcast tag.

The reservation station is released after broadcasting its result.

---

## 5.3 Example of Operand Tracking

Consider:

```assembly
MUL.D F0, F2, F4
ADD.D F6, F0, F8
```

Suppose the multiply instruction is placed in reservation station `Mult1`.

The second instruction needs `F0`, but `F0` is not yet available.

Its reservation station may contain:

```text
Vj = unavailable
Qj = Mult1
Vk = value of F8
Qk = empty
```

When `Mult1` finishes, it broadcasts:

```text
Tag   = Mult1
Value = multiplication result
```

The add reservation station detects the matching tag and captures the result.

The `ADD.D` can then begin execution.

---

## 5.4 Register Renaming in Tomasulo’s Algorithm

Consider:

```assembly
MUL.D F0, F2, F4
ADD.D F0, F6, F8
```

Both instructions use `F0` as their destination, creating a WAW dependence.

Tomasulo’s algorithm assigns different internal tags:

```text
First write to F0  → Mult1
Second write to F0 → Add1
```

The instructions now have different internal names even though their architectural destination is the same.

The register-status entry for `F0` is updated to point to `Add1`, because it is the latest instruction that will write `F0`.

When `Mult1` finishes, it must not incorrectly overwrite the value expected from `Add1`.

Register renaming eliminates:

- WAR hazards.
- WAW hazards.

It does not eliminate RAW dependence because the consumer genuinely needs the producer’s result.

---

## 5.5 Advantages of Tomasulo’s Algorithm

- Supports out-of-order execution.
- Reduces stalls caused by long-latency instructions.
- Eliminates WAR and WAW hazards through register renaming.
- Forwards results directly to waiting instructions.
- Uses distributed dependency checking.
- Allows multiple instructions to wait near their functional units.

---

## 5.6 Limitations of Basic Tomasulo’s Algorithm

In basic Tomasulo scheduling, results may update registers as soon as they finish.

Therefore:

- Architectural state may be updated out of order.
- Exceptions may become imprecise.
- Recovery from an incorrectly predicted branch is difficult.
- Instructions following a branch cannot be safely committed speculatively.
- A store may update memory before it is known to be safe.

These problems motivate the use of a **Reorder Buffer**.

---

# 6. Speculative Execution and Reorder Buffer

Speculative execution allows instructions following a predicted branch to execute before the actual branch outcome is known.

If the prediction is correct, useful execution time is saved.

If the prediction is incorrect, speculative instructions must be removed without changing the correct architectural state.

---

## 6.1 Reorder Buffer

A **Reorder Buffer (ROB)** stores the results of instructions that have executed but have not yet committed.

Instructions are allocated ROB entries in program order.

The ROB behaves like a queue:

- New instructions enter at the tail.
- Instructions commit from the head.

A ROB entry contains information such as:

| Field | Meaning |
|---|---|
| Instruction type | Register operation, store, branch, etc. |
| Destination | Destination register or memory address |
| Value | Computed result |
| Ready | Indicates whether execution has completed |
| Exception | Indicates whether an exception occurred |

The ROB separates:

- Completion of execution
- Permanent update of architectural state

---

## 6.2 Execution Stages with Speculation

A speculative dynamically scheduled processor uses four conceptual stages:

1. Issue
2. Execute
3. Write result
4. Commit

---

### Stage 1: Issue

When an instruction is issued:

- A reservation station is allocated.
- A ROB entry is allocated.
- The destination register is associated with the ROB entry.
- Available operands are copied.
- Tags are stored for unavailable operands.

Instructions are issued in program order.

If a reservation station or ROB entry is unavailable, issue stalls.

---

### Stage 2: Execute

The instruction waits until its operands become available.

When the operands and functional unit are available:

- Execution begins.
- Instructions may execute out of program order.

For loads and stores, the effective address must also be determined.

---

### Stage 3: Write Result

When execution finishes:

- The result is written into the corresponding ROB entry.
- The ROB entry is marked ready.
- The result may be forwarded to waiting reservation stations.

The architectural register file is not permanently updated at this stage.

---

### Stage 4: Commit

An instruction commits when:

- It reaches the head of the ROB.
- Its execution has completed.
- It has not generated an exception.
- It is not invalid because of incorrect speculation.

The instruction then updates the architectural state.

Although instructions execute and finish out of order, they commit in program order.

---

## 6.3 Register Instruction Commit

When a register-writing instruction reaches the ROB head:

- Its result is written into the architectural register.
- Its ROB entry is removed.

If a younger instruction also writes the same register, the mapping belonging to the younger instruction must be preserved.

---

## 6.4 Store Instruction Commit

A store must not update memory immediately after execution because it may still be speculative.

Instead:

- The store address is held in the ROB.
- The store data is held in the ROB.
- Memory is updated only when the store reaches the ROB head and commits.

This prevents a wrongly speculated store from corrupting memory.

---

## 6.5 Branch Misprediction Recovery

When the actual branch outcome becomes known:

### Correct Prediction

- Speculative execution continues.
- Instructions eventually commit normally.

### Incorrect Prediction

- Younger speculative instructions are flushed.
- Their ROB entries are removed.
- Their reservation stations are cleared.
- Incorrect register mappings are removed.
- Instruction fetching restarts from the correct branch target.

Because speculative instructions were not allowed to commit, the architectural state remains correct.

---

## 6.6 Precise Exceptions

An exception is precise when:

- All instructions before the faulting instruction have completed.
- The faulting instruction has not modified architectural state.
- Instructions after the faulting instruction have not modified architectural state.

The ROB provides precise exceptions through in-order commit.

If an instruction generates an exception:

- The exception information is stored in its ROB entry.
- Earlier instructions are allowed to commit.
- The exception is handled when the faulting instruction reaches the ROB head.
- The faulting and younger instructions are prevented from committing.

---

## 6.7 Basic Tomasulo vs Tomasulo with Speculation

| Property | Basic Tomasulo | Tomasulo with ROB |
|---|---|---|
| Issue | In order | In order |
| Execution | Out of order | Out of order |
| Result production | Out of order | Out of order |
| Architectural update | May occur out of order | Occurs in order |
| Register renaming | Reservation-station tags | ROB tags |
| Precise exceptions | Difficult | Supported |
| Branch speculation | Difficult to recover | Supported |
| Stores update memory | After execution | During commit |
| Commit stage | Not present | Present |

---

# 7. Advanced Pipelining

Processor performance can be improved by:

- Increasing pipeline depth.
- Issuing multiple instructions every cycle.
- Executing instructions out of order.
- Predicting branches.
- Executing instructions speculatively.

---

## 7.1 Superpipelining

A superpipelined processor divides normal pipeline stages into smaller stages.

For example, operations in a five-stage pipeline may be divided into a larger number of shorter stages.

### Benefit

Shorter pipeline stages reduce the amount of logic between pipeline registers.

This can allow the processor to operate at a higher clock frequency.

### Limitations

- Additional pipeline-register overhead.
- Increased branch-misprediction penalty.
- More complex forwarding logic.
- More complex hazard detection.
- Unequal stage delays limit frequency improvement.
- Increasing pipeline depth does not necessarily increase the number of instructions issued per cycle.

---

# 8. Superscalar Processors

A **superscalar processor** can issue multiple instructions during one clock cycle.

For example, a two-issue processor may issue up to two instructions per cycle:

```text
Cycle 1: Issue I1 and I2
Cycle 2: Issue I3 and I4
```

To execute multiple instructions simultaneously, the processor may contain multiple functional units such as:

- Integer ALUs
- Floating-point adders
- Floating-point multipliers
- Load/store units
- Branch units

---

## 8.1 Superscalar Width

If a processor can issue at most `n` instructions per cycle, it is called an **n-issue processor**.

The ideal CPI is:

\[
CPI_{\text{ideal}} = \frac{1}{n}
\]

For a four-issue processor:

\[
CPI_{\text{ideal}} = \frac{1}{4} = 0.25
\]

This is only an ideal value.

The actual issue rate is limited by:

- Data dependencies.
- Control dependencies.
- Memory dependencies.
- Functional-unit availability.
- Instruction-fetch limitations.
- Branch mispredictions.

---

## 8.2 Requirements of a Superscalar Processor

A superscalar processor must be able to:

- Fetch multiple instructions per cycle.
- Decode multiple instructions.
- Check dependencies among instructions.
- Rename registers.
- Dispatch instructions to different functional units.
- Execute multiple instructions simultaneously.
- Handle multiple completed results.
- Commit multiple instructions while preserving program order.

---

## 8.3 Static Multiple Issue

In a statically scheduled multiple-issue processor:

- The compiler identifies independent instructions.
- The compiler groups operations that can execute together.
- The processor performs relatively limited runtime scheduling.

A major example is a **Very Long Instruction Word (VLIW)** processor.

---

## 8.4 VLIW Architecture

A VLIW instruction contains multiple independent operations packed into one long instruction word.

```text
| Integer operation | FP operation | Memory operation | Branch operation |
```

The compiler determines which operations can execute simultaneously.

### Advantages

- Simpler hardware scheduling.
- Less runtime dependency-checking hardware.
- Parallel operations are explicitly identified by the compiler.
- Reduced scheduling complexity inside the processor.

### Limitations

- Performance strongly depends on compiler quality.
- Unused instruction slots may be wasted.
- Code size may increase.
- A schedule created for one processor implementation may not perform well on another.
- Unpredictable events such as cache misses are difficult to handle statically.
- Compatibility across different implementations becomes difficult.

---

## 8.5 Dynamic Multiple Issue

In a dynamically scheduled superscalar processor:

- Multiple instructions are fetched every cycle.
- Instructions are decoded and issued in program order.
- Hardware dynamically checks dependencies.
- Register renaming removes WAR and WAW dependencies.
- Ready instructions execute out of order.
- Results may complete out of order.
- Instructions commit in program order using the ROB.

A modern dynamic superscalar processor combines:

- Multiple-issue execution.
- Dynamic scheduling.
- Reservation stations.
- Register renaming.
- Branch prediction.
- Speculative execution.
- Reorder buffer.
- In-order commit.

---

## 8.6 Superpipelined vs Superscalar

| Feature | Superpipelined | Superscalar |
|---|---|---|
| Main idea | Divide execution into more stages | Issue multiple instructions per cycle |
| Primary goal | Increase clock frequency | Increase instructions executed per cycle |
| Issue width | May remain one | Greater than one |
| Major challenge | Deeper pipeline and branch penalty | Dependency checking and multiple execution units |
| Parallelism | Greater overlap between stages | Simultaneous issue and execution |

A processor may be both superpipelined and superscalar.

---

# 9. Limits of Instruction-Level Parallelism

A processor may support a large issue width, but useful ILP is limited by several factors.

## 9.1 Data Dependencies

RAW dependencies force consumer instructions to wait for their producers.

Register renaming cannot eliminate true data dependencies.

---

## 9.2 Control Dependencies

Branches interrupt the continuous supply of independent instructions.

If a branch is mispredicted:

- Speculative instructions are discarded.
- Pipeline work is lost.
- Instruction fetching restarts from the correct path.

The penalty generally increases with pipeline depth.

---

## 9.3 Memory Dependencies

Loads and stores may access the same memory address.

The processor must ensure that memory operations occur in a logically correct order.

Sometimes the exact addresses are known only at runtime.

---

## 9.4 Limited Instruction Window

The processor can search for independent instructions only among the instructions currently present in its scheduling window.

A small window may not contain enough independent instructions.

A larger window can expose more ILP but increases hardware complexity.

---

## 9.5 Limited Hardware Resources

Performance is limited by the number of:

- Functional units.
- Reservation stations.
- ROB entries.
- Load/store units.
- Register-file ports.
- Result-forwarding resources.

---

## 9.6 Common Data Bus Bottleneck

If only one CDB is available, only one result can be broadcast during a cycle.

Multiple instructions completing simultaneously may have to wait before broadcasting their results.

---

## 9.7 Superscalar Complexity

Increasing issue width requires:

- More instructions to be fetched.
- More instructions to be decoded.
- More dependency comparisons.
- More register-file ports.
- More execution units.
- Wider result-forwarding paths.
- More complex commit logic.

Therefore, doubling the issue width does not necessarily double processor performance.

---

# 10. Complete Execution Picture

A dynamically scheduled superscalar processor broadly operates as follows:

```text
Fetch multiple instructions
        ↓
Decode instructions
        ↓
Issue instructions in program order
        ↓
Allocate reservation stations and ROB entries
        ↓
Rename destination registers
        ↓
Wait until operands become ready
        ↓
Execute ready instructions out of order
        ↓
Forward results to waiting instructions
        ↓
Store completed results in the ROB
        ↓
Commit instructions in program order
```

The ordering properties can be summarized as:

| Operation | Order |
|---|---|
| Instruction issue | In order |
| Instruction execution | Out of order |
| Execution completion | Out of order |
| Architectural commit | In order |

---

# 11. Final Summary

| Technique | Performed by | Main purpose |
|---|---|---|
| Instruction scheduling | Compiler | Separate dependent instructions |
| Loop unrolling | Compiler | Expose parallelism across loop iterations |
| Compiler register renaming | Compiler | Remove false dependencies |
| Dynamic scheduling | Hardware | Execute ready instructions out of order |
| Reservation stations | Hardware | Hold instructions until operands become ready |
| Register-status table | Hardware | Identify the latest producer of a register |
| Register renaming | Hardware | Eliminate WAR and WAW dependencies |
| Common Data Bus | Hardware | Broadcast and forward completed results |
| Reorder buffer | Hardware | Maintain program order during commit |
| Speculative execution | Hardware | Execute beyond predicted branches |
| In-order commit | Hardware | Maintain correct architectural state |
| Superpipelining | Hardware | Increase clock frequency |
| Superscalar execution | Hardware/compiler | Issue multiple instructions per cycle |
| VLIW | Primarily compiler | Encode multiple parallel operations together |

The complete idea can be summarized as:

> The compiler exposes independent instructions, dynamic scheduling finds runtime execution opportunities, Tomasulo’s algorithm enables out-of-order execution, and the reorder buffer restores in-order architectural behaviour.
