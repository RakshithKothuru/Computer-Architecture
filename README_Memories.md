# Semiconductor Memories

## 1. Memory Fundamentals

### Memory Hierarchy

Memory is arranged based on speed, capacity and cost:

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

`1K × 8` means:

- 1024 words
- 8 bits per word
- 10 address lines
- 8 data lines

Total capacity:

`1024 × 8 = 8192 bits`

For `N` memory locations:

`Number of address lines = log₂(N)`

### Read Operation

1. Apply the required address.
2. Enable the memory chip.
3. Activate the read control.
4. Data from the selected location appears at the output.

A read operation normally does not modify the stored data.

### Write Operation

1. Apply the required address.
2. Apply the input data.
3. Enable the memory chip.
4. Activate the write control.
5. The data is stored at the selected location.

### Memory Performance

- **Access time:** Time between applying an address and obtaining valid data.
- **Memory cycle time:** Minimum time between the beginning of two consecutive memory operations.
- **Bandwidth:** Amount of data transferred per unit time.

`Memory cycle time ≥ Access time`

`Bandwidth = Data transferred / Time`

### Volatile and Non-Volatile Memory

| Volatile Memory | Non-Volatile Memory |
|---|---|
| Loses data when power is removed | Retains data without power |
| Used as working memory | Used for long-term storage |
| SRAM and DRAM | ROM, EEPROM and Flash |

### Access Methods

- **Random access:** Any location can be accessed in approximately equal time.  
  Examples: SRAM and DRAM.

- **Sequential access:** Data is accessed in a fixed sequence.  
  Example: Magnetic tape.

- **Associative access:** Data is searched using its contents instead of its address.  
  Example: Content-addressable memory.

---

## 2. Memory-Chip Organization

### Memory-Cell Array

Memory cells are arranged in rows and columns.

- Each cell normally stores one bit.
- A row generally represents a memory word.
- The row-column structure reduces decoding and wiring complexity.

### Word Line

A word line selects a row of memory cells.

When a word line is activated, the cells connected to that row become accessible.

### Bit Lines

Bit lines transfer data between the memory cells and peripheral circuits.

A memory may use:

- A single bit line
- Two complementary bit lines: `BL` and `BL̅`

### Row and Column Decoders

The address selects the required memory location.

- **Row decoder:** Activates one word line.
- **Column decoder:** Selects the required column or group of columns.

An `n`-bit decoder can select one of `2ⁿ` outputs.

### Sense Amplifier

A memory cell may produce only a small voltage change on the bit line.

The sense amplifier:

- Detects the small voltage difference
- Amplifies it to a full logic level
- Improves reading speed and reliability

### Write Driver

The write driver places strong logic values on the bit lines during a write operation.

It forces the selected memory cell to store the required value.

### Memory Signals

Common memory signals include:

- **Address lines:** Select a memory location.
- **Data lines:** Carry input or output data.
- **Chip Select/Enable:** Activates the memory chip.
- **Read Enable/Output Enable:** Enables the output during reading.
- **Write Enable:** Controls the write operation.

### Memory Expansion

#### Increasing the Number of Words

Multiple memory chips can be used to increase the number of stored words.

Example:

`Two 1K × 8 chips → One 2K × 8 memory`

- Address and data lines are connected appropriately.
- Higher address bits select the required chip.

#### Increasing the Word Length

Multiple memory chips can be operated in parallel to increase the number of bits per word.

Example:

`Two 1K × 4 chips → One 1K × 8 memory`

Both chips receive the same address, but each chip provides a different part of the word.

---

## 3. SRAM

SRAM stands for **Static Random-Access Memory**.

It stores data using a bistable circuit and does not require periodic refresh while power is present.

### 6T SRAM Cell

A standard SRAM cell contains six transistors:

- Four transistors form two cross-coupled inverters.
- Two access transistors connect the cell to the bit lines.
- The word line controls the access transistors.

The two cross-coupled inverters store one bit as two complementary internal values.

### Hold Operation

During the hold operation:

- The word line is LOW.
- The access transistors are OFF.
- The cell is disconnected from the bit lines.
- The cross-coupled inverters retain the stored value.

### Read Operation

1. `BL` and `BL̅` are precharged.
2. The word line is activated.
3. The stored value causes one bit line to discharge slightly.
4. The other bit line remains near its precharged value.
5. The sense amplifier detects the difference.
6. The stored data appears at the output.

The read operation should not change the stored value.

### Write Operation

1. The write driver places complementary data on `BL` and `BL̅`.
2. The word line is activated.
3. The bit-line voltages force the internal nodes to the required values.
4. The word line is deactivated.
5. The cross-coupled inverters retain the new value.

### Precharge and Sense Amplification

