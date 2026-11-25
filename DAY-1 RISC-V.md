# RISC-V Workshop – Day 1
### Introduction to the RISC-V ISA, GNU Toolchain and Spike Simulator

![Platform](https://img.shields.io/badge/ISA-RV64I-blue) ![Toolchain](https://img.shields.io/badge/Toolchain-GNU%20GCC-green) ![Simulator](https://img.shields.io/badge/Simulator-Spike-orange)

---

## Table of Contents

1. [Objective](#1-objective)
2. [Introduction](#2-introduction)
3. [Contents](#3-contents)
4. [GNU Compiler Toolchain](#4-gnu-compiler-toolchain)
5. [Lab Experiments](#5-lab-experiments)
6. [Results Summary](#6-results-summary)
7. [Key Learning Outcomes](#7-key-learning-outcomes)

---

## 1. Objective

This report documents the Day 1 laboratory work of the RISC-V workshop. The objectives were to:

- Understand how a high-level program is translated into machine execution, and the role of the Instruction Set Architecture (ISA) in that flow.
- Build a C program with both the native GCC compiler and the RISC-V GNU cross-compiler.
- Inspect the generated RV64I assembly using `objdump` and compare `-O1` with `-Ofast` optimisation.
- Execute and debug the binary on the Spike ISA simulator, observing instruction-level register changes.
- Verify the value ranges of 64-bit signed and unsigned integers experimentally.

**Test environment:** Ubuntu (VirtualBox) with the workshop collateral package (`riscv_workshop_collaterals`) containing the RISC-V GNU toolchain, Spike and the proxy kernel.

---

## 2. Introduction

### 2.1 System Layers

| Layer | Description |
|-------|-------------|
| Application software | User programs written in high-level languages such as C, C++ or Java |
| System software | Operating system, compiler, assembler and system utilities |
| Hardware | The processor, which executes machine-level instructions |

### 2.2 Software-to-Hardware Flow

1. The application is written in a high-level language (C).
2. The **compiler** translates the source into assembly instructions for the target ISA.
3. The **assembler** converts the assembly into binary machine code.
4. The **processor** fetches, decodes and executes the machine instructions.

### 2.3 Role of the ISA

The Instruction Set Architecture is the formal interface between software and hardware. It specifies the supported instructions, the register set, the data types and the operations a processor must implement. Compilers target the ISA, while hardware designers implement it, allowing both to evolve independently.

### 2.4 RTL to Physical Layout

Realising a RISC-V processor in silicon follows the standard VLSI design flow:

1. Define the architecture and specifications.
2. Implement the design in RTL (Verilog / SystemVerilog).
3. Perform logic synthesis to produce a gate-level netlist.
4. Carry out physical design: floorplanning, placement, clock tree synthesis and routing.
5. Generate the final physical layout.

---

## 3. Contents

Topics covered during Day 1:

- Representation of 64-bit unsigned and signed integers
- Application Binary Interface (ABI)
- Memory allocation and the stack pointer
- Pseudo-instructions
- **RV64I** – Base integer instruction set
- **RV64M** – Integer multiplication and division extension
- **RV64F** – Single-precision floating-point extension
- **RV64D** – Double-precision floating-point extension

---

## 4. GNU Compiler Toolchain

| Tool | Purpose |
|------|---------|
| `gcc` | Compiles C programs for the host machine |
| `riscv64-unknown-elf-gcc` | Cross-compiles C programs for RISC-V |
| `riscv64-unknown-elf-objdump` | Disassembles binaries into assembly with addresses |
| `spike` | RISC-V ISA simulator used to execute and debug binaries |
| `pk` (Proxy Kernel) | Provides the minimal runtime environment required by Spike (e.g. `printf` support) |

**Key compiler options**

| Option | Meaning |
|--------|---------|
| `-O1`, `-Ofast` | Optimisation level |
| `-march=rv64i` | Generate only RV64I base-ISA instructions |
| `-mabi=lp64` | 64-bit `long` and pointer types (LP64 ABI) |

---

## 5. Lab Experiments

### 5.1 Experiment 1 – Sum of Integers with Native GCC

**Purpose:** Validate the program logic on the host machine before targeting RISC-V.

**Source file:** `sum1ton.c`

```c
#include<stdio.h>
int main(){
int i,sum=0,n=5;
for(i=0;i<=n;++i){
sum+=i;
}
printf("Sunm of numbers from 1 to %d  is %d\n",n,sum);
return 0;
}
```

**Commands**

```bash
nano sum1ton.c
gcc sum1ton.c
./a.out
```

![Figure 1](images/r1.png)

**Figure 1 – Native compilation and execution.**
The source is created in `nano`, compiled with `gcc` and executed. The output for n = 5 is 15, which matches the expected result (1 + 2 + 3 + 4 + 5).

![Figure 2](images/r2.png)

**Figure 2 – Working directory listing (`ls -ltr`).**
The listing confirms the generated files `sum1ton.c` and `a.out`, alongside the `riscv_workshop_collaterals` directory that contains the toolchain.

---

### 5.2 Experiment 2 – Cross-Compilation for RISC-V (`-O1`)

**Purpose:** Build the same source for a 64-bit RISC-V target using the base integer ISA.

```bash
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c
ls -ltr sum1ton.o
```

![Figure 3](images/r3.png)

**Figure 3 – Source verification and RISC-V compile invocation.**
The source is displayed with `cat` and the cross-compiler is invoked with the RV64I / LP64 options. Note that the optimisation flag is the capital letter `O` (`-O1`).

![Figure 4](images/r4.png)

**Figure 4 – Successful `-O1` build.**
`ls -ltr sum1ton.o` confirms the RISC-V executable was generated. Its size of approximately 167 KB reflects the C library code linked into the binary.

---

### 5.3 Experiment 3 – Assembly Inspection with `objdump` (`-O1`)

**Purpose:** Examine the machine instructions generated for the program.

```bash
riscv64-unknown-elf-objdump -d sum1ton.o | less
```

Within `less`, the command `/main` locates the `main` function.

![Figure 5](images/r5.png)

**Figure 5 – Tail of the disassembly.**
The end of the listing contains compiler support-library routines such as `__moddi3`, `__udivdi3` and `__clzdi2`. These are linked in automatically and are not part of the application source.

![Figure 6](images/r6.png)

**Figure 6 – `main` in the `-O1` binary (address `0x10184`).**
`main` is highlighted within the disassembly, surrounded by start-up and clean-up routines such as `frame_dummy` and `__do_global_dtors_aux`.

![Figure 7](images/r7.png)

**Figure 7 – `main`, `atexit` and `exit` in the `-O1` binary.**
A clearer view of the complete `main` function and the routines that follow it.

**Disassembly of `main` (`-O1`)**

```
10184: addi sp,sp,-16        # allocate stack frame
10188: sd   ra,8(sp)         # save return address
1018c: li   a2,15            # printf argument: sum = 15
10190: li   a1,5             # printf argument: n = 5
10194: lui  a0,0x21          # upper bits of format-string address
10198: addi a0,a0,384        # lower bits of format-string address
1019c: jal  ra,10408 <printf>
101a0: li   a0,0             # return value 0
101a4: ld   ra,8(sp)         # restore return address
101a8: addi sp,sp,16         # release stack frame
101ac: ret
```

**Observation:** The generated code contains no loop. The compiler evaluated the summation at compile time and loads the constants `5` and `15` directly into argument registers `a1` and `a2` before calling `printf`.

**Instructions identified:** `addi`, `sd`, `ld`, `lui`, `jal`, `ret`, and the pseudo-instruction `li`.

---

### 5.4 Experiment 4 – `-O1` versus `-Ofast`

```bash
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c
riscv64-unknown-elf-objdump -d sum1ton.o | less
```

![Figure 8](images/r8.png)

**Figure 8 – `main` in the `-Ofast` binary (address `0x100b0`).**
`main` appears as the first function in the listing, followed by `register_fini` and `_start`.

**Disassembly of `main` (`-Ofast`)**

```
100b0: lui  a0,0x21
100b4: addi sp,sp,-16
100b8: li   a2,15
100bc: li   a1,5
100c0: addi a0,a0,384
100c4: sd   ra,8(sp)
100c8: jal  ra,10408 <printf>
100cc: ld   ra,8(sp)
100d0: li   a0,0
100d4: addi sp,sp,16
100d8: ret
```

**Comparison**

| Parameter | `-O1` | `-Ofast` |
|-----------|-------|----------|
| Address of `main` | `0x10184` | `0x100b0` |
| Instructions in `main` | 11 | 11 |
| Loop present | No (folded to constant 15) | No (folded to constant 15) |
| Instruction ordering | `addi sp` first | `lui a0` first (rescheduled) |

**Observation:** For a program of this size, both optimisation levels produce functionally equivalent code. The differences are the location of `main` within the binary and the scheduling of instructions.

---

### 5.5 Experiment 5 – Execution and Debugging on Spike

**Normal execution**

```bash
spike pk sum1ton.o
```

For this experiment `n` was changed to 100 and the program rebuilt with `-Ofast`. Both native GCC and Spike report **5050**, matching 100 × 101 / 2. The `bbl loader` message is printed by the proxy kernel at start-up.

**Debug mode**

```bash
spike -d pk sum1ton.o
```

| Command | Function |
|---------|----------|
| `until pc 0 100b0` | Run core 0 until the program counter equals `0x100b0` |
| `reg 0 a2` | Display register `a2` of core 0 |
| *Enter* | Execute and display a single instruction |

![Figure 9](images/r9.png)

**Figure 9 – Spike execution and debug session.**
The `-Ofast` binary is built and executed natively and on Spike, both printing 5050. In debug mode, `until pc 0 100b0` halts at the start of `main`, where `a2` reads `0x0000000000000000`. Stepping once executes `lui a2, 0x1`; a subsequent `reg 0 a2` returns `0x0000000000001000`. This is consistent with the `lui` definition, which places its immediate in the upper bits of the register (`0x1 << 12`), and demonstrates the effect of a single instruction on processor state.

---

### 5.6 Experiment 6 – 64-bit Signed and Unsigned Integer Limits

**Theory**

| Type | Range (n bits) | 64-bit range |
|------|----------------|--------------|
| Unsigned | 0 to 2ⁿ − 1 | 0 to 18446744073709551615 |
| Signed (two's complement) | −2ⁿ⁻¹ to 2ⁿ⁻¹ − 1 | −9223372036854775808 to 9223372036854775807 |

- In two's complement, the MSB is the sign bit (0 = non-negative, 1 = negative).
- A negative value is formed by writing the magnitude in binary, inverting all bits (1's complement) and adding 1.
- Unsigned arithmetic exceeding the maximum wraps around modulo 2⁶⁴.

**Commands**

```bash
vim unsignedHighest.c
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o unsignedHighest.o unsignedHighest.c
spike pk unsignedHighest.o
```

![Figure 10](images/r10.png)

**Figure 10 – Unsigned and intermediate signed runs.**
The unsigned program reports the 64-bit maximum, 18446744073709551615. Subsequent runs with modified expressions produced 1023, 0 and −1024, illustrating how strongly the printed result depends on the expression and data type used. The signed program initially printed 32-bit limits (2147483647 and −2147483648), confirming that the type width determines the reported range.

![Figure 11](images/r11.png)

**Figure 11 – Final signed result.**
The program reports a highest value of 9223372036854775807 and a lowest value of −9223372036854775808, which matches the theoretical 64-bit two's-complement range.

---

## 6. Results Summary

| Experiment | Result |
|------------|--------|
| Native GCC, n = 5 | Sum = 15 |
| RISC-V `-O1` build | `main` at `0x10184`; loop folded to constant 15 |
| RISC-V `-Ofast` build | `main` at `0x100b0`; loop folded to constant 15 |
| Spike, n = 100 | Sum = 5050 (matches native GCC) |
| Spike debug | `lui a2,0x1` sets `a2` to `0x1000` |
| 64-bit unsigned maximum | 18446744073709551615 |
| 64-bit signed range | −9223372036854775808 to 9223372036854775807 |

---

## 7. Key Learning Outcomes

- Understood the relationship between application software, system software and hardware, and the ISA's role as the interface between them.
- Reviewed the RTL-to-layout flow used to realise a processor.
- Gained working knowledge of the RISC-V GNU toolchain and the purpose of each tool.
- Compiled and executed the same C program with native GCC and RISC-V GCC with identical results.
- Used `objdump` to inspect RV64I assembly and compared `-O1` and `-Ofast` output.
- Used Spike's debug mode to halt at an address, single-step instructions and inspect registers.
- Verified the ranges of 64-bit signed and unsigned integers experimentally.
