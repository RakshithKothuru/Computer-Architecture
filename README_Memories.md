# Semiconductor Memories

## Index

1. [Memory Fundamentals](#1-memory-fundamentals)
2. [Memory-Chip Organization](#2-memory-chip-organization)
3. [SRAM](#3-sram)
4. [DRAM](#4-dram)
5. [ROM and Non-Volatile Memories](#5-rom-and-non-volatile-memories)
6. [Flash Memory](#6-flash-memory)
7. [Programmable Logic Devices](#7-programmable-logic-devices)
8. [Modern DRAM Concepts](#8-modern-dram-concepts)
9. [Memory Timing and Power](#9-memory-timing-and-power)
10. [Memory Redundancy](#10-memory-redundancy)
11. [Important Comparisons](#11-important-comparisons)

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

### 5.6 MRAM

MRAM stands for **Magnetoresistive Random-Access Memory**.

Unlike SRAM, DRAM and Flash, it stores data using magnetic states rather than electric charge.

A common MRAM cell contains:

- Magnetic tunnel junction
- Fixed magnetic layer
- Free magnetic layer
- Tunnel barrier
- Access transistor

The relative orientation of the magnetic layers determines resistance:

- Parallel orientation → Low resistance
- Antiparallel orientation → High resistance

MRAM is:

- Non-volatile
- Fast
- Highly durable
- Suitable for some embedded-memory applications

---

## 6. Flash Memory

Flash memory is an electrically programmable and erasable non-volatile memory.

It is a form of EEPROM in which data is erased in blocks rather than one byte at a time.

### 6.1 Floating-Gate Flash Cell

A Flash cell is commonly implemented using a floating-gate MOSFET.

It contains:

- Control gate
- Electrically isolated floating gate
- Insulating oxide
- Source
- Drain
- Channel

The floating gate is surrounded by insulation, so trapped electrons can remain for years without power.

### 6.2 Threshold-Voltage Storage

The charge stored on the floating gate changes the transistor's threshold voltage.

#### Erased Cell

- Little or no negative charge is trapped.
- Threshold voltage is relatively low.
- The transistor turns ON during a read.
- It is conventionally interpreted as logic `1`.

#### Programmed Cell

- Electrons are trapped on the floating gate.
- Threshold voltage increases.
- The transistor remains OFF at the normal read voltage.
- It is conventionally interpreted as logic `0`.

Therefore:

`Erased cell → Lower Vth → Conducts → Logic 1`

`Programmed cell → Higher Vth → Does not conduct → Logic 0`

### 6.3 Flash Read Operation

1. A read voltage is applied to the selected word line.
2. A small voltage is applied across the source and drain.
3. The sense amplifier checks whether drain current flows.
4. The current indicates the cell's threshold-voltage range.
5. The corresponding logic value is produced.

The read operation does not remove charge from the floating gate and is therefore non-destructive.

### 6.4 Flash Program Operation

Programming normally places electrons onto the floating gate.

1. High programming voltages are generated internally using charge pumps.
2. Suitable voltages are applied to the control gate and drain.
3. Electrons enter the floating gate through hot-electron injection or tunnelling.
4. The trapped charge increases the threshold voltage.
5. The programmed cell is conventionally read as logic `0`.

Flash generally cannot directly change a programmed `0` back to `1`. The corresponding erase block must first be erased.

### 6.5 Flash Erase Operation

1. A high electric field is applied across the tunnel oxide.
2. Electrons tunnel out of the floating gate.
3. The threshold voltage decreases.
4. The erased cell is conventionally read as logic `1`.

Flash erasure is performed over a block or sector rather than an individual bit.

### 6.6 SLC, MLC, TLC and QLC

A Flash cell can store one or more bits by using multiple threshold-voltage ranges.

| Type | Bits per Cell | Number of States | Main Characteristic |
|---|---:|---:|---|
| SLC | 1 | 2 | Fastest and most reliable |
| MLC | 2 | 4 | Higher density |
| TLC | 3 | 8 | Common in mass storage |
| QLC | 4 | 16 | Highest density but lower endurance |

As the number of bits per cell increases:

- Density increases.
- Cost per bit decreases.
- Programming becomes more complex.
- Read margins decrease.
- Endurance and reliability generally decrease.

### 6.7 NAND Flash

In NAND Flash:

- Cells are connected in series to form a NAND string.
- Multiple cells share one bit line.
- Select transistors connect the string to the bit line and ground.
- Unselected cells must conduct during the reading of a selected cell.

NAND Flash is organized hierarchically:

1. Cell
2. Page
3. Block
4. Plane
5. Die

Typical operations:

- Read at page level
- Program at page level
- Erase at block level

#### Advantages

- High density
- Small area per bit
- Low cost per bit
- Efficient page and block operations

#### Applications

- SSDs
- USB drives
- Memory cards
- Mobile internal storage

### 6.8 NOR Flash

In NOR Flash:

- Cells are connected in parallel to bit lines.
- Individual words or bytes can be accessed directly.
- It provides faster random reads than NAND Flash.
- It supports execute-in-place operation.

#### Advantages

- Fast random access
- Direct code execution
- Suitable for firmware and boot code

#### Limitations

- Lower density than NAND
- Higher cost per bit
- Slower erase and program operations

### 6.9 NAND vs NOR Flash

| Feature | NAND Flash | NOR Flash |
|---|---|---|
| Cell connection | Series | Parallel |
| Read access | Page-oriented | Random byte/word access |
| Density | Higher | Lower |
| Cost per bit | Lower | Higher |
| Program unit | Page | Byte or word |
| Erase unit | Block | Sector |
| Execute in place | Generally not supported | Supported |
| Main use | Mass storage | Firmware and boot code |

---

## 7. Programmable Logic Devices

A **Programmable Logic Device (PLD)** is an integrated circuit whose logic connections can be programmed to implement Boolean functions.

Basic PLDs use:

1. An AND array to generate product terms
2. An OR array to combine product terms

For example:

`F = A̅B + AC`

The AND array generates `A̅B` and `AC`, and the OR array combines them.

### 7.1 PROM as a PLD

In a PROM:

- AND array is fixed.
- OR array is programmable.

The fixed decoder generates all possible minterms. The programmable OR array selects the required minterms for each output.

#### Advantages

- Can implement any combinational function
- Directly implements a truth table

#### Limitation

- Generates every possible minterm even when only a few are required

### 7.2 PLA

PLA stands for **Programmable Logic Array**.

In a PLA:

- AND array is programmable.
- OR array is programmable.

Only the required product terms are generated, and product terms can be shared between outputs.

#### Advantages

- Most flexible basic PLD
- Efficient product-term generation
- Supports product-term sharing

#### Limitations

- More complex
- Generally slower than PAL

### 7.3 PAL

PAL stands for **Programmable Array Logic**.

In a PAL:

- AND array is programmable.
- OR array is fixed.

#### Advantages

- Simpler than PLA
- Generally faster than PLA
- Easier to implement

#### Limitations

- Less flexible than PLA
- Limited product terms per output
- Restricted product-term sharing

### 7.4 PROM vs PLA vs PAL

| Device | AND Array | OR Array | Main Characteristic |
|---|---|---|---|
| PROM | Fixed | Programmable | Generates all minterms |
| PLA | Programmable | Programmable | Maximum flexibility |
| PAL | Programmable | Fixed | Simpler and faster |

`Flexibility: PLA > PAL > PROM`

---

## 8. Modern DRAM Concepts

### 8.1 SDRAM

SDRAM stands for **Synchronous Dynamic Random-Access Memory**.

Its commands and data transfers are synchronized with a clock, enabling:

- Predictable timing
- Pipelined operations
- Burst transfers

### 8.2 DDR SDRAM

DDR stands for **Double Data Rate**.

DDR transfers data on both:

- Rising clock edge
- Falling clock edge

A `1 GHz` memory clock therefore provides approximately `2 billion transfers per second` per data pin.

### 8.3 Banks, Rows and Columns

DRAM is divided into multiple banks. Each bank contains:

- Rows
- Columns
- Sense amplifiers
- A row buffer

A basic access involves:

1. **ACTIVATE:** Opens a row.
2. **READ/WRITE:** Selects columns from the open row.
3. **PRECHARGE:** Closes the row.

Multiple banks allow commands to overlap and improve throughput.

### 8.4 Burst Operation

A burst transfers several consecutive data units after one read or write command.

Benefits include:

- Reduced command overhead
- Higher bandwidth
- Efficient cache-line transfers

### 8.5 Prefetch Architecture

DDR memories internally access multiple data bits at once and transfer them through faster external pins.

Examples:

- DDR: `2n` prefetch
- DDR2: `4n` prefetch
- DDR3: `8n` prefetch
- DDR4: `8n` prefetch

Prefetch increases external data rate without requiring the internal cell array to operate at the same frequency.

### 8.6 Latency and Bandwidth

- **Latency:** Time needed to complete an individual memory access.
- **Bandwidth:** Amount of data transferred per unit time.

A memory can have high bandwidth but still have significant access latency.

### 8.7 Memory Controller

The memory controller performs:

- Address mapping
- Command generation
- Read and write scheduling
- Refresh management
- Bank management
- Timing-constraint enforcement
- Data-transfer coordination

---

## 9. Memory Timing and Power

### 9.1 Important Timing Parameters

- **Read access time:** Time from applying the address to receiving valid data.
- **Write cycle time:** Minimum time required to complete a write.
- **Setup time:** Input must be stable before the active clock or control edge.
- **Hold time:** Input must remain stable after the active edge.
- **Cycle time:** Minimum time between consecutive operations.

Violating timing requirements can cause incorrect or unreliable memory operation.

### 9.2 Dynamic Power

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

### 9.3 Leakage Power

Leakage power is consumed even when the memory is inactive:

`Pleakage = Ileakage × V`

Leakage is significant in large memories because millions of cells contribute leakage current.

### 9.4 Active and Standby Power

- **Active power:** Consumed during read, write and switching operations.
- **Standby power:** Consumed while the memory is powered but inactive.
- **Refresh power:** Consumed by DRAM during periodic refresh.

### 9.5 Power-Reduction Techniques

- Reduce supply voltage.
- Reduce unnecessary bit-line and word-line switching.
- Activate only the required memory bank or sub-array.
- Apply clock gating to peripheral circuits.
- Power-gate unused memory blocks.
- Use low-leakage transistors.
- Use DRAM self-refresh and low-power modes.
- Reduce unnecessary row activations.

---

## 10. Memory Redundancy

Memory arrays contain a very large number of cells, so even a small manufacturing defect can produce faulty bits.

Instead of discarding the entire chip, additional rows and columns are included to replace defective ones.

### Row Redundancy

If one or more cells in a row are faulty:

1. Testing identifies the faulty row address.
2. The address is stored using fuses, e-fuses or repair registers.
3. Incoming addresses are compared with the stored faulty address.
4. A matching address is redirected to a spare row.

### Column Redundancy

If a column or bit line is faulty:

1. Testing identifies the faulty column.
2. The faulty column is disconnected.
3. A spare column is selected in its place.

### Benefits

- Improves manufacturing yield
- Prevents an entire memory from being discarded due to a few defective cells
- Reduces manufacturing cost
- Improves reliability

Memory faults are commonly detected using memory test techniques such as MBIST and March algorithms.

---

## 11. Important Comparisons

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
