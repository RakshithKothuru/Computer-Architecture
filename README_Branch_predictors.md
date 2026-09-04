# Branch Prediction in a 5-Stage RISC-V Pipeline

## 1. Why Branch Prediction?

Consider the classic 5-stage RISC-V pipeline:

```text
IF → ID → EX → MEM → WB
```

For a conditional branch:

```asm
BEQ x1, x2, TARGET
```

the next instruction can come from:

```text
Not Taken → PC + 4
Taken     → TARGET
```

In a basic 5-stage pipeline, the actual branch outcome is resolved in the **EX stage**. Waiting until EX before fetching the next instruction would waste cycles.

Therefore, the processor **predicts the branch outcome** and continues fetching instructions.

- **Correct prediction** → Continue normally
- **Wrong prediction** → Flush wrong-path instructions and redirect the PC

This reduces the **control hazard penalty**.

---

## 2. Static Branch Prediction

Static predictors do not learn from previous branch behavior.

### Always Not Taken

Every branch is predicted as:

```text
Next PC = PC + 4
```

### Always Taken

Every branch is predicted as:

```text
Next PC = Branch Target
```

### Backward Taken, Forward Not Taken

```text
Backward branch → Predict Taken
Forward branch  → Predict Not Taken
```

Backward branches are often associated with loops and are therefore frequently taken.

---

## 3. Dynamic Branch Prediction

Dynamic predictors use the **previous behavior of branches** to predict future outcomes.

### 1-Bit Predictor

The simplest dynamic predictor stores one bit representing the previous outcome:

| Bit | Prediction |
|-----|------------|
| `0` | Not Taken |
| `1` | Taken |

For example, a loop branch may behave as:

```text
T T T T T T T T T N
```

The branch is Taken while the loop continues and Not Taken when the loop exits.

The problem with a 1-bit predictor is that **a single unusual outcome immediately changes the prediction**.

---

## 4. 2-Bit Saturating Counter

A better predictor uses a **2-bit saturating counter**.

| State | Meaning | Prediction |
|-------|---------|------------|
| `00` | Strongly Not Taken | Not Taken |
| `01` | Weakly Not Taken | Not Taken |
| `10` | Weakly Taken | Taken |
| `11` | Strongly Taken | Taken |

### If Actual Outcome = Taken

```text
00 → 01 → 10 → 11
```

### If Actual Outcome = Not Taken

```text
11 → 10 → 01 → 00
```

The counter saturates at `00` and `11`.

For example, suppose the current state is:

```text
11 → Strongly Taken
```

and the actual outcome is Not Taken:

```text
11 → 10
```

The predictor becomes **Weakly Taken**, but still predicts Taken.

If another Not-Taken outcome occurs:

```text
10 → 01
```

the prediction changes to Not Taken.

Thus, a single unusual outcome does not immediately reverse the prediction.

---

## 5. Branch History Table (BHT)

Different branch instructions can behave differently.

For example:

```text
BEQ @ 0x100 → Usually Taken
BNE @ 0x180 → Usually Not Taken
BLT @ 0x240 → Usually Taken
```

Using **one common 2-bit counter** for all branches would mix their behavior.

Ideally, we want:

```text
BEQ @ 0x100 → 11
BNE @ 0x180 → 00
BLT @ 0x240 → 10
```

Therefore, the processor maintains **many prediction counters** in a table called the **Branch History Table (BHT)**.

```text
             BHT

        +-------------+
Entry 0 | 2-bit count |
        +-------------+
Entry 1 | 2-bit count |
        +-------------+
Entry 2 | 2-bit count |
        +-------------+
        |     ...     |
        +-------------+
```

So, in a simple implementation:

> **BHT = Table containing many 2-bit branch prediction counters.**

---

## 6. How Is the BHT Accessed?

The **branch PC** is used to index the BHT.

```text
Branch PC
    │
    ▼
Select index bits
    │
    ▼
   BHT
    │
    ▼
2-bit counter
    │
    ▼
Taken / Not Taken
```

For example:

```text
Branch PC = 0x100
       ↓
BHT Entry = 64
       ↓
BHT[64] = 11
       ↓
Predict Taken
```

When the branch eventually resolves in EX, the same BHT counter is updated based on the actual outcome.

For example:

```text
Current state = 11
Actual         = Not Taken

11 → 10
```

---

## 7. Fixed BHT Size and Aliasing

The BHT is **physical hardware inside the processor**, so its size is fixed when the processor is designed.

For example:

```text
1024-entry BHT
```

means the processor contains:

```text
1024 × 2-bit prediction counters
```

The processor does **not** create one new BHT entry for every branch instruction in a program.

Suppose a program contains:

```text
30 BEQ
10 BNE
 2 BLT
------
42 branch instructions
```

These 42 branches are mapped into the processor's existing BHT using their PCs.

For a 1024-entry BHT:

```text
1024 = 2^10
```

so 10 index bits are required to select one of the 1024 entries.

Because only part of the PC is used as the index, two different branches can sometimes select the same BHT entry.

For example:

