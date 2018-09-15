---
layout: post
title: "More on x86 Architecture"
date: 2018-09-15
---

CPU modes, CISC vs RISC, virtual memory.

# Processor modes

* Real mode
* Protected mode
* Long mode

## Real mode

16-bit registers, no memory protection.

## Protected mode

32-bit registers, includes memory protection mechanisms.

8-byte entries (descriptors) in _Global Descriptor Table_ specify
memory access flags (write, execute, protection ring) for each segment.
There's also a _Local Descriptor Table_, which contains memory segments
that are private to a specific program.

To reference a segment, a program uses a _segment selector_ that points at
a GDT or an LDT entry. The segment selector contains 16 bits —
2 for Request Priviledge Level (protection ring), 1 for GDT/LDT, 13 for index.

**Problems:** segmentation is hard to code against (programming languages
have a flat memory model).

**Enabling protected mode:** create GDT, set up GDTR, set up cr0,
far jump to the _start_ label.

## Long mode

64-bit registers, flat linear virtual addresses (the base of a segment is `0x00`,
the size of a segment is not limited).

`mov eax, ebx` zeroes the upper 32 bits of `rax` register.

# Instruction sets

**CISC** makes sense for hand-written assembly, however, the commands are
usually slower due to many microinstructions involved. On top of that, it
is harder to write an efficient compiler for **CISC**.

**RISC** is better for pipelining.

The boundary nowadays is vague: some RISC processors are incorporating
non-RISC instructions that are faster to do in hardware, etc.

# Virtual memory

An abstraction of the available physical storage resources (RAM, HDD).
Programs are isolated, which improves security and frees application programmers
from from having to manage a shared space.

## Pages

The virtual address space is divided into _pages_, contiguous blocks of
addresses (at least 4 kilobytes on modern systems). Address translation
is performed by a hardware **Memory Management Unit (MMU)** using
_page tables_.

Each page has certain access flags (readable, writeable, executable, private).
Pages that are not private are shared between processes.

In Linux, use `less /proc/$PID/maps` to see a list of memory pages
for a specific PID.

## Performance

Recently used mappings from virtual to physical addresses are stored in
a hardware cache (**Translation Lookaside Buffer (TLB)**). Cache misses
are expensive due to the way translation is performed.
