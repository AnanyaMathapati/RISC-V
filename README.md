# RISC-V Workshop | Practical Learning Portfolio

A first-person, evidence-led record of my RISC-V workshop practice in Ubuntu running inside VirtualBox. This repository follows the order of the practical work: write and run a C program, build it for RISC-V, inspect the generated instructions, execute it in Spike, and investigate signed and unsigned integer limits.

> **Scope and evidence:** The terminal captures in `docs/screenshots/` are original workshop-session evidence. Explanations distinguish results visible in those captures from clean reference programs added to make the experiments reproducible. This is an independent learning portfolio, not a copy of another participant's repository.

## Learning sequence

| Stage | Practical focus | Repository evidence |
|---|---|---|
| 01 | Write a C program and run it with the native Ubuntu compiler | `src/sum1ton.c`, screenshots 01–02 |
| 02 | Cross-compile C for the RV64I target using the RISC-V GCC toolchain | build commands, screenshot 03–04 |
| 03 | Inspect generated machine code as RISC-V assembly | `assembly/disassembly.md`, screenshots 05–08 |
| 04 | Execute the target program with Spike and the proxy kernel (`pk`) | `simulation/spike-debugging.md`, screenshot 09 |
| 05 | Inspect execution state and integer registers in Spike's debugger | `simulation/spike-debugging.md`, screenshot 09 |
| 06 | Explore signed and unsigned integer boundaries | `src/signedHighest.c`, `src/unsignedHighest.c`, screenshots 10–11 |

## Repository map

```text
RISC-V/
├── README.md
├── LICENSE
├── .gitignore
├── src/
│   ├── sum1ton.c
│   ├── signedHighest.c
│   └── unsignedHighest.c
├── assembly/
│   └── disassembly.md
├── simulation/
│   └── spike-debugging.md
└── docs/
    ├── WORKSHOP_NOTES.md
    ├── EVIDENCE_INDEX.md
    └── screenshots/
        ├── 01-native-gcc-run.png
        ├── 02-home-directory-files.png
        ├── 03-source-and-cross-compile-attempts.png
        ├── 04-riscv-cross-compile.png
        ├── 05-disassembly-part-1.png
        ├── 06-disassembly-part-2.png
        ├── 07-disassembly-main-and-runtime.png
        ├── 08-disassembly-entry-and-main.png
        ├── 09-spike-debug-session.png
        ├── 10-integer-limits-experiments.png
        └── 11-signed-integer-boundary-tests.png
```

## Environment and tools

- Operating environment: Ubuntu in Oracle VirtualBox
- Native compiler: `gcc`
- RISC-V cross-compiler: `riscv64-unknown-elf-gcc`
- Disassembler: `riscv64-unknown-elf-objdump`
- ISA simulator: Spike (`spike`)
- Proxy kernel: `pk`
- Target options used in the recorded build: `-march=rv64i -mabi=lp64`

Tool versions were not captured in the supplied terminal evidence, so they are intentionally not guessed here.

## Reproduce the sum experiment

Run these commands from the repository root, assuming the RISC-V toolchain and Spike are installed and available in `PATH`.

```bash
# Native Ubuntu build and run
gcc -Wall -Wextra -O0 src/sum1ton.c -o sum1ton-native
./sum1ton-native

# Cross-compile for RV64I with the LP64 ABI
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i \
  -o sum1ton.elf src/sum1ton.c

# Execute the RISC-V program in Spike using pk
spike pk sum1ton.elf

# Disassemble the executable
riscv64-unknown-elf-objdump -d sum1ton.elf | less

# Start the interactive Spike debugger
spike -d pk sum1ton.elf
```

Expected program result for the `n = 100` version:

```text
Sum of numbers from 1 to 100 is 5050
```

The original native test also records the sum from 1 to 5 as `15`. The source in this repository uses `n = 100` and computes the inclusive sum from zero through `n`; adding zero does not change the sum from 1 through `n`.

## Integer-boundary experiments

The two C files in `src/` are clean, portable reference implementations using standard limit macros and matching format specifiers. They are included to make the concepts testable. The screenshots preserve the actual in-session experiments, including intermediate edits and mistakes; the reference files should not be mistaken for byte-for-byte copies of every source revision typed during the lecture.

```bash
gcc -Wall -Wextra -std=c11 src/signedHighest.c -o signedHighest
./signedHighest

gcc -Wall -Wextra -std=c11 src/unsignedHighest.c -o unsignedHighest
./unsignedHighest
```

## How to read this portfolio

1. Start with [Workshop Notes](docs/WORKSHOP_NOTES.md) for the lecture-practical sequence.
2. Use the [Evidence Index](docs/EVIDENCE_INDEX.md) to connect each screenshot to its topic.
3. Read [Assembly and Disassembly](assembly/disassembly.md) for the `objdump` workflow.
4. Read [Spike Simulation and Debugging](simulation/spike-debugging.md) for execution and register inspection.

## Learning outcomes

Through these exercises, I practised the distinction between native compilation and cross-compilation, identified the role of target architecture and ABI options, inspected compiler-generated RISC-V instructions, ran a target executable in an ISA simulator, and investigated integer representation and boundary values. I also learned to treat compiler errors and debugger mistakes as part of the engineering workflow: correct the source or command, rebuild, and verify the new result.

## Next validation steps

- Record exact compiler, Spike, and `pk` versions from the workshop machine.
- Capture a fresh final disassembly after rebuilding the current source.
- Re-run the integer programs and save their complete outputs alongside the corresponding source revisions.
- Add later workshop exercises only after completing them and capturing their evidence.