```text
Branch A ──┐
           ├──→ BHT[64] → 2-bit counter
Branch B ──┘
```

Now both branches share the same prediction state and can interfere with each other.

This is called **BHT aliasing**.

For basic understanding:

> **Conceptually:** One branch → One prediction counter  
>
> **Real hardware:** Multiple branches may map to the same BHT counter.

---

## 8. Branch Target Buffer (BTB)

The BHT only answers:

> **Will the branch be Taken or Not Taken?**

If the branch is predicted Taken, the processor also needs to know:

> **Where should it fetch from?**

This is the job of the **Branch Target Buffer (BTB)**.

The BTB stores predicted target addresses:

```text
Branch PC → BTB → Target Address
```

Therefore:

```text
              Branch PC
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
       BHT                 BTB
        │                   │
        ▼                   ▼
Taken / Not Taken      Target Address
        │                   │
        └─────────┬─────────┘
                  ▼
                Next PC
```

Remember:

> **BHT → Taken or Not Taken?**  
> **BTB → If Taken, where to go?**

---

## 9. Cycle-by-Cycle Example

Consider:

```asm
0x100: BEQ x1, x2, TARGET
0x104: ADD x3, x4, x5
0x108: SUB x6, x7, x8

0x200: TARGET:
       AND x9, x10, x11
```

Suppose:

```text
BHT prediction = 11 → Strongly Taken
BTB target     = 0x200
```

### Cycle 1

The `BEQ` is fetched:

```text
IF: BEQ
```

The prediction hardware gives:

```text
BHT → Strongly Taken
BTB → Target = 0x200
```

Therefore:

```text
Next PC = 0x200
```

---

### Cycle 2

The branch moves to ID while the target instruction is fetched:

```text
ID: BEQ
IF: AND @ 0x200
```

Because the branch was predicted Taken, the processor fetches from `0x200` instead of `0x104`.

---

### Cycle 3

The branch reaches EX:

```text
EX: BEQ
ID: AND
IF: Next target instruction
```

The actual branch condition is now evaluated.

Two cases are possible.

---

### Case 1: Prediction Is Correct

Suppose:

```text
x1 == x2
```

Therefore:

```text
Prediction = Taken
Actual     = Taken
```

The target instructions already being executed are correct.

No flush is required.

The predictor remains:

```text
11 → 11
```

since it is already Strongly Taken.

---

### Case 2: Prediction Is Wrong

Suppose:

```text
x1 != x2
```

Therefore:

```text
Prediction = Taken
Actual     = Not Taken
```

At this point:

```text
EX : BEQ       → Actual = Not Taken
ID : AND       → Wrong path
IF : Next inst → Wrong path
```

The instructions in IF and ID must be **flushed**.

```text
ID instruction → FLUSH
IF instruction → FLUSH
```

The processor redirects the PC to the correct sequential instruction:

```text
PC = 0x104
```

The BHT counter is also updated:

```text
11 → 10

Strongly Taken → Weakly Taken
```

Execution then continues from:

```asm
0x104: ADD x3, x4, x5
```

---

## 10. Branch Misprediction Penalty

In the classic pipeline:

```text
IF → ID → EX → MEM → WB
```

if the branch is resolved in **EX**, two younger instructions may already be present in:

```text
ID
IF
```

If the prediction is wrong, both instructions must be flushed.

Therefore, in this simple pipeline, the branch misprediction penalty is approximately:

```text
2 cycles
```

---

## 11. Complete Branch Prediction Flow

```text
                    Current PC
                        │
            ┌───────────┴───────────┐
            ▼                       ▼
           BHT                     BTB
            │                       │
            ▼                       ▼
   Taken / Not Taken          Target Address
            │                       │
            └───────────┬───────────┘
                        ▼
                 Next PC Selection
                  /             \
             Not Taken          Taken
                 │                │
               PC + 4          Target
```

Later, when the branch reaches EX:

```text
Branch reaches EX
        │
        ▼
Actual outcome determined
        │
   ┌────┴────┐
   ▼         ▼
Correct     Wrong
   │         │
Continue   Flush wrong
           instructions
               │
               ▼
          Redirect PC
               │
               ▼
          Update BHT
```

---

## 12. Quick Summary

- Branch instructions create **control hazards**.
- Branch prediction predicts the outcome before the branch is resolved.
- **Static predictors** do not learn from previous outcomes.
- **Dynamic predictors** learn from previous branch behavior.
- A **1-bit predictor** stores the previous outcome.
- A **2-bit saturating counter** provides more stable prediction.
- **BHT** stores multiple branch prediction counters.
- The **branch PC indexes the BHT**.
- BHT size is fixed in hardware.
- Multiple branches mapping to the same BHT entry causes **aliasing**.
- **BHT → Taken or Not Taken?**
- **BTB → If Taken, what is the target address?**
- Correct prediction → Continue normally.
- Wrong prediction → Flush wrong-path instructions and redirect the PC.
- In a classic 5-stage pipeline with branch resolution in EX, the misprediction penalty is approximately **2 cycles**.