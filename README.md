# SHAKTI Microprocessor - COA Assignment Overview

## Table of Contents

1. [Introduction](#introduction)
2. [Types or Versions](#types-or-versions)
3. [Architecture Overview](#architecture-overview)
4. [Instruction Set Architecture (ISA)](#instruction-set-architecture-isa)
5. [Endianness & Number Formats](#endianness--number-formats)
6. [Pipelining](#pipelining)
7. [Interrupt Handling](#interrupt-handling)
8. [Registers](#registers)
9. [Cache and Memory Hierarchy](#cache-and-memory-hierarchy)
10. [Matrix Multiplication Support](#matrix-multiplication-support)
11. [OS Support](#os-support)
12. [Toolchain & Codebase](#toolchain--codebase)
13. [Benchmarks & Performance](#benchmarks--performance)
14. [Research Component](#research-component)
15. [Conclusion](#conclusion)

---

## Introduction

**Processor:** SHAKTI
**Developer:** Reconfigurable Intelligent Systems Engineering (RISE) group at IIT Madras
**Vision:** To build an open-source, production-grade processor ecosystem in India. The initiative focuses on SoCs that are competitive in performance, power, and area.
**License:** Modified BSD License (Open Source)

---

## Types or Versions

SHAKTI has various processor classes for different applications:

### SHAKTI Processor Classes:

* **E-Class:** Small, low-power devices like smart cards, sensors, IoT. Speed <200 MHz, supports RTOS like FreeRTOS.
* **C-Class:** Industrial machines, networking equipment. Moderate speed (\~100–200 MHz), Linux support, has MMU.
* **I-Class:** Smartphones, routers. High speed (1.5–2.5 GHz), out-of-order, multi-threaded, deep pipelines.
* **M-Class:** General-purpose computing. Multi-core (up to 8), 2.5 GHz.
* **S-Class:** Enterprise systems. 2–16 cores, 1.2–3 GHz.
* **H-Class:** HPC/supercomputers. Up to 128 cores.
* **T-Class:** Security-focused applications. Built-in hardware protections.
* **F-Class:** Fault-tolerant class for space/defense with backup/error correction.

### Other RISC-V Implementations:

* **SiFive:** E, S, U series
* **AndesCore:** N22, AX45MP
* **Alibaba Xuantie:** Xuantie-910, E902/E906
* **Western Digital SweRV:** EH1, EL2
* **Academic Models:** BOOM, PULP

---

## Architecture Overview

### Components and Interactions:

* **Instruction Fetch:** From L1 instruction cache using PC.
* **Decode/Register Fetch:** Decodes instruction, fetches registers.
* **Execute (ALU):** Integer, branch, optional Mult/Div.
* **Memory Access:** Accesses D-cache, loads/stores via AXI.
* **Write Back:** Writes result to register file.
* **Control Unit:** Handles signals, stalling, flow.
* **Pipeline:** 5-stage (IF, ID, EX, MEM, WB).
* **CSRs:** Control & status management.
* **Interrupt Controller:** Handles timer/external/software interrupts.
* **Debug Module:** JTAG, trace.
* **Memory Interfaces:** AXI/AHB to DRAM/peripherals.

### Instruction Flow:

* IF -> ID -> EX -> MEM -> WB.
* Pipeline manages parallel execution.
* CSRs & control unit ensure synchronization & hazard handling.

---

## Instruction Set Architecture (ISA)

* **Type:** RISC-V (RISC)
* **Instruction Format:**

  * **R-type:** `ADD rd, rs1, rs2`
  * **I-type:** `LW rd, offset(rs1)`
  * **J-type:** `JAL rd, offset`
* **Instruction Types:** Arithmetic, Logical, Control
* **Operands:** 3 per instruction (2 source + 1 dest)

---

## Endianness & Number Formats

* **Endianness:** Little-endian
* **IEEE 754 Floating Point:** Supported in I-Class
* **Data Types:**

  * Signed/Unsigned integers
  * Floating-point (IEEE 754)
  * Fixed-point (software)

---

## Pipelining

* **Stages:** IF → ID → EX → MEM → WB

### Hazards:

1. **Data Hazards**

   * Solved by forwarding and stalling
2. **Control Hazards**

   * Solved with prediction/static delays
3. **Structural Hazards**

   * Avoided using Harvard architecture

### Execution:

* E/C-Class: In-order, scalar
* H-Class: Superscalar, out-of-order (issue queues, ROBs)

---

## Interrupt Handling

* **Types:** Hardware (timers, I/O), Software (ecall)
* **Mechanism:**

  * Registers: `mstatus`, `mie`, `mip`, `mcause`, `mtvec`
  * Trap vector, handler, mret return
* **Priority:** External > Timer > Software; Machine > Supervisor > User
* **Software:** Custom ISRs via SHAKTI SDK (C or ASM)

---

## Registers

* **General-Purpose:** x0–x31 (32 total)

  * x0 is hardwired to 0
* **Special-Purpose:** CSRs for MMU, interrupts, privilege
* **Size:**

  * E/C-Class: 32 or 64-bit
  * I/M/S/H-Class: 64-bit

---

## Cache and Memory Hierarchy

### Cache Details:

| SHAKTI Core | Cache Present  | Type             | Size                | Policy        |
| ----------- | -------------- | ---------------- | ------------------- | ------------- |
| E-Class     | No             | -                | -                   | -             |
| C-Class     | Yes (L1 I\&D)  | Harvard          | 16–32 KB each       | LRU / Pseudo  |
| I-Class     | Yes (L1 & L2)  | Unified/separate | 64 KB L1, 256 KB L2 | Pseudo/custom |
| H-Class     | Yes (L1/L2/L3) | Deep hierarchy   | High capacity       | Advanced      |

### Cache Type:

* Mostly write-back
* 2-way or 4-way associativity

### Memory Hierarchy:

```
+-------------------------+
|   L1 Instruction Cache  | (16–32 KB)
+-------------------------+
|      L1 Data Cache      | (16–32 KB)
+-------------------------+
|       L2 Cache          | (128–256 KB)
+-------------------------+
|   Main Memory (DRAM)    | (via AXI)
+-------------------------+
|   Storage / IO Devices  |
+-------------------------+
```

---

## Matrix Multiplication Support

* **Native Instruction:** No SIMD/matrix instructions
* **Software:** Implemented via load/store/add/mul
* **Optimizations:**

  * Hand-optimized ASM
  * Libraries like Eigen, CMSIS-DSP
  * FPGA accelerators supported via modularity

---

## OS Support

* **Linux:** Supported (C/I/M/S/H-Class with MMU)
* **RTOS:** FreeRTOS, Zephyr (E/C-Class)
* **Bare-metal:** All classes

### Compatible Environments:

* Linux Kernel (RISC-V port)
* FreeRTOS / Zephyr
* BusyBox Linux
* GCC, QEMU, Renode

---

## Toolchain & Codebase

* **HDL:** Bluespec SystemVerilog (BSV)
* **Structure:** Modular and hierarchical
* **FPGA Boards:**

  * Arty A7-35T (E-Class)
  * Arty A7-100T (C-Class)
  * Kintex Ultrascale (M/S-Class)

### Tools:

* GCC (RISC-V), Verilator, Bluesim
* Linux, FreeRTOS, Sel4

---

## Benchmarks & Performance

* **Bit-width:** 64-bit (I-Class)
* **Benchmarks:** No public CoreMark; performance comparable in class
* **Target Apps:** General computing, mobile, storage, networking

---

## Research Component

* **GitHub:** [SHAKTI GitHub](https://github.com/shaikt)
* **Paper:** "SHAKTI: An Open-Source Processor Ecosystem Based on RISC-V"
* **Communities:** LinkedIn, Discord, RISC-V Intl.

### External Project: Moushik Processor

* Secure, open-source microprocessor
* IoT-focused with crypto extensions
* Secure boot, modular design

---

## Conclusion

### Key Learnings:

1. Open-source processor design from scratch
2. Application of RISC-V ISA
3. Bluespec SystemVerilog (HDL)
4. Different processor classes and tradeoffs
5. Compilation & simulation workflow
6. MMU, CSRs, interrupts
7. Pipelining and performance optimization
8. Security and fault tolerance in hardware

### Connection to COA Topics:

| Topic      | SHAKTI Implementation                                |
| ---------- | ---------------------------------------------------- |
| ISA        | RISC-V                                               |
| Pipelining | 5-stage + OoO in H-Class                             |
| Memory     | Cache + MMU + DRAM hierarchy                         |
| Registers  | x0–x31 + CSRs                                        |
| Interrupts | Full RISC-V privilege and trap system                |
| Microarch  | IF, ID, EX, MEM, WB pipeline                         |
| I/O        | Real-world integration via memory-mapped peripherals |

### Career Applications:

* VLSI and chip design (BSV/SystemVerilog)
* Embedded systems (RTOS/Linux)
* Computer architecture and compilers
* Security in hardware (T/F-Class)
* Open-source contribution to Indian silicon ecosystem
* Research and higher studies in related fields


