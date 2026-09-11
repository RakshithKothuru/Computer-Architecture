# Semiconductor Memories

## Index

1. [Memory Basics](#1-memory-basics)
2. [Memory Organization](#2-memory-organization)
3. [RAM](#3-ram)
4. [SRAM](#4-sram)
5. [DRAM](#5-dram)
6. [ROM](#6-rom)
7. [Flash Memory](#8-flash-memory)
8. [Programmable Logic Devices](#9-programmable-logic-devices)
9. [Modern DRAM](#11-modern-dram)
10. [Memory Timing and Power](#12-memory-timing-and-power)
11. [Memory Expansion](#13-memory-expansion)
12. [Important Comparisons](#14-important-comparisons)

---

## 1. Memory Basics

A semiconductor memory stores binary information using electronic circuits.

### Memory Hierarchy

1. Registers
2. Cache
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

Total capacity:

`1024 × 8 = 8192 bits`

For `N` memory locations:

`Number of address lines = log₂(N)`

For `M` bits per word:

`Number of data lines = M`

### Basic Memory Operations

#### Read

1. Apply the address.
2. Enable the memory chip.
3. Activate the read control.
4. Read data from the output.

A read operation normally does not change the stored data.

#### Write

1. Apply the address.
2. Apply the input data.
3. Enable the memory chip.
4. Activate the write control.
5. Store the data in the selected location.

### Memory Characteristics

- **Access time:** Time from applying an address until valid data becomes available.
- **Cycle time:** Minimum time between two consecutive memory operations.
- **Bandwidth:** Amount of data transferred per unit time.
- **Density:** Number of bits stored per unit chip area.
- **Volatility:** Whether the memory retains data without power.

`Memory cycle time ≥ Access time`

`Bandwidth = Data transferred / Time`

### Volatile vs Non-Volatile Memory

| Volatile Memory | Non-Volatile Memory |
|---|---|
| Loses data when power is removed | Retains data without power |
| Used as temporary working memory | Used for permanent storage |
| SRAM and DRAM | ROM, EEPROM and Flash |

### Access Methods

- **Random access:** Any location can be accessed in approximately equal time.
- **Sequential access:** Locations are accessed in a fixed sequence.
- **Associative access:** Data is searched using its contents instead of its address.

---

## 2. Memory Organization

A memory chip contains:

- Memory-cell array
- Row decoder
- Column decoder
- Word lines
- Bit lines
- Sense amplifiers
- Write drivers
- Control circuitry

### Memory-Cell Array

- Memory cells are arranged in rows and columns.
- Each cell normally stores one bit.
- A row generally contains one or more memory words.

### Word Line

A word line selects a row of memory cells.

When a word line is activated, the cells connected to that row can be accessed.

### Bit Lines

Bit lines transfer data between the memory cells and peripheral circuits.

A memory may use:

- A single bit line
- Complementary bit lines `BL` and `BL̅`

### Row Decoder

The row decoder receives the row-address bits and activates one word line.

An `n`-bit decoder can select one of `2ⁿ` rows.

### Column Decoder

The column decoder selects the required bit or group of bits from the activated row.

### Sense Amplifier

The memory cell may produce only a small voltage change on the bit line.

The sense amplifier:

- Detects the small voltage difference
- Amplifies it into a full logic level
- Improves read speed and reliability

### Write Driver

The write driver places strong logic values on the bit lines and forces the selected cell to store the required data.

### Common Memory Signals

- **Address lines:** Select the memory location.
- **Data lines:** Carry input or output data.
- **Chip Select:** Enables the memory chip.
- **Output Enable:** Enables the output during reading.
- **Write Enable:** Controls the write operation.

---

## 3. RAM

RAM stands for **Random-Access Memory**.

- Any memory location can be accessed directly.
- RAM supports both read and write operations.
- It is generally volatile.

The two major types are:

1. SRAM
2. DRAM

---

## 4. SRAM

SRAM stands for **Static Random-Access Memory**.

It stores data using a bistable latch and does not require periodic refresh while power is present.

### 6T SRAM Cell

A standard SRAM cell contains six transistors:

- Four transistors form two cross-coupled inverters.
- Two access transistors connect the cell to `BL` and `BL̅`.
- A word line controls the access transistors.

### Hold Operation

- Word line is LOW.
- Access transistors are OFF.
- The cell is isolated from the bit lines.
- Cross-coupled inverters retain the stored value.

### Read Operation

1. `BL` and `BL̅` are precharged.
2. The word line is activated.
3. The stored value slightly discharges one bit line.
4. The sense amplifier detects the difference between the bit lines.
5. The output is produced.

The read operation is non-destructive.

### Write Operation

1. The write driver applies complementary values to `BL` and `BL̅`.
2. The word line is activated.
3. The bit-line voltages force the internal nodes to the new state.
4. The word line is deactivated.
5. The cell retains the new value.

### SRAM Cell Requirements

- **Read stability:** Reading should not accidentally flip the stored value.
- **Write ability:** The write driver should be able to overwrite the old value.
- **Hold stability:** The cell should retain data when it is not selected.

### Advantages

- Very fast
- No refresh required
- Non-destructive read
- Simple interface

### Limitations

- Large cell area
- Lower density
- Higher cost per bit

### Applications

- Processor caches
- Register files
- On-chip memories
- Buffers

---

## 5. DRAM

DRAM stands for **Dynamic Random-Access Memory**.

It stores data as charge in a capacitor.

### 1T–1C DRAM Cell

A DRAM cell contains:

- One access transistor
- One storage capacitor

The word line controls the transistor, and the bit line is used for reading and writing.

### Write Operation

1. Data is applied to the bit line.
2. The word line is activated.
3. The capacitor is charged or discharged.
4. The word line is deactivated.

The capacitor's charged and discharged conditions represent binary values.

### Read Operation

1. The bit line is precharged.
2. The word line is activated.
3. Charge sharing occurs between the capacitor and bit line.
4. A small voltage change appears on the bit line.
5. The sense amplifier detects the stored value.
6. The value is restored to the capacitor.

### Destructive Read

Reading disturbs the charge stored in the capacitor.

Therefore:

- DRAM reading is destructive.
- The read value must be written back after sensing.
- This process is called restoration.

### Refresh

The capacitor gradually loses charge because of leakage.

Therefore, DRAM cells must be periodically read and restored. This process is called **refresh**.

Refresh:

- Preserves stored data
- Consumes power
- Uses memory cycles
- Is controlled by the memory controller

### DRAM Array and Row Buffer

- DRAM is arranged into rows and columns.
- Activating a row transfers its contents to the sense amplifiers.
- These sense amplifiers collectively act as the row buffer.
- A column is then selected from the active row.

Accessing the already-open row is faster than opening a different row.

### RAS and CAS

DRAM can use the same pins for row and column addresses.

- **RAS:** Row Address Strobe
- **CAS:** Column Address Strobe

This reduces the number of external address pins.

### Advantages

- Small cell area
- High density
- Low cost per bit
- Suitable for large memories

### Limitations

- Slower than SRAM
- Requires refresh
- Destructive read
- More complex control

### Applications

- Main memory
- Graphics memory
- Large memory systems

---

## 6. ROM

ROM stands for **Read-Only Memory**.

- It is non-volatile.
- It retains data without power.
- During normal operation, its contents are mainly read.
- Programming and erasing depend on the ROM type.

### ROM Structure

A ROM can be viewed as:

- A fixed decoder generating minterms
- A stored connection array
- Output circuitry

For `n` inputs:

`Number of decoder outputs = 2ⁿ`

Each address selects one stored output word.

### ROM as a Logic Device

ROM can implement combinational logic.

- Input variables act as address lines.
- The decoder generates all possible minterms.
- Stored connections combine the required minterms.
- Data outputs represent the required Boolean functions.

In a ROM:

- The AND array is fixed as a decoder.
- The OR array is programmable.

##  Types of ROM

### Mask ROM

- Programmed during manufacturing
- Cannot be modified later
- Suitable for large-volume production
- Low cost per bit in mass production

### PROM

PROM stands for **Programmable Read-Only Memory**.

- Supplied initially unprogrammed
- Programmed once by the user
- Uses fuses or antifuses
- Cannot normally be erased

### EPROM

EPROM stands for **Erasable Programmable Read-Only Memory**.

- Electrically programmed
- Erased using ultraviolet light
- The entire chip is usually erased
- Can be reprogrammed after erasure

### EEPROM

EEPROM stands for **Electrically Erasable Programmable Read-Only Memory**.

- Electrically programmed and erased
- Supports byte-level erasure
- Writing is slower than reading
- Has a limited number of write cycles

### Flash Memory

Flash is a type of EEPROM.

- Electrically programmed
- Electrically erased
- Erased in blocks instead of individual bytes
- Offers high density and low cost per bit

---

## 7. Flash Memory

Flash memory is widely used for non-volatile storage.

### Applications

- Mobile internal storage
- Solid-state drives
- Memory cards
- USB drives
- Firmware storage

### NOR Flash

- Fast random read
- Supports direct code execution
- Lower density
- Higher cost per bit
- Commonly used for firmware

### NAND Flash

- High density
- Lower cost per bit
- Efficient block reading and writing
- Commonly used for mass storage
- Data is generally copied to RAM before execution

### NOR vs NAND Flash

| Feature | NOR Flash | NAND Flash |
|---|---|---|
| Random read | Faster | Slower |
| Block operations | Slower | Faster |
| Density | Lower | Higher |
| Cost per bit | Higher | Lower |
| Direct code execution | Supported | Generally not supported |
| Main application | Firmware | Mass storage |

---

## 8. Programmable Logic Devices

A **Programmable Logic Device (PLD)** is an integrated circuit whose internal logic connections can be programmed to implement digital functions.

Instead of connecting many individual logic gates, the required Boolean functions are programmed into a single device.

### Basic PLD Structure

PLDs commonly use two logic arrays:

1. **AND array:** Generates product terms.
2. **OR array:** Combines product terms to form sum-of-products expressions.

Example:

`F = A̅B + AC`

The AND array generates:

- `A̅B`
- `AC`

The OR array combines them to produce `F`.

### Types of PLDs

Basic PLDs include:

- PROM
- PLA
- PAL

More advanced devices include:

- CPLD
- FPGA

---

## PROM, PLA and PAL

### PROM as a PLD

In a PROM:

- AND array is fixed.
- OR array is programmable.

The fixed decoder generates all possible minterms. The programmable OR array selects the minterms required for each output.

#### Advantages

- Can implement any combinational function
- Simple structure
- Useful for truth-table implementation

#### Limitations

- Generates all possible minterms
- May waste hardware when only a few product terms are required

### PLA

PLA stands for **Programmable Logic Array**.

In a PLA:

- AND array is programmable.
- OR array is programmable.

Only the required product terms are generated and combined.

#### Advantages

- Highly flexible
- Generates only the required product terms
- Product terms can be shared between outputs

#### Limitations

- More complex
- Generally slower and costlier than PAL

### PAL

PAL stands for **Programmable Array Logic**.

In a PAL:

- AND array is programmable.
- OR array is fixed.

The required product terms are generated using the programmable AND array. The fixed OR array limits how those product terms can be combined.

#### Advantages

- Simpler than PLA
- Faster than PLA
- Easier to manufacture

#### Limitations

- Less flexible than PLA
- Limited number of product terms for each output
- Product-term sharing is restricted

### PROM vs PLA vs PAL

| Device | AND Array | OR Array | Main Feature |
|---|---|---|---|
| PROM | Fixed | Programmable | Generates all minterms |
| PLA | Programmable | Programmable | Most flexible |
| PAL | Programmable | Fixed | Faster and simpler |

### Flexibility Order

`PLA > PAL > PROM`

PLA is generally the most flexible because both arrays are programmable.

### Complexity and Speed

- PROM may waste hardware by generating every minterm.
- PLA provides maximum flexibility but has two programmable arrays.
- PAL is faster and simpler because only the AND array is programmable.

---

## 9. Modern DRAM

### SDRAM

SDRAM stands for **Synchronous Dynamic Random-Access Memory**.

Its operations are synchronized with a clock, allowing:

- Predictable timing
- Pipelined operations
- Burst transfers

### DDR SDRAM

DDR stands for **Double Data Rate**.

DDR transfers data on both:

- Rising edge of the clock
- Falling edge of the clock

Thus, a `1 GHz` clock can provide approximately `2 billion transfers per second`.

### Banks, Rows and Columns

DRAM is divided into multiple banks. Each bank contains rows, columns and a row buffer.

A typical DRAM access involves:

1. **Activate:** Open the required row.
2. **Read/Write:** Access the required columns.
3. **Precharge:** Close the row before opening another row in that bank.

Multiple banks allow operations to overlap and improve bandwidth.

### Burst Operation

A burst transfers multiple consecutive data units after a single read or write command.

Advantages:

- Reduced command overhead
- Higher bandwidth
- Efficient cache-line transfers

### Prefetch Architecture

DDR internally fetches multiple bits and sends them sequentially through faster external data pins.

Examples:

- DDR: `2n` prefetch
- DDR2: `4n` prefetch
- DDR3: `8n` prefetch
- DDR4: `8n` prefetch

### Latency and Bandwidth

- **Latency:** Time required to complete one memory access.
- **Bandwidth:** Amount of data transferred per unit time.

A memory can have high bandwidth while still having relatively high access latency.

### Memory Controller

The memory controller connects the processor or SoC to DRAM.

It performs:

- Address mapping
- Command generation
- Read and write scheduling
- Refresh control
- Bank management
- Timing-constraint enforcement
- Data-transfer coordination

---

## 10. Memory Timing and Power

### Read Timing

1. Address and control signals are applied.
2. The selected row and column are activated.
3. The memory cell affects the bit line.
4. The sense amplifier detects the value.
5. Valid data appears at the output.

**Read access time** is the time between applying the address and receiving valid output data.

### Write Timing

1. Address and input data are applied.
2. Write enable is activated.
3. The selected cell stores the data.
4. Address and data are kept stable for the required duration.

### Setup and Hold Time

- **Setup time:** Minimum time for which an input must be stable before the active control or clock edge.
- **Hold time:** Minimum time for which the input must remain stable after the active edge.

Violating these requirements can cause an incorrect or unreliable memory operation.

### Dynamic Power

Dynamic power is consumed when memory nodes switch.

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
- Input/output circuits

### Leakage Power

Leakage power is consumed even when the memory is inactive.

`Pleakage = Ileakage × V`

Leakage is significant in large memories because they contain millions of cells.

### Active and Standby Power

- **Active power:** Consumed during read, write, refresh and switching.
- **Standby power:** Consumed while the memory is powered but inactive.
- **Refresh power:** Consumed by DRAM during periodic refresh.

### Power-Reduction Techniques

- Reduce the supply voltage.
- Reduce unnecessary switching.
- Activate only the required bank.
- Apply clock gating to peripheral circuits.
- Power-gate unused memory blocks.
- Use low-leakage transistors.
- Use DRAM self-refresh modes.
- Reduce unnecessary row activations.

---

## 11. Memory Expansion

### Increasing the Number of Words

Memory chips are connected to increase the number of addressable locations.

Example:

`Two 1K × 8 chips → One 2K × 8 memory`

- Lower address bits select a location inside each chip.
- Higher address bits select the required chip.
- Data width remains unchanged.

### Increasing the Word Length

Memory chips are connected in parallel to increase the number of bits in each word.

Example:

`Two 1K × 4 chips → One 1K × 8 memory`

- Both chips receive the same address.
- Both chips are enabled together.
- Each chip provides four bits of the eight-bit word.

### General Capacity

For a memory organized as `N × M`:

`Total capacity = N × M bits`

`Address lines = log₂(N)`

`Data lines = M`

---

## 12. Important Comparisons

### RAM vs ROM

| Feature | RAM | ROM |
|---|---|---|
| Normal operations | Read and write | Mainly read |
| Volatility | Generally volatile | Non-volatile |
| Data modification | Easy and frequent | Fixed or specially programmed |
| Applications | Working memory | Firmware and fixed data |

### SRAM vs DRAM

| Feature | SRAM | DRAM |
|---|---|---|
| Storage element | Latch | Capacitor |
| Cell structure | Typically 6T | 1T–1C |
| Refresh | Not required | Required |
| Read | Non-destructive | Destructive |
| Speed | Higher | Lower |
| Density | Lower | Higher |
| Cost per bit | Higher | Lower |
| Main application | Cache | Main memory |

### ROM Types

| Memory | Programming | Erasing |
|---|---|---|
| Mask ROM | During manufacturing | Not possible |
| PROM | Once by user | Not possible |
| EPROM | Electrically | UV light |
| EEPROM | Electrically | Electrically, usually byte-wise |
| Flash | Electrically | Electrically, block-wise |
