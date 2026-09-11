# Semiconductor Memories

## Index

1. [Memory Fundamentals](#1-memory-fundamentals)
2. [Memory-Chip Organization](#2-memory-chip-organization)
3. [SRAM](#3-sram)
4. [DRAM](#4-dram)
5. [ROM and Non-Volatile Memories](#5-rom-and-non-volatile-memories)
6. [Flash Memory](#6-flash-memory)
7. [Programmable Logic Devices](#7-programmable-logic-devices)
8. [Memory Timing and Power](#9-memory-timing-and-power)
9. [Memory Redundancy](#10-memory-redundancy)
10. [Important Comparisons](#11-important-comparisons)

---

## 1. Memory Fundamentals

Semiconductor memory stores binary information using integrated electronic circuits.

### Memory Hierarchy

1. Registers
2. Cache memory
3. Main memory
4. Secondary storage

Moving downward in the hierarchy:

- Capacity increases.
- Cost per bit decreases.
- Access time increases.
- Speed decreases.

### Memory Organization

Memory is represented as:

`Number of words × Bits per word`

Example:

`1K × 8`

This memory contains:

- `1024` words
- `8` bits per word
- `10` address lines
- `8` data lines
- Total capacity of `8192 bits`

For a memory organized as `N × M`:

`Total capacity = N × M bits`

`Number of address lines = log₂(N)`

`Number of data lines = M`

### Memory Operations

#### Read Operation

1. Apply the address.
2. Enable the memory chip.
3. Activate the read or output-enable signal.
4. The selected data appears at the output.

A read operation normally does not modify the stored data, except internally in memories such as DRAM where the value must be restored.

#### Write Operation

1. Apply the address.
2. Apply the input data.
3. Enable the memory chip.
4. Activate write enable.
5. The data is stored in the selected location.

### Memory Performance

- **Access time:** Time between applying an address and obtaining valid data.
- **Cycle time:** Minimum time between the start of two consecutive memory operations.
- **Bandwidth:** Amount of data transferred per unit time.
- **Density:** Number of bits stored per unit chip area.

`Memory cycle time ≥ Access time`

`Bandwidth = Data transferred / Time`

### Volatile and Non-Volatile Memory

| Volatile Memory | Non-Volatile Memory |
|---|---|
| Loses data when power is removed | Retains data without power |
| Used as temporary working memory | Used for permanent storage |
| SRAM and DRAM | ROM, EEPROM, Flash and MRAM |

### Access Methods

- **Random access:** Any location can be accessed in approximately equal time.  
  Examples: SRAM and DRAM.

- **Sequential access:** Data is accessed in a particular sequence.  
  Example: Magnetic tape.

- **Associative access:** Data is searched using its contents rather than its address.  
  Example: Content-addressable memory.

---

## 2. Memory-Chip Organization

A semiconductor memory generally contains:

- Memory-cell array
- Row decoder
- Column decoder and multiplexer
- Word-line drivers
- Bit lines
- Sense amplifiers
- Write drivers
- Control circuitry

### Memory-Cell Array

Memory cells are arranged in a two-dimensional array.

- Rows are selected using word lines.
- Columns are connected using bit lines.
- Each cell stores one bit.
- Multiple cells in a selected row form a memory word.

Large memories are divided into smaller **sub-arrays** to reduce wire length, capacitance and access delay.

### Word Line

A word line runs horizontally through a row of cells.

- The row decoder selects one word line.
- Activating the word line connects the cells in that row to their bit lines.
- Word lines are highly capacitive, so word-line drivers are used to drive them.

### Bit Lines

Bit lines run vertically through the array and carry data between the cells and peripheral circuits.

A memory may use:

- A single bit line
- Differential bit lines `BL` and `BL̅`

Long bit lines have large capacitance. Therefore, memory cells usually produce only a small voltage change, which is detected by a sense amplifier.

### Row Decoder

The row decoder converts the row-address bits into one active word line.

An `n`-bit row address can select one of `2ⁿ` rows.

### Column Decoder and Multiplexer

The column decoder selects the required column or group of columns from the activated row.

If the memory word has `k` bits, the column circuitry selects `k` bit lines and connects them to the input/output circuitry.

### Sense Amplifier

A sense amplifier:

- Detects a small bit-line voltage difference
- Amplifies it into a full logic level
- Improves read speed
- Reduces the strength required from the memory cell

### Write Driver

The write driver places strong logic values on the bit lines during writing.

It must be strong enough to overwrite the old value stored in the selected cell.

### Common Memory Signals

- **Address lines:** Select the required location.
- **Data lines:** Carry input or output data.
- **Chip Select:** Enables the memory chip.
- **Output Enable:** Enables data output during reading.
- **Write Enable:** Controls writing.
- **Clock:** Synchronizes operations in synchronous memories.

### Memory Expansion

#### Increasing the Number of Words

Example:

`Two 1K × 8 chips → One 2K × 8 memory`

- Lower address bits are connected to both chips.
- A higher address bit selects one of the chips.
- The word size remains 8 bits.

#### Increasing the Word Length

Example:

`Two 1K × 4 chips → One 1K × 8 memory`

- Both chips receive the same address.
- Both chips are enabled simultaneously.
- Each chip provides four bits of the eight-bit word.

---

## 3. SRAM

SRAM stands for **Static Random-Access Memory**.

It retains data as long as power is supplied and does not require periodic refresh.

### 3.1 6T SRAM Cell

A standard 6T SRAM cell contains:

- Two cross-coupled CMOS inverters
- Two NMOS access transistors
- Two complementary storage nodes: `Q` and `Q̅`
- Two complementary bit lines: `BL` and `BL̅`
- One word line: `WL`

The cross-coupled inverters form a bistable latch with two stable states:

| Stored Value | Q | Q̅ |
|---|---:|---:|
| Logic 0 | 0 | 1 |
| Logic 1 | 1 | 0 |

The access transistors connect `Q` and `Q̅` to the bit lines when `WL = 1`.

### 3.2 Hold Operation

During hold:

1. `WL = 0`.
2. Both access transistors are OFF.
3. The cell is disconnected from `BL` and `BL̅`.
4. The cross-coupled inverters continuously reinforce each other.
5. The stored value is retained while power is available.

No refresh operation is required.

### 3.3 SRAM Read Operation

Assume the cell stores:

`Q = 0` and `Q̅ = 1`

#### Step 1: Precharge

- `BL` and `BL̅` are precharged to `VDD`.
- The precharge circuit is then disabled.
- Both bit lines are initially at the same voltage.

#### Step 2: Row Selection

- The row decoder activates the required word line.
- `WL` becomes HIGH.
- The access transistors turn ON.
- The internal storage nodes connect to the bit lines.

#### Step 3: Bit-Line Discharge

Because `Q = 0`:

- `BL` obtains a discharge path through the access and pull-down transistors.
- `BL` starts falling slightly below `VDD`.
- `BL̅` remains close to `VDD` because `Q̅ = 1`.

Only a small differential voltage is normally developed:

`ΔV = BL̅ - BL`

#### Step 4: Sense Amplification

- The sense amplifier compares `BL` and `BL̅`.
- It amplifies the small voltage difference into a full digital value.
- Since `BL < BL̅`, the stored value at `Q` is interpreted as `0`.

#### Step 5: Completion

- The word line is deactivated.
- The cell is disconnected from the bit lines.
- The bit lines are precharged again for the next operation.

#### SRAM Read Summary

`Precharge → Activate WL → One bit line discharges → Sense difference → Deactivate WL`

The read is **non-destructive**, but the cell must be designed so that the internal `0` node does not rise enough to flip the cell.

### 3.4 SRAM Write Operation

#### Writing Logic 1 to Q

To write:

`Q = 1` and `Q̅ = 0`

#### Step 1: Drive the Bit Lines

The write drivers force:

- `BL = 1`
- `BL̅ = 0`

Unlike reading, the bit lines are strongly driven during writing.

#### Step 2: Activate the Word Line

- The row decoder makes `WL = 1`.
- The access transistors connect the bit lines to `Q` and `Q̅`.

#### Step 3: Overwrite the Old Value

- The LOW value on `BL̅` pulls `Q̅` toward ground.
- As `Q̅` falls, the cross-coupled inverter drives `Q` HIGH.
- Positive feedback completes the state transition.
- The previous value is overwritten.

#### Step 4: Store the New Value

- The word line is deactivated.
- The write drivers release the bit lines.
- The cross-coupled inverters retain the new value.

To write logic `0` to `Q`:

- `BL = 0`
- `BL̅ = 1`

#### SRAM Write Summary

`Drive BL and BL̅ → Activate WL → Overwrite cell → Deactivate WL`

### 3.5 SRAM Cell Design Requirements

#### Read Stability

During reading, the voltage at the internal `0` node rises slightly because it is connected to a precharged bit line.

The pull-down NMOS must be strong enough relative to the access transistor to prevent the cell from flipping.

#### Write Ability

During writing, the write driver must overcome the previous value stored by the cross-coupled inverters.

The access transistor must be sufficiently strong relative to the pull-up PMOS to write a new value successfully.

#### Hold Stability

When the word line is inactive, the cross-coupled inverters must retain the stored state despite leakage, noise and process variations.

### 3.6 SRAM Characteristics

#### Advantages

- Very fast
- No refresh required
- Non-destructive read
- Suitable for frequent accesses

#### Limitations

- Six transistors per bit
- Large cell area
- Lower density
- Higher cost per bit

#### Applications

- L1, L2 and L3 caches
- Register files
- On-chip buffers
- Small embedded memories

---

## 4. DRAM

DRAM stands for **Dynamic Random-Access Memory**.

It stores data as electrical charge in a capacitor. Since this charge gradually leaks away, DRAM requires periodic refresh.

### 4.1 1T-1C DRAM Cell

A DRAM cell contains:

- One access transistor
- One storage capacitor
- One word line
- One bit line

The access transistor connects the capacitor to the bit line when the word line is activated.

| Capacitor Condition | Stored Value |
|---|---|
| Charged | Logic 1 |
| Discharged | Logic 0 |

This mapping is conventional and can depend on the sensing scheme.

### 4.2 DRAM Write Operation

#### Writing Logic 1

1. The write driver raises the bit line toward `VDD`.
2. The row decoder activates the word line.
3. The access transistor turns ON.
4. Charge flows from the bit line into the storage capacitor.
5. The capacitor becomes charged.
6. The word line is deactivated, isolating the capacitor.

#### Writing Logic 0

1. The write driver pulls the bit line toward ground.
2. The word line is activated.
3. The access transistor turns ON.
4. Charge is removed from the capacitor.
5. The word line is deactivated.
6. The discharged capacitor represents logic `0`.

#### DRAM Write Summary

`Drive bit line → Activate word line → Charge/discharge capacitor → Deactivate word line`

### 4.3 DRAM Read Operation

Reading a DRAM cell is based on charge sharing between the small cell capacitor and the much larger bit-line capacitance.

#### Step 1: Bit-Line Precharge

- The bit line is precharged to approximately `VDD/2`.
- The precharge circuit is then disabled.
- The bit line is left at a known intermediate voltage.

#### Step 2: Row Activation

- The row decoder activates the required word line.
- The access transistor turns ON.
- The storage capacitor connects to the bit line.

#### Step 3: Charge Sharing

The capacitor and bit line share charge.

If the capacitor stores logic `1`:

- Charge flows from the capacitor to the bit line.
- The bit-line voltage rises slightly above `VDD/2`.

If the capacitor stores logic `0`:

- Charge flows from the bit line toward the capacitor.
- The bit-line voltage falls slightly below `VDD/2`.

Therefore:

`Stored 1 → VBL slightly greater than VDD/2`

`Stored 0 → VBL slightly less than VDD/2`

#### Step 4: Sense Amplification

- The sense amplifier compares the bit-line voltage with a reference or complementary bit line.
- It detects whether the voltage is above or below `VDD/2`.
- It amplifies the small difference to either `VDD` or `0`.

#### Step 5: Restoration

Charge sharing disturbs the capacitor's original charge.

While the word line is still active, the sense amplifier drives the full detected value back into the cell capacitor.

This restores the data.

#### Step 6: Precharge

- The word line is deactivated.
- The bit line returns to `VDD/2`.
- The bank becomes ready for another row activation.

#### DRAM Read Summary

`Precharge to VDD/2 → Activate row → Charge sharing → Sense → Restore → Precharge`

### 4.4 Destructive Read

The DRAM read operation is called **destructive** because connecting the storage capacitor to the bit line disturbs its original charge.

Therefore, every read must be followed internally by restoration.

### 4.5 Refresh

The storage capacitor loses charge because of leakage currents.

To preserve data:

1. A row is activated.
2. The sense amplifiers detect the stored values.
3. The values are restored into the cells.
4. This process is repeated periodically for every row.

Refresh:

- Preserves stored data
- Consumes power
- Uses memory cycles
- Is controlled by the memory controller

### 4.6 DRAM Array and Row Buffer

DRAM is organized into banks containing rows and columns.

When a row is activated:

- The entire row is transferred to the sense amplifiers.
- The sense amplifiers temporarily hold the row.
- This collection of sense amplifiers acts as the **row buffer**.

#### Row-Buffer Hit

The requested data belongs to the currently open row.

Only a column access is required, so the access is relatively fast.

#### Row-Buffer Miss

The requested data belongs to another row.

The current row must be:

1. Written back if required
2. Precharged and closed
3. Replaced by activating the new row

Therefore, a row-buffer miss has higher latency.

### 4.7 RAS and CAS

DRAM commonly multiplexes row and column addresses over the same address pins.

- **RAS:** Row Address Strobe; captures the row address.
- **CAS:** Column Address Strobe; captures the column address.

Address multiplexing reduces the number of external pins.

### 4.8 DRAM Characteristics

#### Advantages

- One transistor and one capacitor per bit
- Small cell area
- High density
- Low cost per bit
- Suitable for large memories

#### Limitations

- Requires refresh
- Destructive read
- Slower than SRAM
- More complex control and timing

#### Applications

- Main memory
- Graphics memory
- Server memory
- Mobile memory

---

## 5. ROM and Non-Volatile Memories

ROM stands for **Read-Only Memory**.

- It retains data without power.
- Its contents are normally read during system operation.
- Programming and erasing depend on the ROM type.

### 5.1 ROM Organization

A ROM contains:

- Address decoder
- Memory-cell array
- Output or sensing circuitry

For `n` address inputs:

`Number of addressable words = 2ⁿ`

Each address selects a stored output word.

### 5.2 Mask ROM

- Programmed during fabrication
- Cannot be modified afterward
- Economical for large production volumes
- Used when the stored data is permanently fixed

### 5.3 PROM

PROM stands for **Programmable Read-Only Memory**.

- Initially supplied unprogrammed
- Programmed once by the user
- Uses fuses or antifuses
- Cannot normally be erased

### 5.4 EPROM

EPROM stands for **Erasable Programmable Read-Only Memory**.

- Electrically programmed
- Erased using ultraviolet light
- The complete chip is generally erased together
- Can be reprogrammed after erasure

### 5.5 EEPROM

EEPROM stands for **Electrically Erasable Programmable Read-Only Memory**.

- Electrically programmed and erased
- Usually supports byte-level writing and erasure
- Can be reprogrammed inside the system
- Writing is slower than reading
- Supports a limited number of program/erase cycles

EEPROM is commonly used for:

- Configuration data
- Calibration values
- Device settings
- Small amounts of firmware

---

## 6. Flash Memory

Flash memory is an electrically programmable and erasable non-volatile memory.

It is a type of EEPROM, but unlike EEPROM, Flash erases data in blocks rather than individual bytes.

### Floating-Gate Cell

A Flash cell commonly uses a floating-gate MOSFET.

- The floating gate is electrically isolated by an oxide layer.
- Electrons trapped on the floating gate remain present even without power.
- The stored charge changes the transistor's threshold voltage.

An erased cell has a lower threshold voltage and is conventionally read as logic `1`.

A programmed cell contains trapped electrons, has a higher threshold voltage and is conventionally read as logic `0`.

### Read Operation

1. A read voltage is applied to the control gate.
2. A small voltage is applied between the source and drain.
3. The sense amplifier checks whether the transistor conducts.
4. The conduction state determines the stored value.

Reading does not remove the charge and is therefore non-destructive.

### Program Operation

- A high voltage is used to place electrons onto the floating gate.
- The trapped electrons increase the threshold voltage.
- The programmed cell is conventionally read as logic `0`.

### Erase Operation

- A high electric field removes electrons from the floating gate.
- The threshold voltage decreases.
- The erased cell is conventionally read as logic `1`.
- Flash memory is erased in blocks or sectors.

### NAND vs NOR Flash

| Feature | NAND Flash | NOR Flash |
|---|---|---|
| Cell connection | Series | Parallel |
| Access | Page-oriented | Random byte/word access |
| Density | Higher | Lower |
| Cost per bit | Lower | Higher |
| Direct code execution | Generally not supported | Supported |
| Main use | SSDs, USB drives and mobile storage | Firmware and boot code |

---

## 7. Programmable Logic Devices

A **Programmable Logic Device (PLD)** is an integrated circuit whose logic connections can be programmed to implement Boolean functions.

Basic PLDs use:

1. An AND array to generate product terms
2. An OR array to combine product terms

For example:

`F = A̅B + AC`

The AND array generates `A̅B` and `AC`, and the OR array combines them.

####The difference between PROM, PLA and PAL depends on which logic arrays are programmable:

- **PROM:** Fixed AND array and programmable OR array.  
  The fixed decoder generates all minterms, and the OR array selects the required ones. It can implement any combinational function but may generate unnecessary minterms.

- **PLA (Programmable Logic Array):** Programmable AND and OR arrays.  
  It generates only the required product terms and allows sharing between outputs, making it the most flexible but more complex.

- **PAL (Programmable Array Logic):** Programmable AND array and fixed OR array.  
  It is simpler and generally faster than PLA but has limited product terms and flexibility.

| Device | AND Array | OR Array | Main Feature |
|---|---|---|---|
| PROM | Fixed | Programmable | Generates all minterms |
| PLA | Programmable | Programmable | Most flexible |
| PAL | Programmable | Fixed | Simpler and faster |

`Flexibility: PLA > PAL > PROM`

---

## 8. Memory Timing and Power

### 8.1 Important Timing Parameters

- **Read access time:** Time from applying the address to receiving valid data.
- **Write cycle time:** Minimum time required to complete a write.
- **Setup time:** Input must be stable before the active clock or control edge.
- **Hold time:** Input must remain stable after the active edge.
- **Cycle time:** Minimum time between consecutive operations.

Violating timing requirements can cause incorrect or unreliable memory operation.

### 8.2 Dynamic Power

Dynamic power is consumed when circuit nodes switch:

`Pdynamic = αCV²f`

Where:

- `α` = Switching activity
- `C` = Switched capacitance
- `V` = Supply voltage
- `f` = Operating frequency

Major sources include:

- Bit-line charging and discharging
- Word-line switching
- Decoders
- Sense amplifiers
- Write drivers
- Input/output circuits
- DRAM refresh operations

### 8.3 Leakage Power

Leakage power is consumed even when the memory is inactive:

`Pleakage = Ileakage × V`

Leakage is significant in large memories because millions of cells contribute leakage current.

### 8.4 Active and Standby Power

- **Active power:** Consumed during read, write and switching operations.
- **Standby power:** Consumed while the memory is powered but inactive.
- **Refresh power:** Consumed by DRAM during periodic refresh.

### 8.5 Power-Reduction Techniques

- Reduce supply voltage.
- Reduce unnecessary bit-line and word-line switching.
- Activate only the required memory bank or sub-array.
- Apply clock gating to peripheral circuits.
- Power-gate unused memory blocks.
- Use low-leakage transistors.
- Use DRAM self-refresh and low-power modes.
- Reduce unnecessary row activations.

---

## 9. Memory Redundancy

Memory arrays contain a very large number of cells, so even a small manufacturing defect can produce faulty bits.

Instead of discarding the entire chip, additional rows and columns are included to replace defective ones.

During manufacturing testing:

1. Faulty rows or columns are identified.
2. Their addresses are stored using fuses or e-fuses.
3. Future accesses to those addresses are redirected to spare rows or columns.

### Benefits

- Improves manufacturing yield
- Prevents an entire memory from being discarded due to a few defective cells
- Reduces manufacturing cost
- Improves reliability

Memory faults are commonly detected using memory test techniques such as MBIST and March algorithms.

---

## 10. Important Comparisons

### RAM vs ROM

| Feature | RAM | ROM |
|---|---|---|
| Normal operations | Read and write | Mainly read |
| Volatility | Generally volatile | Non-volatile |
| Modification | Easy and frequent | Fixed or specially programmed |
| Main use | Working memory | Firmware and permanent data |

### SRAM vs DRAM

| Feature | SRAM | DRAM |
|---|---|---|
| Storage element | Bistable latch | Capacitor |
| Typical cell | 6T | 1T-1C |
| Refresh | Not required | Required |
| Read | Non-destructive | Destructive |
| Speed | Faster | Slower |
| Density | Lower | Higher |
| Cost per bit | Higher | Lower |
| Main application | Cache | Main memory |

### ROM Types

| Memory | Programming | Erasing |
|---|---|---|
| Mask ROM | During manufacturing | Not possible |
| PROM | Once by the user | Not possible |
| EPROM | Electrically | Using ultraviolet light |
| EEPROM | Electrically | Electrically, usually byte-wise |
| Flash | Electrically | Electrically, block-wise |
