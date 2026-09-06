# 8085 Microprocessor — OA Revision Notes
*(Source: Bharat Acharya's 8085 Notes — content restricted to the first ~35 pages of the uploaded PDF: Introduction to Microprocessors → Architecture → Registers → Flags → Interrupts → Timing/Control pins → Addressing Modes → Instruction Set → Programming patterns → Machine Cycles & Opcode-Fetch Timing Diagram.)*

> **Note on scope:** The PDF actually contains two different internal page-numbering schemes stitched together (a "Page X" scheme for the intro chapters and a separate "Page No: X" scheme for the detailed 8085 chapter). This note follows the **physical order of pages as they appear in the uploaded file**, covering everything from the title page up to the **Opcode Fetch timing diagrams** (the point ~page 35 of the actual PDF). Content after this (Memory Read/Write timing diagrams onward) is **excluded** as instructed.

---

## 1. Basic Organization of a Computer

A computer system = **Processor (µP) + Memory + I/O**, all connected via the **System Bus** (Address Bus + Data Bus + Control Bus).

| Block | Function |
|---|---|
| µP (Microprocessor) | Fetches, Decodes, and Executes instructions |
| Memory | Stores programs and data |
| I/O | Takes user input, displays/prints output |

**OA-relevant fact:** The System Bus itself is a bundle of three buses — **Address Bus, Data Bus, Control Bus**. Don't confuse "System Bus" as a single wire.

---

## 2. The Processor — µP

- The **main function of a µP**: **Fetch → Decode → Execute** (this repeating process = **Instruction Cycle**).
- **Opcode** = the unique binary pattern of an instruction stored in memory; µP decodes the opcode to know what operation to perform.
- Historical note: 8085 had a few thousand transistors; modern CPUs (Core i7) have 1 billion+. Not exam-critical but good context.

**Common trap:** "Instruction Cycle" = Fetch + Execute of ONE instruction (not the whole program).

---

## 3. Memory

- Stores **two kinds of info**: Programs and Data.
- **Primary memory** = RAM + ROM (this is what "memory" means by default in 8085 discussions).
- **Secondary memory** = Hard disk, Floppy, CD/DVD.
- **Cache** = high-speed memory made of SRAM (implemented later in processor evolution, not part of basic 8085 model).
- Memory = a series of **locations**, each with a **unique address**.
- **Every memory location stores exactly 1 Byte (8 bits)** — this is a very important, often-tested fact, tied to why 8085's data bus is 8-bit.

**Common trap:** Do not assume a memory location can hold 16 bits — it is **always 1 byte per address** in 8085.

---

## 4. I/O Devices

- Used to enter data (input) and display/print results (output).
- A touch screen = both input & output device.
- µP, Memory, and I/O are all connected via the **System Bus**.

---

## 5. Architecture of 8085 — Block Diagram

Key functional blocks (from the 8085 architecture diagram):

| Block | Role |
|---|---|
| Accumulator (8-bit) | Holds one operand for ALU ops; stores result of most arithmetic/logic ops |
| Temp Register (8-bit) | Holds the other ALU operand; **not accessible to programmer** |
| Flag Flip-Flops (5 flags) | Store status of last ALU operation |
| ALU (8-bit) | Performs arithmetic & logic operations |
| Instruction Register (8-bit) | Holds fetched opcode |
| Instruction Decoder & Machine Cycle Encoding | Decodes opcode, generates control sequence |
| Register Array (W,Z,B,C,D,E,H,L — 8-bit each) | General purpose + temporary registers |
| Stack Pointer (16-bit) | Points to top of stack |
| Program Counter (16-bit) | Points to next instruction address |
| Incrementer/Decrementer Address Latch (16-bit) | Increments/decrements PC and SP internally |
| Address Buffer (8-bit) | Drives A15–A8 (higher order address bus) |
| Data/Address Buffer (8-bit) | Drives AD7–AD0 (multiplexed lower address + data bus) |
| Timing and Control Unit | Generates control signals (RD, WR, ALE, etc.) using clock inputs X1, X2 |
| Interrupt Control | Manages INTR, RST7.5, RST6.5, RST5.5, TRAP, INTA |
| Serial I/O Control | Manages SID, SOD serial lines |
| Internal Data Bus | 8-bit bus connecting all internal blocks |

**OA-critical diagram facts:**
- 8085's **address bus is 16-bit** (A15–A0), but only **A15–A8 are dedicated address lines**; **AD7–AD0 is a multiplexed Address/Data bus** (demultiplexed externally using ALE + a latch, typically 74LS373).
- 8085's **data bus is 8-bit** (D7–D0), so 8085 is called an "8-bit microprocessor" (word size = 8 bits) but has 16-bit addressing capability → **64 KB (2^16) addressable memory**.
- Internal data bus = 8-bit.

---

## 6. Registers of 8085

| Register | Size | Programmer Access | Purpose |
|---|---|---|---|
| Accumulator (A) | 8-bit | Yes (R/W) | Holds one ALU operand; stores most ALU results |
| B, C, D, E, H, L | 8-bit each | Yes (R/W) | General purpose; can pair up (BC, DE, HL) for 16-bit data |
| Program Counter (PC) | 16-bit | Indirect only (via JMP/CALL etc.) | Holds address of **next** instruction to fetch |
| Stack Pointer (SP) | 16-bit | Indirect only | Holds address of **top of stack** |
| Flag Register (F) | 8-bit (5 flags used) | Indirect (via PUSH/POP PSW) | Stores status of ALU results |
| W, Z (Temp reg pair) | 16-bit | **No** — used internally by µP (e.g., during CALL/JMP) | Temporary storage |
| Temp Register | 8-bit | **No** | Holds one operand during ALU op |
| INR/DCR register | 16-bit | **No** | Internal — increments PC / inc-dec SP for PUSH/POP |

**Register Pairs:**
- **BC pair**, **DE pair**, **HL pair**
- **HL pair** doubles as the **Memory Pointer "M"** — i.e., `M` in instructions like `MOV A, M` means "the memory location addressed by HL".

**Common traps:**
- PC and SP are **always 16-bit**, never 8-bit.
- WZ pair and internal Temp register are **not accessible to the programmer** — a favorite trick question.
- "M" is NOT a register — it's a **memory location pointed to by HL** (indirect addressing).
- PC holds address of the **NEXT** instruction, not the current one, at any given time.
- SP holds the **address of the top of stack**, not the top-of-stack **value** itself.

---

## 7. Flag Register of 8085

8085 uses **5 flags** out of 8 bits (3 bits unused, marked X):

```
D7  D6  D5  D4  D3  D2  D1  D0
S   Z   X   AC  X   P   X   C
```

| Flag | Full name | Set (1) when | Reset (0) when |
|---|---|---|---|
| S | Sign Flag | MSB (D7) of result = 1 (result is –ve, for signed numbers) | MSB = 0 |
| Z | Zero Flag | Result = 0 | Result ≠ 0 |
| AC | Auxiliary Carry | Carry from lower nibble → higher nibble (8-bit ops only) | No such carry |
| P | Parity Flag | Result has **even** parity | Result has **odd** parity |
| C | Carry Flag | Carry/Borrow generated out of MSB (D7) | No carry/borrow |

**Important sub-facts:**
- **AC flag** is used **only** in the **DAA** (Decimal Adjust Accumulator) instruction — for BCD arithmetic. It is **not affected** by 16-bit operations.
- Flag register can ONLY be read/written by the programmer via **PUSH PSW / POP PSW** (PSW = Accumulator + Flag register, A = higher byte, F = lower byte). This is the **only** way to write into the flag register directly.

**Common trap:** Flag register bit positions (S Z X AC X P X C) are a classic MCQ — memorize the exact order and which bits are unused (D5, D3, D1).

---

## 8. Interrupts of 8085

| Interrupt | Priority | Trigger type | Maskable (SIM) | Disabled by DI | Vectored? | Vector Address |
|---|---|---|---|---|---|---|
| TRAP | 1 (highest) | Edge **and** Level | No | No | Yes | 0024H |
| RST 7.5 | 2 | Edge | Yes | Yes | Yes | 003CH |
| RST 6.5 | 3 | Level | Yes | Yes | Yes | 0034H |
| RST 5.5 | 4 | Level | Yes | Yes | Yes | 002CH |
| INTR | 5 (lowest) | Level | No (can't be masked via SIM) | Yes | **No** (non-vectored) | Address from external hardware via INTA |

**Key definitions:**
- **Vectored interrupt** = fixed ISR address (e.g., TRAP → 0024H always).
- **Non-vectored interrupt** = ISR address supplied externally (INTR).
- **INTA** = acknowledgment signal for INTR only; used to fetch opcode/address from external hardware.
- **TRAP** = non-maskable, cannot be disabled even by DI — used for critical events like power failure.
- Interrupt response sequence: µP finishes current instruction → pushes PC (return address) onto stack → resets INTE flip-flop → jumps to ISR address.

**INTE Flip-Flop:**
- Set by **EI** instruction (enables interrupts).
- Reset by: (i) µP reset, (ii) **DI** instruction, (iii) automatically when any interrupt (except TRAP) is recognized (must re-enable via EI inside ISR if nested interrupts needed).
- Affects **all interrupts except TRAP**.

**Masking vs Disabling — a classic trap:**

| Method | Effect | Which interrupts affected |
|---|---|---|
| **DI** instruction | Disables **all** maskable interrupts at once | RST7.5, RST6.5, RST5.5, INTR (not TRAP) |
| **SIM** (masking) | Selectively enable/disable individual interrupts | **ONLY RST 7.5, RST 6.5, RST 5.5** (NOT INTR, NOT TRAP) |

**Common traps:**
- Only 3 interrupts (RST7.5/6.5/5.5) can be individually masked via SIM — INTR **cannot** be masked (only disabled via DI), and TRAP can be **neither** masked nor disabled.
- RST7.5 is **edge-triggered**; RST6.5 and RST5.5 are **level-triggered**; TRAP is **both**.
- Priority order (highest→lowest): **TRAP > RST7.5 > RST6.5 > RST5.5 > INTR**.
- Vector addresses are **8 bytes apart** for RSTn (n×8): RST0=0000H, RST1=0008H, RST2=0010H ... RST7=0038H. But RST7.5/6.5/5.5 (hardware interrupts) have **different** vector addresses (003CH, 0034H, 002CH) — do NOT confuse these with software RST5, RST6, RST7 addresses (0028H, 0030H, 0038H).

---

## 9. Serial I/O, ALU, Instruction Register/Decoder

- **SID** (Serial In Data): µP receives data bit-by-bit through this pin; read via **RIM** instruction.
- **SOD** (Serial Out Data): µP sends data bit-by-bit; set via **SIM** instruction.
- **ALU**: 8-bit; performs arithmetic (ADD, SUB) and logic (AND, OR, XOR, NOT); takes input from Accumulator + Temp register; output usually goes back to Accumulator.
- **Instruction Register**: holds the fetched opcode (from PC address).
- **Instruction Decoder**: decodes the opcode; feeds decoded info to Timing & Control unit for execution.

---

## 10. Timing and Control Circuit — Pins & Signals

| Pin/Signal | Type | Function |
|---|---|---|
| X1, X2 | Input | Clock input (from crystal oscillator) |
| CLK OUT | Output | Clock signal sent to other peripherals for synchronization |
| RESET IN | Input, **active low** | Resets µP; PC ← 0000H (so **Reset Vector Address = 0000H**) |
| RESET OUT | Output | Resets all connected peripherals |
| READY | Input, **active high** | Used to sync µP with slower peripherals; if LOW, µP enters WAIT state |
| ALE (Address Latch Enable) | Output | High when AD7-AD0 carries address (used to latch address externally) |
| IO/M̅ | Output | HIGH = I/O operation, LOW = Memory operation |
| RD̅ | Output, **active low** | Indicates a read operation |
| WR̅ | Output, **active low** | Indicates a write operation |
| S1, S0 | Output | Status lines indicating type of machine cycle |
| HOLD | Input | DMA controller requests bus control |
| HLDA | Output | µP acknowledges HOLD, releases system bus |
| INTA̅ | Output | Acknowledge for INTR |

**S1, S0 Status Table:**

| S1 | S0 | Status |
|---|---|---|
| 0 | 0 | Idle |
| 0 | 1 | Write |
| 1 | 0 | Read |
| 1 | 1 | Opcode Fetch |

**Common traps (very OA-relevant — active high/low confusion):**
- **READY** is **active HIGH**.
- **RD̅ and WR̅** are **active LOW** (bar over the signal name = active low).
- **RESET IN** is **active LOW**.
- **IO/M̅**: HIGH → I/O op; LOW → Memory op (many students reverse this).
- If READY is not needed, it **must be tied to Vcc**, not left floating — otherwise µP goes into infinite WAIT states.

---

## 11. Addressing Modes of 8085

| Mode | Data location | Example | Advantage | Disadvantage |
|---|---|---|---|---|
| **Immediate** | In the instruction itself | `MVI A, 35H` ; `LXI B, 4000H` | Operand easily visible | Multi-byte → slower fetch |
| **Register** | In a register | `MOV B, C` ; `INR B` | Fastest — 1 machine cycle fetch | Operand not visible in code |
| **Direct** | Address given in instruction | `LDA 2000H` ; `STA 2000H` | Address is explicit | 3-byte instruction → 3 fetch cycles |
| **Indirect** | Address is inside a register (pair) | `STAX B` ; `INR M` (via HL) | Reusable in loops; compact | Needs register initialization first |
| **Implied** | Operand implied by opcode itself | `STC`, `CMC` | 1-byte instruction | Operand value not visible |

**Common trap:** `INR M` and `MOV A, M` use **Indirect addressing** (HL points to memory) — NOT "Register addressing," even though M "looks like" a register in mnemonics.

---

## 12. Instruction Set of 8085 (Key Groups)

### Common Notations
| Symbol | Meaning |
|---|---|
| Addr | 16-bit address |
| Data | 8-bit data |
| Data 16 | 16-bit data |
| R, r1, r2 | A register |
| Rp | A register pair (BC→B, DE→D, HL→L) |
| Port | 8-bit I/O address |

### A. Data Transfer Group (no flags affected by ANY of these)

| Instruction | Operation | Addr. Mode | Bytes/Cycles | T-States |
|---|---|---|---|---|
| MOV r1,r2 | r1 ← r2 | Register | 1 cycle | 4 |
| MOV r1,M | r1 ← [HL] | Indirect | 2 cycles | 7 |
| MOV M,r2 | [HL] ← r2 | Register | 2 cycles | 7 |
| MVI r,data | r ← 8-bit data | Immediate | 2 cycles | 7 |
| LXI rp,data16 | rp ← 16-bit data | Immediate | 3 cycles | 10 |
| MVI M,data | [HL] ← 8-bit data | Immediate | 3 cycles | 10 |
| LDA addr | A ← [addr] | Direct | 4 cycles | 13 |
| STA addr | [addr] ← A | Direct | 4 cycles | 13 |
| LHLD addr | L←[addr], H←[addr+1] | Direct | 5 cycles | 16 |
| SHLD addr | [addr]←L, [addr+1]←H | Direct | 5 cycles | 16 |
| LDAX rp | A ← [rp] | Indirect | 2 cycles | 7 |
| STAX rp | [rp] ← A | Indirect | 2 cycles | 7 |
| PCHL | PC ← HL | Register | 1 cycle | 6 |
| SPHL | SP ← HL | Register | 1 cycle | 6 |
| XCHG | HL ↔ DE | Register | 1 cycle | 4 |
| XTHL | HL ↔ [SP],[SP+1] | Register | 5 cycles | 16 |

**Traps:** LDA/STA/LHLD/SHLD are all **3-byte, Direct addressing** instructions (address embedded), but note their T-states differ (13 vs 16) because LHLD/SHLD access **two** memory bytes (L and H separately), while LDA/STA access only one.

### B. Arithmetic Group

| Instruction | Operation | Flags Affected |
|---|---|---|
| ADD R / ADD M / ADI data | A ← A + operand | ALL |
| ADC R / ADC M / ACI data | A ← A + operand + Carry | ALL |
| SUB / SBB / SUI / SBI | Subtraction (with/without borrow) | ALL |
| INR R | R ← R+1 | **All except Carry** |
| DCR R | R ← R−1 | **All except Carry** |
| INX Rp / DCX Rp | 16-bit inc/dec of register pair | **None** |
| DAD Rp | HL ← HL + Rp | **Only Carry** |
| DAA | Adjusts A to valid BCD after addition | ALL |

**Big trap (very frequently tested):**
- **INR/DCR** (8-bit inc/dec) → affects all flags **except Carry**.
- **INX/DCX** (16-bit inc/dec of a register pair) → affects **NO flags at all**.
- **DAD** → affects **only** the Carry flag.

### C. Logic Group

| Instruction | Operation | Flags |
|---|---|---|
| ANA/ANI | A ← A AND operand | ALL |
| ORA/ORI | A ← A OR operand | ALL |
| XRA/XRI | A ← A XOR operand | ALL |
| CMP R / CMP M / CPI | A − operand (result **NOT stored**, only flags set) | ALL |
| STC | Set Carry flag (Cy←1) | Only Carry |
| CMC | Complement Carry flag | Only Carry |
| CMA | Complement Accumulator (1's complement) | None |

**Bit manipulation tricks (important for OA logic questions):**
- To **clear** a bit → AND it with 0 (rest with 1). E.g., `ANI F0H` clears lower nibble.
- To **set** a bit → OR it with 1 (rest with 0). E.g., `ORI 0FH` sets lower nibble.
- To **complement** a bit → XOR it with 1 (rest with 0). E.g., `XRI 0FH` complements lower nibble.

**CMP result interpretation (classic OA question):**

| Comparison result (A vs R, i.e. A−R) | Zero Flag (Z) | Carry Flag (Cy) |
|---|---|---|
| A > R | 0 | 0 |
| A = R | 1 | 0 |
| A < R | 0 | 1 |

**Trap:** CMP computes `A − R`, **not** `R − A`. Direction matters.

### D. Rotate Instructions (all affect only Carry flag)

| Instruction | Action |
|---|---|
| RLC | Rotate A left; MSB → Carry AND → LSB |
| RRC | Rotate A right; LSB → Carry AND → MSB |
| RAL | Rotate A left **through** Carry; MSB→Carry, old Carry→LSB |
| RAR | Rotate A right **through** Carry; LSB→Carry, old Carry→MSB |

**Trap:** RLC/RRC move the bit into carry *and* wrap it to the other end simultaneously. RAL/RAR route **through** the existing Carry flag (the old carry value moves into the vacated bit) — these are different mechanisms; don't mix them up.

### E. Branch Group

**Condition codes:**

| Condition | True if |
|---|---|
| NZ | Z = 0 |
| Z | Z = 1 |
| NC | C = 0 |
| C (Carry) | C = 1 |
| PO (Parity Odd) | P = 0 |
| PE (Parity Even) | P = 1 |
| P (Plus) | S = 0 |
| M (Minus) | S = 1 |

| Instruction | Operation | T-States |
|---|---|---|
| JMP addr | PC ← addr (unconditional) | 10 |
| Jcond addr | PC ← addr, only if condition true | 10 (true) / 7 (false) |
| CALL addr | Push PC, then PC ← addr | 18 |
| Ccond addr | Conditional CALL | 18 (true) / 9 (false) |
| RET | Pop PC (return from subroutine) | 10 |
| Rcond | Conditional RET | 12 (true) / 6 (false) |
| RSTn | Push PC; PC ← (n × 8) | 12 |

**Trap:** T-states **differ** between the "condition true" and "condition false" paths for conditional Jump/Call/Return — because when the condition is false, the µP still fetches the address bytes for Jcond (so it's 7T not less) but skips the stack-push machine cycles entirely for Ccond/Rcond (so those save more T-states, going from 18→9 and 12→6).

**RST n details:**
- RSTn is functionally similar to CALL, but is a **1-byte instruction** (vs CALL's 3 bytes) — address is calculated as **n × 8**, not fetched from following bytes.
- Used for **software interrupts**.
- `PCHL` also causes a branch (PC ← HL) — remember it's part of the Data Transfer Group, but functionally a jump.

### F. Stack, I/O, and Machine Control Group

| Instruction | Operation | T-States |
|---|---|---|
| PUSH Rp | SP←SP−1,[SP]←Rh; SP←SP−1,[SP]←Rl | 12 |
| PUSH PSW | Pushes A (higher byte) and Flags (lower byte) | 12 |
| POP Rp | Pops into register pair (lower byte first) | 10 |
| POP PSW | Pops into A and Flag register | 10 |
| IN port | A ← [port] (I/O) | 10 |
| OUT port | [port] ← A (I/O) | 10 |
| SIM | Set Interrupt Mask / send serial data via SOD | 4 |
| RIM | Read Interrupt Mask / receive serial data via SID | 4 |
| EI | Enable interrupts (INTE ← 1) | 4 |
| DI | Disable interrupts (INTE ← 0) | 4 |
| NOP | No operation (used for delays) | 4 |
| HLT | Halts µP (sets Halt flip-flop) | 5 (4+1T) |

**Key stack facts (high-yield for OA):**
- **PUSH always accesses the HIGHER byte of a register pair first** (e.g., `PUSH B`: B pushed before C). This is the **only operation** that accesses the higher byte before the lower byte.
- **POP always retrieves the LOWER byte first** (matches LIFO — since PUSH put lower byte on top).
- **No stack operation uses 8-bit operands** — you cannot PUSH/POP a single register; always a register pair (or PSW).
- **PSW** = Accumulator (higher byte) + Flag register (lower byte); PSW can **only** be used with PUSH/POP.
- I/O instructions (`IN`, `OUT`) can transfer data **only through the Accumulator**.
- 8085 supports **256 I/O ports** (8-bit port address: 00H–FFH).

---

## 13. Programming Patterns (from worked examples in the notes)

Frequently reused techniques seen in the sample programs — good for recognizing logic in OA MCQs:

- **Carry accumulation while summing an array:** `JNC SKIP` / `INC B` pattern — increment a separate counter register whenever a Carry occurs during an ADD, to track "overflow count" separately from the sum.
- **Finding max/min in an array:** Use `CMP M` in a loop; conditionally update the accumulator (`JNC/JC` based on comparison) while looping through with `DCR C` / `JNZ`.
- **Classifying positive/negative/zero:** Compare each element against 00H using `CMP M`; branch on `JZ` (zero), `JC` (A < M, meaning positive number since A=0), or fall-through (negative).
- **Sorting (Bubble sort pattern):** Nested loops (`DCR B`/`DCR C` + `JNZ`), comparing adjacent elements and swapping via a temp register (E) if out of order.
- **Block transfer variants:**
  - *Simple transfer*: increment both source & destination pointers (`INX B`, `INX D`).
  - *Overlapping transfer*: care taken with overlap; source & destination addresses close together.
  - *Inverted transfer*: source pointer increments while destination pointer **decrements** (`INX B`, `DCX D`) — reverses order.
- **8-bit multiplication:** Implemented as **repeated addition** using `DAD D` in a loop (add multiplicand to itself, multiplier number of times) — 8085 has **no hardware MUL instruction**.
- **8-bit division:** Implemented as **repeated subtraction** — subtract divisor from dividend repeatedly, counting subtractions (quotient) until remainder < divisor.
- **Serial data transmission via SIM:** Uses accumulator bit pattern `01000000` (send 0) / `11000000` (send 1) loaded before each `SIM`, sending LSB-first with start bit (0) and stop bit (1).

**OA-relevant conceptual takeaway:** 8085 has **no direct MUL/DIV instruction** — must be programmed via loops (shift-add / repeated subtraction). This is a classic conceptual MCQ.

---

## 14. Software Delay Formula (Numerical / Conceptual — High-Yield for OA)

**T-State** = one clock cycle = `T = 1 / Clock Frequency`

**General delay-loop formula:**
```
TD = MT + [(Count)d × NT] – 3T
```
Where:
- `NT` = T-states consumed **inside** the loop (per iteration)
- `MT` = T-states consumed **outside** the loop (fixed overhead, e.g., load + final RET/JNZ-false)
- `(Count)d` = the loop count in decimal
- The `–3T` correction accounts for the *last* iteration's `JNZ` being "false" (7T instead of 10T)

**Example (1 register 8-bit loop @ 3 MHz):**
```
MVI B, count   ; 7T (outside loop)
LOOP: DCR B    ; 4T  (inside loop)
      JNZ LOOP ; 10T/7T
RET            ; 10T (outside loop)
```
- Max count (8-bit) = 255 → **Max delay ≈ 1.18 msec** at 3 MHz.

**Quick summary table (all @ 3 MHz):**

| Delay method | Approx. Max Delay |
|---|---|
| Single NOP | 1.332 µsec |
| One 8-bit register loop | 1.18 msec |
| One 16-bit register loop | 0.525 sec |
| Two nested 8-bit registers | 0.304 sec |
| One 8-bit + one 16-bit nested | 133.69 sec |
| Two nested 16-bit registers | ~9 hr 32 min |

**OA trap:** If clock frequency changes, `1T = 1/(new frequency)` — always recompute T before plugging into the formula. If not given, notes assume **3 MHz or 5 MHz**.

---

## 15. Machine Cycles and T-States

| Term | Definition |
|---|---|
| **T-State** | One clock period of the µP: `T = 1/f` |
| **Machine Cycle** | Time for µP to complete ONE operation of accessing memory or I/O (one byte transfer) |
| **Instruction Cycle** | Time to fetch AND execute one complete instruction = Fetch Cycle + Execute Cycle |
| **Fetch Cycle** | Time to fetch all bytes of an instruction |

**Machine Cycle Types & Control Signal States:**

| Machine Cycle | IO/M̅ | RD̅ | WR̅ | S1 | S0 | INTA̅ | T-States |
|---|---|---|---|---|---|---|---|
| Opcode Fetch | 0 | 0 | 1 | 1 | 1 | 1 | 4 (or 6) |
| Memory Read | 0 | 0 | 1 | 1 | 0 | 1 | 3 |
| Memory Write | 0 | 1 | 0 | 0 | 1 | 1 | 3 |
| I/O Read | 1 | 0 | 1 | 1 | 0 | 1 | 3 |
| I/O Write | 1 | 1 | 0 | 0 | 1 | 1 | 3 |
| Interrupt Acknowledge | 1 | 1 | 1 | 1 | 1 | 0 | 3 or 6 |
| Bus Idle | 0 | 1 | 1 | 0 | 0 | 1 | 3 |

**Trap — Machine Cycle vs T-State:** A **Machine Cycle** = one byte access = made up of **multiple T-States** (3, 4, or 6). Do not treat these as equivalent units.

**Trap — Every instruction MUST start with Opcode Fetch** (it's the only **compulsory** machine cycle); Memory Read/Write, I/O Read/Write cycles are optional depending on the instruction.

---

## 16. Opcode Fetch Timing Diagram (4 T-States — the default case)

**During T1:**
- A15–A8 carries the **higher byte of PC (PCH)**.
- Since ALE is HIGH, AD7–AD0 carries the **lower byte of PC (PCL)**.
- Since it's an opcode fetch, **S1 and S0 both go HIGH**.
- Since it's a memory operation, **IO/M̅ goes LOW**.

**During T2:**
- ALE goes LOW → address removed from AD7–AD0.
- RD̅ goes LOW → opcode data appears on AD7–AD0.
- µP checks **READY** pin — if LOW, µP inserts **WAIT states** until READY goes HIGH.

**During T3:**
- Data (opcode) remains on AD7–AD0 as long as RD̅ is LOW.

**During T4:**
- µP **decodes** the opcode (no bus activity).

**6 T-State Opcode Fetch variant:** Some instructions (e.g., those needing extra internal processing like `PCHL`, `SPHL`, `RSTn` push cycles) require **2 extra T-states (T5, T6)** beyond the standard 4, during which the address bus shows **"unspecified"** values — this is a common trick in timing-diagram MCQs (don't assume address bus is always meaningful in every T-state).

**OA-relevant summary points about this diagram:**
- ALE pulse in T1 is used to **latch the lower address byte (PCL)** externally (into a device like a 74LS373), because AD7–AD0 is a **multiplexed** bus.
- PC is auto-incremented (**PC ← PC+1**) during the fetch, preparing for the next byte/instruction.
- Opcode Fetch is **always the first machine cycle** of any instruction and always sets **S1=1, S0=1**.

---

## 17. Quick-Fire Fact Sheet (Common Traps Consolidated)

| Category | Trap / Correct Fact |
|---|---|
| Data bus width | 8-bit (D7–D0) |
| Address bus width | 16-bit (A15–A0), but only A15–A8 dedicated; AD7–AD0 multiplexed |
| Addressable memory | 2^16 = 64 KB |
| I/O ports | 256 (8-bit port address, 00H–FFH) |
| Memory location size | Always 1 byte |
| PC / SP size | Always 16-bit |
| WZ pair / Temp reg | NOT accessible to programmer |
| M (memory pointer) | Not a register — it's [HL], i.e. Indirect addressing |
| PUSH/POP unit | Always a register PAIR (or PSW), never a single 8-bit register |
| PUSH order | Higher byte first |
| POP order | Lower byte first |
| INR/DCR flags | All except Carry |
| INX/DCX flags | None |
| DAD flags | Only Carry |
| CMP direction | A − operand (not operand − A) |
| RD̅, WR̅ | Active LOW |
| READY | Active HIGH |
| RESET IN | Active LOW |
| Reset vector | PC = 0000H |
| IO/M̅ | HIGH = I/O, LOW = Memory |
| Masking (SIM) | Only works on RST7.5, RST6.5, RST5.5 |
| DI instruction | Disables all maskable interrupts (not TRAP) |
| TRAP | Cannot be masked OR disabled |
| INTR | Can be disabled (DI) but NOT masked (SIM); non-vectored |
| Interrupt priority | TRAP > RST7.5 > RST6.5 > RST5.5 > INTR |
| Multiplication/Division | No hardware instruction; done via loops |
| AC flag | Only meaningful for 8-bit ops / DAA; unaffected by 16-bit ops |
| Machine Cycle vs T-State | Machine cycle = group of T-states (one byte transfer); NOT the same unit |
| Opcode fetch | Always first & compulsory machine cycle; sets S1=S0=1 |

---

*End of revision notes (Pages 1–35 scope). For Machine Cycle timing diagrams beyond Opcode Fetch (Memory Read/Write, I/O Read/Write, stack & branch instruction cycle breakdowns), those appear later in the PDF and were intentionally excluded per your scope instruction.*