Before a read:

- Both bit lines are normally precharged to the same voltage.
- The selected cell creates a small voltage difference between them.
- The sense amplifier converts this difference into a full logic value.

### Read Stability

Read stability is the ability of the SRAM cell to retain its value during a read operation.

The pull-down transistor is generally made stronger than the access transistor so that reading does not accidentally flip the cell.

### Write Ability

Write ability is the ease with which the stored value can be changed.

The access transistor must be sufficiently strong compared with the pull-up transistor to overwrite the previous value.

### SRAM Timing

Important SRAM timing parameters include:

- Address access time
- Read cycle time
- Write cycle time
- Write pulse width
- Data setup time
- Data hold time

### SRAM Characteristics

#### Advantages

- Very fast
- No refresh required
- Read operation is non-destructive
- Simple interface

#### Limitations

- Requires more transistors per bit
- Large cell area
- Lower storage density
- Higher cost per bit

#### Applications

- Processor caches
- Register files
- Small on-chip memories
- Buffers

---

## 4. DRAM

DRAM stands for **Dynamic Random-Access Memory**.

It stores data as electrical charge in a capacitor.

### 1T–1C DRAM Cell

A DRAM cell contains:

- One access transistor
- One storage capacitor

The word line controls the access transistor, while the bit line is used for reading and writing.

### Write Operation

1. The required data is driven onto the bit line.
2. The word line is activated.
3. The access transistor turns ON.
4. The storage capacitor is charged or discharged.
5. The word line is deactivated.

The charged and discharged states represent logic `1` and logic `0`.

### Read Operation

1. The bit line is precharged.
2. The word line is activated.
3. Charge sharing occurs between the capacitor and bit line.
4. A small voltage change appears on the bit line.
5. The sense amplifier detects and amplifies this change.
6. The stored value is restored to the capacitor.

### Destructive Read and Restoration

Reading a DRAM cell disturbs the charge stored in its capacitor.

Therefore:

- The read operation is destructive.
- The detected value must be written back into the cell.
- This process is called restoration.

### Leakage and Refresh

The capacitor gradually loses charge because of leakage current.

Therefore, every DRAM row must be periodically read and restored. This process is called **refresh**.

Refresh:

- Preserves stored data
- Consumes power
- Temporarily occupies the memory
- Is managed by the memory controller

### DRAM Array Organization

DRAM cells are arranged into rows and columns.

- The row decoder activates one word line.
- The selected row transfers data to the sense amplifiers.
- The column decoder selects the required part of that row.

### Row Buffer

The sense amplifiers that hold the active row collectively form the **row buffer**.

- Accessing another column in the open row is relatively fast.
- Accessing a different row requires closing the current row and opening the new row.

### RAS and CAS

DRAM can use the same external address pins for row and column addresses.

- **RAS:** Row Address Strobe; captures the row address.
- **CAS:** Column Address Strobe; captures the column address.

This technique reduces the required number of address pins.

### SRAM vs DRAM

| Feature | SRAM | DRAM |
|---|---|---|
| Storage element | Bistable latch | Capacitor |
| Typical cell | 6 transistors | 1 transistor and 1 capacitor |
| Refresh | Not required | Required |
| Read operation | Non-destructive | Destructive |
| Speed | Faster | Slower |
| Density | Lower | Higher |
| Cost per bit | Higher | Lower |
| Main application | Cache memory | Main memory |

---

## 5. ROM

ROM stands for **Read-Only Memory**.

It is non-volatile and retains its contents even when power is removed.

During normal system operation, ROM is mainly read. Depending on its type, it may be programmed or erased using a special procedure.

### ROM Organization

A ROM contains:

- Address decoder
- Memory-cell array
- Output or sensing circuitry

The stored connection pattern determines the output data for each address.

### Mask ROM

- Programmed during chip fabrication
- Cannot be modified after manufacturing
- Low cost for large production volumes
- Used when the stored data is permanently fixed

### PROM

PROM stands for **Programmable Read-Only Memory**.

- Programmed by the user
- Can be programmed only once
- Uses fuses or antifuses
- Cannot normally be erased

### EPROM

EPROM stands for **Erasable Programmable Read-Only Memory**.

- Electrically programmed
- Erased using ultraviolet light
- The complete chip is usually erased together
- Can be reprogrammed after erasure

### EEPROM

EEPROM stands for **Electrically Erasable Programmable Read-Only Memory**.

- Electrically programmed
- Electrically erased
- Can often erase individual bytes
- Writing is slower than reading
- Supports a limited number of write cycles

### Flash Memory

Flash memory is a type of EEPROM that erases data in larger blocks instead of individual bytes.

Advantages include:

