# User-Space Thread Scheduler

A high-performance, multi-core user-space threading library built in C++17 and raw x86-64 assembly. This project expands upon the cooperative runtime foundation of Stanford CS 240's Chloros lab, extending it into a fully-featured concurrent threading system with preemptive multitasking, lock-free messaging, atomic synchronization primitives, and a multi-core work-stealing scheduler—all implemented without relying on `std::thread`.

---

## Features & Architecture

* **x86-64 Assembly Context Switching:** Custom assembly context switcher (`swtch.S`) that directly manages System V AMD64 ABI callee-saved registers (`rbx`, `rsp`, `rbp`, `r12–r15`) and instruction pointer manipulation.
* **Cooperative & Preemptive Multitasking:** Supports both explicit yield mechanics (`thread_yield`) and timer-driven signal preemption (`SIGALRM` / `sigaction`) to prevent long-running tasks from starving the scheduler.
* **Custom Atomic Synchronization:** Lock primitives (Spinlocks, Mutexes, Condition Variables) built from scratch using `std::atomic` and Compare-And-Swap (`compare_exchange`) loops with explicit acquire-release memory orderings.
* **Lock-Free Inter-Thread Messaging:** Single-Producer Single-Consumer (SPSC) ring buffer enabling thread-safe communication without mutex contention.
* **Work-Stealing Multi-Core Scheduler:** Per-core task deques featuring a Chase-Lev work-stealing algorithm where idle worker threads steal tasks from busy workers for dynamic load balancing.

---

## Stack Alignment & Runtime Mechanics

* **16-Byte Stack Alignment:** Custom stack allocation routines enforce strict 16-byte alignment prior to thread function invocation to meet SSE/SIMD instruction set requirements in x86-64 Unix environments.
* **Zero Kernel Transitions:** Context switches execute entirely in user space by saving and restoring minimal register contexts, drastically reducing switch latency compared to OS thread context switches.

---

## Repository Structure

```text
├── include/
│   ├── scheduler.h      # Thread scheduler & core API interfaces
│   ├── sync.h           # Mutex, Spinlock, and Condition Variable primitives
│   └── spsc_queue.h     # Lock-free SPSC ring buffer implementation
├── src/
│   ├── scheduler.cpp    # Round-robin & work-stealing queue logic
│   ├── swtch.S          # x86-64 assembly context switching routine
│   └── sync.cpp         # Atomic synchronization mechanics
├── tests/
│   ├── test_coop.cpp    # Cooperative yield test suite
│   ├── test_preempt.cpp # Signal-based preemption validation
│   └── test_sync.cpp    # Concurrency and lock-free queue stress tests
├── Makefile             # Build and sanitizer configuration
└── README.md
