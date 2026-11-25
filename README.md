# 16-Bit Modular ALU (Logisim)

> 🇹🇷 [Türkçe Dokümantasyon için tıklayın (README.tr.md)](README.tr.md)

This repository contains the modular design and incremental evolution of a **16-bit Arithmetic Logic Unit (ALU)** built in **Logisim**. The project demonstrates a structured hardware design workflow, progressing from a core Arithmetic & Comparison unit (`ALU_v1`), to an 8-operation ALU (`ALU_v2`) with a Logic Unit (`LU`), to a modular 11-operation ALU (`ALU_v3`) adding a dedicated `Shifter` subcircuit (`SLL`, `SRL`, `SRA`), 4-bit `OpCode`, and a 16-input `MUX`.

---

## Project Versions & Evolution

### 🔹 `ALU_v1` — Arithmetic & Comparison Core
* Preserved as a verified baseline for arithmetic and comparison operations before expanding the datapath.
* Supports **4 operations** (`ADD`, `SUB`, `SLT`, `SLTU`) using the **Arithmetic Unit (`AU`)** subcircuit.
* Uses `OpCode bit[0]` (LSB) to control two's complement subtraction (`Sub` and `Arit_Mux`).

### 🔹 `ALU_v2` — Full 8-Operation ALU
* Integrates the **Logic Unit (`LU`)** (`AND`, `OR`, `XOR`, `NOR`) alongside the existing `AU` subcircuit.
* Expands the output selection to an **8-to-1 Multiplexer (`MUX`)** driven by a 3-bit `OpCode`.
* **Control Refactoring:** Moves subtraction/comparison control decoding from `bit[0]` to **`bit[2]` (`1xx` group)** to cleanly separate arithmetic/comparison operations from logic operations.

### 🔹 `ALU_v3` — Modular Shifter & 16-Input MUX ALU (Current)
* Adds a dedicated **`Shifter`** subcircuit supporting **1-bit `SLL`**, **1-bit `SRL`**, and **1-bit `SRA`** operations from `Input 1`.
* Expands `OpCode` to **4 bits** and routes output selection through a **16-input `MUX`**.
* Drives `Sub` and `Arit_mux` control lines via `OpCode bit[2]` using a 4-bit `Splitter`.

---

## Architecture Overview (`ALU_v3`)

* **16-Bit Datapath (`Input 1`, `Input 2`):** Dual 16-bit inputs routed cleanly across subcircuits via Logisim tunnels.
* **Arithmetic Unit (`AU`):**
  * **Addition (`ADD`):** Performs standard 16-bit binary addition (`A + B`).
  * **Two's Complement Subtraction (`SUB`):** When `bit[2] = 1`, `Arit_mux` selects the inverted `Input 2` (`~B`) via a `NOT` gate, while `Sub` drives `C in = 1` to compute `A + (~B) + 1`.
  * **SLTU (Set Less Than Unsigned):** Evaluates unsigned `A < B` via the inverted carry-out (`~Cout`) of the MSB stage during subtraction.
  * **SLT (Set Less Than Signed):** Evaluates signed `A < B` using `MSB XOR Overflow` of the subtraction result.
  * **Bit Extension:** 1-bit `Slt` and `Sltu` flags are zero-extended to 16 bits (`0..16 extend`) before entering the main `MUX`.
* **Logic Unit (`LU`):**
  * Computes 16-bit bitwise `AND`, `OR`, `XOR`, and `NOR` in parallel directly from `Input 1` and `Input 2`.
* **Shifter (`Shifter` Subcircuit):**
  * Uses Logisim `Splitter` components on `Input 1` to produce three parallel 16-bit shift outputs:
    * **`SLL`:** Shifts bits left by 1 (`0..14` to `1..15`) and inserts `0` at bit `0`.
    * **`SRL`:** Shifts bits right logically by 1 (`1..15` to `0..14`) and inserts `0` at bit `15`.
    * **`SRA`:** Shifts bits right arithmetically by 1 (`1..15` to `0..14`) and preserves the sign bit (`Input 1[15]`) at bit `15`.

---

## OpCode Table (`ALU_v3`)