- High storage density
- Faster block erase
- Lower cost per bit
- Non-volatile storage

Applications include:

- Mobile internal storage
- Solid-state drives
- Memory cards
- USB drives
- Firmware storage

### NOR Flash vs NAND Flash

| NOR Flash | NAND Flash |
|---|---|
| Fast random reads | Fast block reads and writes |
| Supports direct code execution | Code is generally copied to RAM before execution |
| Lower density | Higher density |
| Higher cost per bit | Lower cost per bit |
| Used for firmware storage | Used for mass storage |

---

## 6. Modern DRAM Concepts

### SDRAM

SDRAM stands for **Synchronous Dynamic Random-Access Memory**.

Its commands and data transfers are synchronized with a clock.

This allows:

- Predictable timing
- Pipelined operations
- Burst data transfers
- Better coordination with the processor

### DDR SDRAM

DDR stands for **Double Data Rate**.

DDR memory transfers data on both:

- Rising edge of the clock
- Falling edge of the clock

Therefore, a `1 GHz` memory clock can provide approximately `2 billion transfers per second`.

### Banks, Rows and Columns

DRAM is divided into multiple banks.

Each bank contains:

- Rows
- Columns
- A row buffer

Multiple banks allow the memory controller to overlap operations and improve bandwidth.

A typical access involves:

1. **Activate:** Open the required row.
2. **Read/Write:** Select the required columns.
3. **Precharge:** Close the row before opening another row in that bank.

### Burst Operation

A burst transfers multiple consecutive data units after a single read or write command.

Benefits include:

- Reduced command overhead
- Higher bandwidth
- Efficient transfer of cache lines

### DDR Generations

Newer DDR generations generally provide:

- Higher data-transfer rates
- Greater bandwidth
- Lower operating voltage
- Improved power management
- Greater internal parallelism

The external transfer rate can increase without operating the internal memory-cell array at the same high frequency.

### Prefetch Architecture

DDR memory fetches multiple bits internally and transfers them sequentially through faster external data pins.

Examples:

- DDR: `2n` prefetch
- DDR2: `4n` prefetch
- DDR3: `8n` prefetch
- DDR4: `8n` prefetch

Prefetch allows a high external data rate while keeping the internal memory array relatively slower.

### Latency and Bandwidth

- **Latency:** Time required to complete a particular memory access.
- **Bandwidth:** Amount of data transferred per unit time.

A memory can have high bandwidth while still having relatively high latency.

### Memory Controller

The memory controller connects the processor or SoC to DRAM.

Its responsibilities include:

- Generating memory commands
- Address mapping
- Scheduling reads and writes
- Controlling refresh
- Managing DRAM banks
- Enforcing timing constraints
- Coordinating data transfers

---

## 7. Memory Timing and Power

### Read Timing

During a read operation:

1. Address and control signals are applied.
2. The required row and column are selected.
3. The memory cell affects the bit line.
4. The sense amplifier detects the stored value.
5. Valid data appears at the output.

The delay between applying the address and obtaining valid data is called the **read access time**.

### Write Timing

During a write operation:

1. The address is applied.
2. Input data is placed on the data lines.
3. Write enable is activated.
4. The selected cell stores the input data.
5. Address and data must remain stable for the required interval.

### Setup and Hold Time

- **Setup time:** Minimum time for which an input must remain stable before the active clock or control edge.
- **Hold time:** Minimum time for which an input must remain stable after the active clock or control edge.

Violating setup or hold requirements can result in an incorrect or unreliable operation.

### Dynamic Power

Dynamic power is consumed when circuit nodes switch between logic states.

`Pdynamic = αCV²f`

Where:

- `α` = Switching activity
- `C` = Switched capacitance
- `V` = Supply voltage
- `f` = Operating frequency

Major sources of dynamic power include:

- Charging and discharging bit lines
- Word-line switching
- Address decoders
- Sense amplifiers
- Input/output circuitry

### Leakage Power

Leakage power is consumed even when the memory is not switching.

`Pleakage = Ileakage × V`

Leakage becomes significant in large memory arrays because millions of memory cells contribute leakage current.

### Active and Standby Power

- **Active power:** Power consumed during read, write, refresh and switching operations.
- **Standby power:** Power consumed while the memory is powered but not actively accessed.

DRAM consumes refresh power even when normal read and write operations are not taking place.

### Power-Reduction Techniques

- Reduce supply voltage.
- Reduce unnecessary switching.
- Activate only the required memory bank.
- Apply clock gating to peripheral circuits.
- Power-gate unused memory blocks.
- Use low-leakage transistors.
- Reduce unnecessary DRAM row activations.
- Use low-power and self-refresh modes.
- Keep frequently accessed data in the same open DRAM row.
