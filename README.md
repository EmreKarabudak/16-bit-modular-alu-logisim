# 16-Bit Modular ALU (Logisim)

> 🇹🇷 [Türkçe Dokümantasyon için tıklayın (README.tr.md)](README.tr.md)

This repository contains the modular design and incremental evolution of a **16-bit Arithmetic Logic Unit (ALU)** built in **Logisim**. The project demonstrates a structured hardware design workflow, progressing from a core Arithmetic & Comparison unit (`ALU_v1`) to a full 8-operation ALU (`ALU_v2`) integrating a Logic Unit (`LU`) and MSB-based operation grouping.

---

## Project Versions & Evolution

### 🔹 `ALU_v1` — Arithmetic & Comparison Core
* Preserved as a verified baseline for arithmetic and comparison operations before expanding the datapath.
* Supports **4 operations** (`ADD`, `SUB`, `SLT`, `SLTU`) using the **Arithmetic Unit (`AU`)** subcircuit.
* Uses `OpCode bit[0]` (LSB) to control two's complement subtraction (`Sub` and `Arit_Mux`).

### 🔹 `ALU_v2` — Full 8-Operation ALU (Current)
* Integrates the **Logic Unit (`LU`)** (`AND`, `OR`, `XOR`, `NOR`) alongside the existing `AU` subcircuit.
* Expands the output selection to an **8-to-1 Multiplexer (`MUX`)** driven by a 3-bit `OpCode`.
* **Control Refactoring:** Moves subtraction/comparison control decoding from `bit[0]` to **`bit[2]` (`1xx` group)** to cleanly separate arithmetic/comparison operations from logic operations.

---

## Architecture Overview (`ALU_v2`)

* **16-Bit Datapath (`Input 1`, `Input 2`):** Dual 16-bit inputs routed cleanly across subcircuits via Logisim tunnels.
* **Arithmetic Unit (`AU`):**
  * **Addition (`ADD`):** Performs standard 16-bit binary addition (`A + B`).
  * **Two's Complement Subtraction (`SUB`):** When `bit[2] = 1`, `Arit_mux` selects the inverted `Input 2` (`~B`) via a `NOT` gate, while `Sub` drives `C in = 1` to compute `A + (~B) + 1`.
  * **SLTU (Set Less Than Unsigned):** Evaluates unsigned `A < B` via the inverted carry-out (`~Cout`) of the MSB stage during subtraction.
  * **SLT (Set Less Than Signed):** Evaluates signed `A < B` using `MSB XOR Overflow` of the subtraction result.
  * **Bit Extension:** 1-bit `Slt` and `Sltu` flags are zero-extended to 16 bits (`0..16 extend`) before entering the main `MUX`.
* **Logic Unit (`LU`):**
  * Computes 16-bit bitwise `AND`, `OR`, `XOR`, and `NOR` in parallel directly from `Input 1` and `Input 2`.

---

## OpCode Table (`ALU_v2`)

| OpCode | Group | Operation | Description |
| :--- | :--- | :--- | :--- |
| `000` | `0xx` | `ADD` | 16-bit addition (`A + B`) |
| `001` | `0xx` | `AND` | Bitwise AND (`A & B`) |
| `010` | `0xx` | `OR` | Bitwise OR (`A \| B`) |
| `011` | `0xx` | `XOR` | Bitwise XOR (`A ^ B`) |
| `100` | `1xx` | `SUB` | 16-bit two's complement subtraction (`A - B`) |
| `101` | `1xx` | `SLTU` | Set Less Than Unsigned (`A < B` unsigned) |
| `110` | `1xx` | `SLT` | Set Less Than Signed (`A < B` signed) |
| `111` | `1xx` | `NOR` | Bitwise NOR (`~(A \| B)`) |

*(Note: When `OpCode = 111` (`NOR`), `bit[2] = 1` puts `AU` into subtraction mode in the background, while the main `MUX` selects the `LU` `NOR` output. Thus, the `Result` output is unaffected, though the `Overflow` flag reflects the background `AU` state.)*

---

## Test & Verification Scenarios

The `ALU_v1` and `ALU_v2` circuits were verified across arithmetic, comparison, overflow, and bitwise logic test cases:

| Scenario | Inputs (`A`, `B`) | OpCode | Expected / Observed Result |
| :--- | :--- | :--- | :--- |
| **Positive Addition** | `12`, `25` | `000` (`ADD`) | `37` (`0x0025`), `Overflow = 0` |
| **Signed Addition** | `-10`, `4` / `-10`, `-5` | `000` (`ADD`) | `-6` (`0xFFFA`) / `-15` (`0xFFF1`), `Overflow = 0` |
| **Signed Overflow Boundary** | `32767` (`0x7FFF`), `1` (`0x0001`) | `000` (`ADD`) | `-32768` (`0x8000`), `Overflow = 1` |
| **Zero Subtraction Edge Case** | `0`, `0` | `100` (`SUB`) | `0` (`0x0000`), `Overflow = 0` |
| **Small Minus Large Subtraction** | `10`, `25` | `100` (`SUB`) | `-15` (`0xFFF1`) |
| **Signed Subtraction** | `49`, `-9` | `100` (`SUB`) | `58` (`0x003A`) |
| **Equal Values Comparison** | `20`, `20` | `101` (`SLTU`) / `110` (`SLT`) | `0` (`0x0000`) for both `SLTU` and `SLT` |
| **Signed vs. Unsigned (`A > B` signed)** | `49` (`0x0031`), `-9` (`0xFFF7` / `65527`) | `110` (`SLT`) / `101` (`SLTU`) | `SLT = 0` (`49 < -9` false), `SLTU = 1` (`49 < 65527` true) |
| **Bitwise Pattern (`LU`)** | `0xFF00`, `0x0F0F` | `001` (`AND`), `010` (`OR`), `011` (`XOR`), `111` (`NOR`) | `AND = 0x0F00`, `OR = 0xFF0F`, `XOR = 0xF00F`, `NOR = 0x00F0` |
| **Extreme Bit Patterns (`LU`)** | `0x0000` / `0xFFFF` | `001`–`111` | Verified bit-level outputs including `NOR` inversion (`0xFFFF` for `0x0000`, `0x0000` for `0xFFFF`) |

---

## How to Run

1. Open **Logisim** (Classic / Evolution).
2. Load the `.circ` project file from this repository.
3. Switch between `ALU_v1` and `ALU_v2` circuits in the left explorer pane.
4. Toggle `Input 1`, `Input 2`, and the 3-bit `OpCode` pin to observe `Result` and `Overflow`.

---

## Roadmap (`ALU_v3` — Planned)

* Expand `OpCode` from 3 bits to **4 bits** (16 operations) with a 16-to-1 output `MUX`.
* Integrate a **Barrel Shifter** unit supporting `SLL`, `SRL`, and `SRA`.
* Add **`MAX` / `MIN`** selection units reusing the existing `SLT` / `SLTU` comparison logic.