| OpCode | Group | Operation | Description |
| :--- | :--- | :--- | :--- |
| `0000` | `0xxx` | `ADD` | 16-bit addition (`A + B`) |
| `0001` | `0xxx` | `AND` | Bitwise AND (`A & B`) |
| `0010` | `0xxx` | `OR` | Bitwise OR (`A \| B`) |
| `0011` | `0xxx` | `XOR` | Bitwise XOR (`A ^ B`) |
| `0100` | `0xxx` | `SUB` | 16-bit two's complement subtraction (`A - B`) |
| `0101` | `0xxx` | `SLTU` | Set Less Than Unsigned (`A < B` unsigned) |
| `0110` | `0xxx` | `SLT` | Set Less Than Signed (`A < B` signed) |
| `0111` | `0xxx` | `NOR` | Bitwise NOR (`~(A \| B)`) |
| `1000` | `1xxx` | `SLL` | 1-bit Shift Left Logical on `Input 1` |
| `1001` | `1xxx` | `SRL` | 1-bit Shift Right Logical on `Input 1` |
| `1010` | `1xxx` | `SRA` | 1-bit Shift Right Arithmetic on `Input 1` |

---

## Test & Verification Scenarios

The `ALU_v1`, `ALU_v2`, and `ALU_v3` circuits were verified across arithmetic, comparison, overflow, bitwise logic, and shift test cases:

| Scenario | Inputs (`A`, `B`) | OpCode | Expected / Observed Result |
| :--- | :--- | :--- | :--- |
| **Positive Addition** | `12`, `25` | `000` (`ADD`) / `0000` | `37` (`0x0025`), `Overflow = 0` |
| **Signed Addition** | `-10`, `4` / `-10`, `-5` | `000` (`ADD`) / `0000` | `-6` (`0xFFFA`) / `-15` (`0xFFF1`), `Overflow = 0` |
| **Signed Overflow Boundary** | `32767` (`0x7FFF`), `1` (`0x0001`) | `000` (`ADD`) / `0000` | `-32768` (`0x8000`), `Overflow = 1` |
| **Zero Subtraction Edge Case** | `0`, `0` | `100` (`SUB`) / `0100` | `0` (`0x0000`), `Overflow = 0` |
| **Small Minus Large Subtraction** | `10`, `25` | `100` (`SUB`) / `0100` | `-15` (`0xFFF1`) |
| **Signed Subtraction** | `49`, `-9` | `100` (`SUB`) / `0100` | `58` (`0x003A`) |
| **Equal Values Comparison** | `20`, `20` | `101` (`SLTU`) / `110` (`SLT`) | `0` (`0x0000`) for both `SLTU` and `SLT` |
| **Signed vs. Unsigned (`A > B` signed)** | `49` (`0x0031`), `-9` (`0xFFF7` / `65527`) | `110` (`SLT`) / `101` (`SLTU`) | `SLT = 0` (`49 < -9` false), `SLTU = 1` (`49 < 65527` true) |
| **Bitwise Pattern (`LU`)** | `0xFF00`, `0x0F0F` | `001` (`AND`), `010` (`OR`), `011` (`XOR`), `111` (`NOR`) | `AND = 0x0F00`, `OR = 0xFF0F`, `XOR = 0xF00F`, `NOR = 0x00F0` |
| **Extreme Bit Patterns (`LU`)** | `0x0000` / `0xFFFF` | `001`–`111` | Verified bit-level outputs including `NOR` inversion (`0xFFFF` for `0x0000`, `0x0000` for `0xFFFF`) |
| **Shift Negative MSB (`ALU_v3`)** | `0x8006` (`1000 0000 0000 0110`) | `1000` (`SLL`), `1001` (`SRL`), `1010` (`SRA`) | `SLL = 0x000C`, `SRL = 0x4003`, `SRA = 0xC003` (sign bit `1` preserved) |
| **Arithmetic Right Shift Positive (`ALU_v3`)** | `0x0C66` (`0000 1100 0110 0110`) | `1010` (`SRA`) | `0x0633` (`0000 0110 0011 0011`, sign bit `0` preserved) |

---

## How to Run

1. Open **Logisim** (Classic / Evolution).
2. Load the `.circ` project file from this repository.
3. Switch between `ALU_v1`, `ALU_v2`, and `ALU_v3` circuits in the left explorer pane.
4. Toggle `Input 1`, `Input 2`, and the `OpCode` pin to observe `Result` and `Overflow`.

---

## Roadmap (`ALU_v4` — Planned)

* Add variable shift amount (`shamt`) control (4-bit `0–15` shift) using multi-stage shifting.
* Add **`MAX` / `MIN`** selection units reusing the existing `SLT` / `SLTU` comparison logic.