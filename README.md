# ShobixRT

ShobixRT is a small preemptive real-time operating system for ARM Cortex-M, developed
from first principles in C and a small amount of ARM assembly.

The project begins on a Windows laptop using WSL2 Ubuntu, GNU Arm Embedded GCC, QEMU,
GDB, Make, and Python. QEMU will emulate an ARM Cortex-M4 development target so that
the firmware can be built, executed, inspected, and debugged without hardware. After
the QEMU version is stable, the same kernel will be ported to an STM32F411 Cortex-M4.

Project reference: [ShobixRT on GitHub](https://github.com/ANUKOOL324/ShobixRT)

## Project purpose

ShobixRT is both an operating-system implementation and an embedded-systems learning
project. Every subsystem is built incrementally so its purpose, memory behavior,
register interaction, runtime flow, and debugging method can be explained clearly.

The project is intended to demonstrate:

- ARM Cortex-M startup and exception behavior
- vector tables, linker scripts, `.data`, `.bss`, and stack placement
- task control blocks and independent task stacks
- cooperative and preemptive scheduling
- SysTick, PendSV, PSP, MSP, exception frames, and context switching
- READY, RUNNING, BLOCKED, and SLEEPING task states
- delays, software timers, semaphores, mutexes, and priority inversion
- bounded message queues and producer-consumer communication
- fixed-block memory management
- HardFault diagnostics and GDB-based debugging
- runtime profiling, stack usage, and scheduler measurements
- portability from QEMU to STM32 hardware

The implementation must be original. Existing RTOS source code is not copied into
ShobixRT. External documentation may be used to understand ARM Cortex-M, QEMU, GNU
tools, and STM32 hardware.

## Target architecture

```text
                         ShobixRT
                            |
        +-------------------+-------------------+
        |                                       |
  Hardware-independent kernel           Cortex-M architecture layer
        |                                       |
        |                         startup, exceptions, SysTick,
        |                         PendSV, PSP/MSP, context switching
        |
  +-----+------------------+
  |                        |
QEMU platform          STM32 platform
virtual peripherals    real peripherals and board setup
```

The kernel contains generic operating-system policy. The Cortex-M layer contains CPU
mechanisms shared by targets. The platform layers contain QEMU- or STM32-specific
peripheral code.

## Planned repository

```text
ShobixRT/
├── kernel/
│   ├── task.c/.h
│   ├── scheduler.c/.h
│   ├── mutex.c/.h
│   ├── semaphore.c/.h
│   ├── queue.c/.h
│   ├── timer.c/.h
│   └── memory.c/.h
├── arch/
│   └── cortex_m/
│       ├── startup.s
│       ├── context_switch.s
│       ├── exception.c
│       ├── cortex_m.h
│       └── registers.h
├── platform/
│   ├── qemu/
│   │   ├── uart.c/.h
│   │   ├── timer.c
│   │   └── platform.h
│   └── stm32/
│       ├── uart.c/.h
│       ├── timer.c
│       ├── gpio.c
│       └── platform.h
├── drivers/
├── apps/
├── profiler/
│   └── monitor.py
├── linker/
│   ├── qemu.ld
│   └── stm32.ld
├── tests/
├── docs/
├── Makefile
└── README.md
```

## Development stages

### Stage 0 - Environment and project structure

Define the repository boundaries, toolchain expectations, documentation system, and
design decisions.

### Stage 1 - Bare-metal ARM hello world

Build the smallest Cortex-M image with a vector table, reset handler, linker script,
and `main`. Boot it in QEMU and establish the GDB workflow.

### Stage 2 - UART output

Add the first platform console interface and prove that application output can be
produced through a target-specific backend.

### Stage 3 - SysTick and interrupts

Add exception handlers, the system tick, vector-table entries, and interrupt
diagnostics. Confirm the selected QEMU clock before choosing a tick interval.

### Stage 4 - Tasks and task control blocks

Define task identity, priority, state, stack ownership, wake time, and saved process
stack pointer.

### Stage 5 - Separate task stacks

Construct an initial Cortex-M exception frame for each task and verify stack layout in
GDB.

### Stage 6 - Cooperative scheduler

Implement a simple round-robin selection policy and explicit yielding before adding
interrupt-driven preemption.

### Stage 7 - Cortex-M context switching

Implement the first-task startup path and PendSV save/restore flow using PSP, MSP,
exception return, and the hardware-stacked register frame.

### Stage 8 - Preemptive scheduling

Use SysTick to update kernel time and request PendSV so scheduling work occurs at the
lowest-priority exception level.

### Stage 9 - Task states

Support READY, RUNNING, BLOCKED, and SLEEPING transitions with an idle task when no
application task is ready.

### Stage 10 - Delays and software timers

Implement `os_delay()` and tick-based wakeups without busy-waiting. Handle tick-counter
wraparound deliberately.

### Stage 11 - Semaphores

Add counting and event synchronization with task-specific wait queues.

### Stage 12 - Mutexes

Add ownership, blocking, unlock behavior, and a later priority-inheritance extension.

### Stage 13 - Message queues

Implement bounded ring-buffer queues with correct full, empty, producer, and consumer
blocking behavior.

### Stage 14 - Memory management

Start with deterministic fixed-size memory pools, allocation-failure statistics,
high-water marks, and optional debug patterns.

### Stage 15 - Fault diagnostics

Capture HardFault stack frames and inspect PC, LR, SP, xPSR, CFSR, HFSR, MMFAR, and
BFAR where supported.

### Stage 16 - Runtime profiler

Measure CPU usage, per-task runtime, context switches, stack headroom, interrupt count,
scheduler overhead, queue depth, and memory-pool use.

### Stage 17 - Python monitor

Stream structured statistics from the target and display task, timing, queue, memory,
and fault information on the host.

### Stage 18 - Testing and documentation

Add host-side tests for scheduler selection, queues, allocator bookkeeping, and other
portable logic. Add long-run stability tests and complete the technical documentation.

### Stage 19 - STM32F411 port

Replace the QEMU linker memory map and platform drivers with the exact STM32F411
memory map, clock configuration, UART, GPIO, and debug/flash workflow while keeping
the kernel API and most kernel logic unchanged.

## Engineering rules

- Build and test one stage at a time.
- Explain the problem, design, registers, memory, runtime flow, failure modes, and
  debugging method for every new feature.
- Keep hardware-independent policy in `kernel/`.
- Keep Cortex-M mechanisms in `arch/cortex_m/`.
- Keep board and emulator details in `platform/`.
- Investigate failures with compiler output, map files, disassembly, GDB, stack
  inspection, and QEMU logs before replacing code.
- Prefer explicit mechanisms over library calls that hide the behavior being learned.
- Record significant alternatives and decisions in the project documentation.
- Do not begin a later stage until the current stage builds, runs, and is documented.

## Validation plan

The completed project should demonstrate:

1. ARM cross-compilation and a booting QEMU image.
2. Multiple preemptively scheduled tasks.
3. A delayed task blocking while other tasks continue.
4. Producer-consumer communication through a bounded queue.
5. Mutex-protected shared output or resource access.
6. Runtime statistics for CPU use, switches, stacks, queues, and memory.
7. GDB inspection of PendSV, PSP, and a saved task frame.
8. A deliberate fault with captured register information.
9. The same kernel architecture running on STM32F411 hardware.

## Roadmap status

- Stage 0 - Environment and project structure: Complete
- Stage 1 - Bare-metal ARM hello world: Complete
- Stage 2 - UART output: In progress; runtime validation requires the documented GNU Arm toolchain and QEMU
- Stage 3 - SysTick and interrupts: In progress; basic 1 kHz SysTick support added
- Stage 4 onward: Not started

See `docs/stages/Stage-02-UART-Driver.md` for the QEMU CMSDK UART design,
commands, and validation record. The repository's current `.gitignore` excludes
`/docs/`, so that local stage note must be force-added deliberately if it should
be versioned.

## Current QEMU workflow

Build the Cortex-M4 firmware with the GNU Arm toolchain:

```sh
make clean
make
```

Run it on QEMU's `mps2-an386` board:

```sh
make run
```

UART0 output appears in the terminal. Press `Ctrl+C` to stop QEMU. Use
`make debug` to start QEMU paused for a GDB session on port 1234.
