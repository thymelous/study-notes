---
layout: post
title: "Introduction to Low Level Programming"
date: 2018-09-08
---

Linux, NASM.

# Architecture

Von Neumann:
* no interactivity
* no multitasking
* memory access is expensive

Intel 64:
* _Interrupts_ to allow interactivity
* _Virtual memory_ to allow multitasking
* _Registers_ to speed up memory access

## Registers

**r0**, **r1**, ..., **r7** are referred to by their historical names
(e.g. **rax**, accumulator, **rcx**, counter, and so on).

**r8** to **r15** are 64-bit only extensions and are referred to
by **r** + index.

**r_x** stands for a 64 bit register, **e_x** stands for lower 32 bits,
**_x** stands for lower 16 bits, **_h** and **_l** are higher and lower 8 bits
of **_x**.

Here's an illustrated example:

```
| -------- rax -------- |
        | ---- eax ---- |
             | -- ax -- |
             | ah || al |
```

# Executables

## Data

`dX`, where X is `b` for one byte, `w` for word (2b), `d` for double word (4b),
q for quad word (8b).

Example:
```nasm
label: db 'hello, world!', 10 ; 10 is the ASCII code for newline
```

Note that Intel CPUs are little-endian; a double world `0x11 0x22 0x33 0x44`,
for instance, is stored as `0x44 0x33 0x22 0x11`.

## Instructions

### Memory

`mov` — dst and src have the same size; cannot move from memory source to memory
destination, need to copy to a register.

Constant values — `mov rax, 10`, registers — `mov rax, rbx`, direct memory addressing —
`mov rax, [10]`, indirect — _address = base + index * scale + displacement_, _base_ is
the array head, _scale_ is the size of a single element, _displacement_ is the index
within the element.

When writing constant values to memory, we must specify the operand size:

```nasm
mov byte [buffer], 'x'
mov dword [buffer], 0
```

This is not required for register operands, since the register's name
already includes its size.

### System calls

[Linux system call table](https://syscalls.kernelgrok.com/)

```nasm
mov eax, 1   ; set the syscall #
mov ebx, ... ; set the arguments (check the syscall table)
syscall      ; execute system call
```

### Function calls

`call <address>` — pushes **rip** (instruction pointer) and jumps
to the address, `ret` — pops **rip**.

Arguments are passed via **rdi**, **rsi**, **rdx**, **rcx**, **r8**, **r9**,
and stack.

**rbx**, **rbp**, **rsp**, **r12-r15** are (conventionally) callee-saved,
i.e. the function must restore them. Others must be saved by the caller.

# Assembler

```nasm
section .data ; code is split into sections (see above)
message:      ; labels are converted to addresses
  db 'hhh', 10

section .text
_start:       ; _start marks the entry point
  mov rax, 1  ; syscall #1 is sys_write
  mov rdi, 1  ; 1 = stdout
  mov rsi, message
  mov rdx, 3  ; message length
  syscall
  mov rax, 60 ; syscall #60 is sys_exit
  mov ebx, 0  ; return code
  syscall
```

To compile, run `nasm -f elf64 listing.asm` (add `-g` to generate GDB debugging
symbols), then `ld -o exec listing.o` (where `exec` is the executable name).

## Debugging

Using GDB, the common flow is:

```
b _start # break at _start
r        # run
ni       # step to next instruction
```
