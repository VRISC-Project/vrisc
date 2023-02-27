# Instruction Set

## Basic Instruction Set

Basic instruction set ('instruct set' will be aliases by 'ins' after time) occupies opcode `0` to `63`,
which gives managing instructions, the most basic arithmetic and logic options and memory r/w instructions.

One instruction is consist of two parts:

* opcode: The number map to instruction name.
* oprand: Oprands has three types: register oprand, immediate number and memory oprand. Most of the
  instructions have only register oprands, while only nessesary instructions have imm oprands and
  memory oprands.

> Opcode with `*` means this is a system managing instruction, must be used in kernel mode, or CPU will
> push a `permission denied interrupt`.

code|instruction form|encode|name|explanation
:-:|:-|:-|:-|:-
0       |nop                 |00                                    |no option   |stop CPU until an interrupt occurs.
1       |add r1, r2, r3      |01 r1[4],r2[4] r3[4]                  |add
2       |sub r1, r2, r3      |02 r1[4],r2[4] r3[4]                  |subtraction
3       |inc r1              |03 r1[4]                              |increment
4       |dec r1              |04 r2[4]                              |decrement
5       |cmp r1, r2          |05 r1[4],r2[4]                        |comparision
6       |and r1, r2, r3      |06 r1[4],r2[4] r3[4]                  |and option
7       |or r1, r2, r3       |07 r1[4],r2[4] r3[4]                  |or option
8       |not r1, r2          |08 r1[4],r2[4]                        |not option
9       |xor r1, r2, r3      |09 r1[4],r2[4] r3[4]                  |xor option
10      |jc imm              |0a 16/32/64[4],cond[4] imm[16/32/64]  |jump with condition
11      |cc imm              |0b 16/32/64[4],cond[4] imm[16/32/64]  |function call with condition|the return address stores in x0
12      |r                   |0c                                    |return from function
13*     |ir imm              |0d mod[8]                             |return from interrupt function
14      |sysc                |0e                                    |stick in kernel mode
15*     |sysr                |0f                                    |return to user mode
16      |loop r, imm         |10 r1[8],imm[32]                      |loop with condition|jump to relative address when r1==true
17      |shl r1, r2          |11 r1[4],r2[4]                        |left shift  |left shift r2 by r1 time
18      |shr r1, r2          |12 r1[4],r2[4]                        |right shift
19      |rol r1, r2          |13 r1[4],r2[4]                        |left rolling
20      |ror r1, r2          |14 r1[4],r2[4]                        |right rolling
21      |ldi imm, r          |15 8/16/32/64[4],r[4] imm[8/16/32/64] |load immediate number
22      |ldm r1, r2          |16 r1[4],r2[4]                        |load data from memory|load 8-byte data from address r1 to register r2
23      |stm r1, r2          |17 r1[4],r2[4]                        |store data to memory
24*     |ei                  |18                                    |enable interrupt
25*     |di                  |19                                    |disable interrupt
26*     |ep                  |1a                                    |enable paging
27*     |dp                  |1b                                    |diable paging
28      |mv r1/\*m1,r2/\*m2  |1c mvflg[8] r1[4],r2[4]               |move data   |move data r1 to r2
29*     |livt r              |1d r[8]                               |load interrupt vector table
30*     |lkpt r              |1e r[8]                               |load kernel mode paging table
31*     |lupt r              |1f r[8]                               |load user mode paging table
32,33   |                    |                                      |reserved
34      |initext r           |22 r[8]                               |initialize extension|r is instruction set code
35      |destext             |23                                    |destory extension
36*     |in imm/r1,r2        |24 r2[4],mvflg[4] imm[8]/r1[4]        |input from io port|imm/r1 for port code, input one byte to r2
37*     |out r1,imm/r2       |25 r1[4],mvflg[4] imm[8]/r2[4]        |output to io port |imm/r2 for port code, output one byte from r1
38      |cut r,imm           |26 r[4],imm[4]                        |cut register data
39      |icut r,imm          |27 r[4],imm[4]                        |cut with sign
40      |iexp r,imm          |28 r[4],imm[4]                        |extend register data with sign|extend to 64 bits
41      |cpuid               |29                                    |get cpu information|see [cpuid](#cpuid)

* conditional instruction:

[jc]|[cc]
:-|:-
j       |c
je      |ce
jb      |cb
js      |cs
jne     |cne
jnb     |cnb
jns     |cns
jh      |ch
jl      |cl
jnh     |cnh
jnl     |cnl
jo      |co
jz      |cz

* condition code:

[cond]|explanation
:-:|:-
0       |non
1       |equal
2       |bigger
3       |smaller
4       |n-equal
5       |n-bigger
6       |n-smaller
7       |higher
8       |lower
9       |n-higher
a       |n-lower
b       |overflow
c       |zero

* interrupt returning code

[mod]|explanation
:-:|:-
0       |panic, restart cpu
1       |retry, re-run instruction
2       |skip, run from next instruction

* move instruction flags

[mvflg]|explanation
:-:|:-
0       |r1,r2
1       |r,m
2       |m,r
3       |m1,m2

### cpuid

cpuid instruction can get firmware information (or some special function).

> The `*` under x1 register means register x1 can be any other value.

parameter register|x0   |x1    |returned result
:-:|:-|:-|:-
|     |0     |0     |EOM string, mostly lengths 32 bytes, which will be restored in x1,x2,x3,x4.
|     |1     |*     |The number of core.
|     |2     |*     |Store core id into x0 register.
|     |3     |*     |print string that `x1` points to on terminal.

## Register

* Universal registers: `x0`~`x15`, instruction can only use these registers.
* Instruction pointer: ip, It's more reasonable to store this inst on software,
  than to store next inst on firmware.
* Flags register
  * ^0 equal flag
  * ^1 bigger flag
  * ^2 smaller flag
  * ^3 zero flag
  * ^4 sign flag
  * ^5 overflow flag
  * ^6 interrupt enable flag
  * ^7 paging enable flag
  * ^8 priority flag(0 for kernel and 1 for user mode)
  * ^9 higher flag
  * ^10 lower flag
* interrupt vector table register(ivt): points to the first address of ivt.
  (See [Interrupt](interrupt.md).)
* kernel/user mode page table register(k/upt): (See [Paging](#paging).)
* System call pointer: (See [System Call](#system-call))

## Paging

Paging enables by flags^7.

`kpt/upt`stores the top paging table.

CPU can access memory by vritual address area.

A normal page sizes 16K.

### Vritual address

With paging disabled, CPU uses physical address.

With paging enables, CPU uses virtual address.

Virtual address is a 64-bit address, contains the total virtual address area.
The highest bit is user flag, user mode when set, and kernel mode when reset.

bit sections in virtual address:

bit|item
:-|:-:
63      |user flag
62~54   |reserved
53~44   |page table entry level 4
43~34   |page table entry level 3
33~24   |page table entry level 2
23~14   |page table entry level 1
13~0    |offset in a page

### page table & page table entry

A page table entry sizes 8 bytes. A page table has a maximum of 1024 page table entries.

The starting address of the page table can only be an integer multiple of 16K.

A page table entry is a physical address, which points to the lower level page table.
And page table entry level 1 points to a physical page.

When page table entries are less than 1024, the one after last entry must be 0, means
the page table terminated.

Because the lowest 14 bits in entry is always zero, we set some flags.

bit|item|explanation
:-|:-:|:-
13~2  |reserved       |
1     |big page flag  |The higher level entry points to a big page(See [bigpage](#bigpage).) when set.
0     |effective flag |This page is already swapped when reset.

### Bigpage

Bigpage is that a higher level entry points to.

When bigpage flag set, entry must be a physical address.

Bigpage sizes: 16M, 16G and 16T.There is no hardware limitation.

## system call

Operating system must prepare a function that will be called as system call.
There is no other aggrement.

## Instruction Extension

Use `initext` and `destext` to load and remove instruction extension.

### Basic algorithm extension(bae)

### Advances vector extension(ave)

### Single instruction multi data extension(simde)
