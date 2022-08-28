---
author: Davide Quaranta
title: "Basics of MIPS Assembly"
date: 2018-08-01
categories: [computer-architecture]
tags: [mips, assembly, asm]
description: "We have a fairly dense post in front of us, in which we will see the basics of programming in MIPS assembly. By the end of the guide we will be able to do quite a bit (such as operating on vectors and matrices), and also ready to tackle recursion."
toc: true
---

This post has been written after attending the Computer Architectures course by prof. Andrea Sterbini at Sapienza University of Rome; contents are heavily based on that course.

Here is a roadmap of what we will see today:

- Specialized and non-specialized registers;
- Syscalls;
- Directives to the assembler;
- Instruction formats;
- Conditional and non-conditional jumps;
- Cycles;
- Vectors and arrays.

Programming in assembly is a totally different experience than programming with any high-level language. We have direct access to registers, and we can optimize (or make worse) code with a high level of precision.

In this post we will use [MARS](http://courses.missouristate.edu/KenVollmar/MARS/): a MIPS IDE that also integrates a simulator.

Without wasting any more time on small talk, let's get started.

## Registers

In assembly we have direct access to registers. In particular, we have 31 registers (plus those about floating-point data), which conventionally are used to do specific things.

Here are the standard MIPS registers:

| #             | Name        | Description or conventional use                                     | Notes                                           |
| ------------- | ----------- | ------------------------------------------------------------------- | ----------------------------------------------- |
| `$0`          | `$zero`     |                                                                     | Contains only the value `0`                     |
| `$1`          | `$at`       |                                                                     | Reserved for pseudo-instructions                |
| `$2`-`$3`     | `$v0`-`$v1` | Function return values                                              |
| `$4`-`$7`     | `$a0`-`$a3` | Arguments of functions, not preserved between functions             |
| `$8`-`$15`    | `$t0`-`$t7` | Temporary data, not preserved between functions                     |
| `$16`-`$23`   | `$s0`-`$s7` |                                                                     | Saved data, preserved between functions         |
| `$24`-`$25`   | `$t8`-`$t9` |                                                                     | Temporary data, not preserved between functions |
| `$26`-`$27`   | `$k0`-`$k1` |                                                                     | Preserved for kernel                            |
| `$28`         | `$gp`       | Global Area Pointer (data segment base)                             |
| `$29`         | `$sp`       | Stack Pointer                                                       |
| `$30`         | `$fp`       | Frame Pointer                                                       |
| `$31`         | `$ra`       | Return address                                                      |
| `$f0`-`$f3`   | -           | Floating point return values                                        |
| `$f4`-`$f10`  | -           | Temporary floating-point registers, not preserved between functions |
| `$f12`-`$f14` | -           | First two floating-point arguments, not preserved between functions |
| `$f16`-`$f18` | -           | Temporary floating-point data, not preserved between functions      |
| `$f20`-`$f31` | -           | Saved floating-point data, preserved between functions              |

A couple of notes:

- In the code we can use either register number (e.g. `$0`) or name (e.g. `$zero`);
- The difference between preserved and non-preserved data between functions is only conventional, and does not concern behavior that happens automatically. To give an example, it means that *by convention* you should not use temporary registers to save or pass data between functions;
- The register with the constant `$zero` exists because it is very useful.

### Examples

Before we get on with the formats, let's see some examples of instructions in ASM MIPS involving registers:

```
# $t0 = $t0 + $t1
add $t0, $t0, $t1

# $t0 = $t0 + 1
add $t0, $t0, 1

# $s0 = $t2
move $s0, $t2

# $t1 = read_mem($sp + 4)
lw $t1, 4($sp)
```

A few words about these examples:

- We found that the comments begin with `#`;
- We found that the syntax is quite simple: it may or may not always be in the style `instruction_code op1, op2 [, op3]`;
- To use constant values there is an *i* at the end of the instructions (we will see why);
- The last instruction (`load word`) reads from memory at the address given by `$sp` + 4 bytes (offset 4), and saves it in `$t1`; we can also see the situation as accessing an array that starts at the address contained in `$sp`, and our vector is 32-bit word (4 bytes), accessing byte 4 means logically accessing index 1.

Nice huh?

## Syscall

Syscalls are literally *calls to the operating system*, which are primarily for input and output operations. An example might be reading characters from the keyboard, and writing something on the screen.

There are several types of syscalls, identified by a number, and they work like this:

1. We load the syscall code into an appropriate register;
2. We load any values into appropriate argument registers;
3. We call the syscall;
4. We retrieve any return values from the appropriate result registers.

A very quiet example for printing a number:

```
addi $a0, $0, 42 # alternative to move $a0, 42
li $v0, 1 # load service "print integer"
syscall # call syscall 1
# result: print 42
```

So, I understand that at this instant the situation may be a little confusing, but it is intended. We will now put the pieces back together with a nice list of syscalls that MARS provides, along with their descriptions and registers involved.

| Service         | Code  | Arguments                                                        | Result               |
| --------------- | :---: | ---------------------------------------------------------------- | -------------------- |
| print integer   |  `1`  | `$a0` = integer to print                                         |                      |
| print float     |  `2`  | `$f12` = float to print                                          |                      |
| print double    |  `3`  | `$f12` = double to print                                         |                      |
| print string    |  `4`  | `$a0` = string address (`NULL`-terminated) to print              |                      |
| read integer    |  `5`  |                                                                  | `$v0` ← integer read |
| read float      |  `6`  |                                                                  | `$f0` ← float read   |
| read double     |  `7`  |                                                                  | `$f0` ← double read  |
| read string     |  `8`  | `$a0` = input buffer address<br />`$a1` = max characters to read |                      |
| exit            | `10`  |                                                                  |                      |
| print character | `11`  | `$a0` = character to print                                       |                      |
| read character  | `12`  |                                                                  |                      |

This list is incomplete, because it includes only the syscalls that we will use most frequently in this series of articles. You can read the complete list in the MARS documentation (press `F1`), or in the [online documentation](https://courses.missouristate.edu/KenVollmar/mars/Help/SyscallHelp.html).

Examples of other syscalls are those to generate random numbers, manage files, ask for the date, make windows appear, and cast spells.

### Example

*Before trying this example, disable popup windows for keyboard input; find the option under Settings*.

Let's pretend we want to take two numbers from the keyboard, add them up, and spit out the result. In doing so, we also want to facilitate the user with guidance messages, such as "Enter number:" and "The result is:".

We could do something like this (also available [also on GitHub](https://github.com/davquar/Architetture2/blob/master/0.%20Simple%20things/add-numbers.asm)):

```
.globl main

.data
  prompt1: .asciiz "Integer 1: "
  prompt2: .asciiz "Integer 2: "
  resultDescr: .asciiz "The result is "

.text

main:
  # prompt1
  the $a0, prompt1
  li $v0, 4
  syscall

  # read number 1
  li $v0, 5
  syscall
	
  # move it to $t0
  move $t0, $v0
	
  # prompt number 2
  la $a0, prompt2
  li $v0, 4
  syscall
	
  # read number 2
  li $v0, 5
  syscall

  # sum the two numbers
  addu $a1, $v0, $t0
	
  # save result text
  the $a0, resultDescr
  li $v0, 4
  syscall
	
  # load result where needed
  move $a0, $a1
	
  # print integer
  li $v0, 1
  syscall
	
  # terminate program
  li $v0, 10
  syscall
```

In this example we also introduced some new little things. Let's see.

- Directives:
  - `.globl main` tells the assembler that the `main` symbol can also be accessed from another file;
  - `.data` delimits the beginning of the `data` segment, which is the container of static data in the object file:
    - `.asciiz` indicates that the string in quotes is ASCII and terminated by `NULL` (byte `0`).
  - `.text` delimits the beginning of the `text` segment, that is, the container of the instructions in the object file.
- Symbols:
  - `main` is in this case our entry point.

When executed, the program does this:

1. With `load address (la)` we load into `$a0` the string address of the first prompt;
2. We load the `print string (4)` service with the `load immediate (li)`;
3. We execute the syscall to bring up the string;
4. We load the `read integer (4)` service and run the syscall;
5. Now the read number is available in `$v0`, but for convenience we copy it to `$t0`;
6. We repeat steps 1-4 for the second prompt and number;
7. We sum the numbers with an `addu (add unsigned)` to avoid having to handle any overflow. In case we want to, we can use a simple `add`;

The rest is quite understandable having analyzed what happens before. We end execution with the `10` syscall.

## Directives to the assembler

We saw a couple of these a little while ago, and now we explain them in general. With directives we tell the assembler:

- How to handle certain things (e.g., alignment of values to byte);
- How to prepare the object file (e.g., start of segments);
- What to put in it (e.g., static data);
- Whether to use more *human* names for registers;
- Whether to group a sequence of instructions in *macro*.

Directives begin with a period, and the ones we will use are:

| Directive                 | Description                                              |
| ------------------------- | -------------------------------------------------------- |
| `.text`                   | Beginning of the instruction block                       |
| `.globl x`                | Indicates that label `x` is accessible from another file |
| `.data`                   | Start of static data block                               |
| `.eqv $name, $reg`        | Allows us to use `$name` to refer to `$reg`              |
| `.macro` and `.end_macro` | Defines a macro                                          |

Within the `.data` block we can define static data in these ways:

```
.data
label: .type val1, val2, ..., valn # n comma separated values
label: .type val:n # n repeated values 
```

Where `.directive` can be any of these:

| Directive  | Usage                                     | Example                |
| ---------- | ----------------------------------------- | ---------------------- |
| `.align k` | Align the next data to the power `k^2`    | `.align 2`             |
| `.space`   | Reserve `n` bytes                         | `s: .space 255`        |
| `.word`    | Allocates space for some words            | `n: .word 1,2,3`       |
| `.half`    | Allocates space for some half-word        | `h: .half 0:10`        |
| `.asciiz`  | Allocates an ASCII text terminated by `0` | `txt: .asciiz "Hello"` |
| `.ascii`   | Allocates an ASCII text                   | `txt: ascii "Hello"`   |

We need to say a few words about the `.align` directive.
If we use it, the assembler will cause all data to be *aligned* by a certain number of bytes, given by `2^k`. In other words, the start positions of the data will be aligned to an accuracy of `2^k`.

Usually `.align 2` is used to align data at multiples of `4`. The reason is simple: since the hardware is optimized to transfer words (4 bytes), we make it easier to retrieve them (the start position of the data is easily computed, because it has precision of one word).

The downside of this directive is that it inevitably creates holes in memory.

The **macros** allow us to define blocks of code as if they were functions. During assembly, they are replaced by the code they contain.

An example of a macro might be:

```
.macro takeInt (%regDst)
	li $v0, 5
	syscall
	move %regDst, $v0
.end_macro
```

This macro generalizes the behavior of taking an integer as input, as we saw earlier. The macros are called in the code as if they were high-level functions, so:

```
# ...
takeInt ($s0)
# ...
```

Nice.

## Let's stop for a moment

Before we move on to jumps (which are super cute), we have to deal with something boring and ugly, but we need it to understand why in MIPS we can't do virtuous things like `lw $t0($t1)`, and also to understand what that *i* means in instructions like `adds`, `subs`, `mules`, etc.

So I would say take a break, where you do some exercises to consolidate these few bits of knowledge we've seen. Some ideas:

- Create any variation of the example seen earlier;
- Take as input two integers and a separator character, and print these things separated by the given character:
  - Number 1;
  - Number 2;
  - Their product;
  - Their sum + 42.

## Instructions format

Here we go again.

~~This part goes hand in hand with the [single-clock-cycle MIPS CPU scheme](../cpu-1-clock-cycle), so if you haven't seen it yet I suggest you do it about now.~~ (I have not translated the article yet)

We should know that the instructions we use in ASM MIPS are an abstraction that masks us writing sequences of bits; at a low level, in fact, an instruction is a sequence of bits that is fed to the CPU.

At appropriate stages of execution, equally appropriate functional units of the CPU take the bits of the instruction they need (based on the active control signals).

I have spoken in a rather obfuscated way here, because these are things we see more clearly in the theory article I linked to a few lines ago.

We can also see that MIPS instructions can be divided into 4 groups, based on how they work. In fact, we have instructions that:

- Operate only on registers;
- Operate on registers, and constants;
- Make jumps:
  - Under certain conditions;
  - In any case.

Hence the existence of 3 encodings of MIPS instructions.
I true: the numbers do not add up; the reason is that we can summarize those 4 behaviors in only 3 encodings.

### R (Register) encoding.

These instructions operate only on registers (e.g. `add`, `move`), and their format is this:

|      |  oc   |  rs   |  rt   |  rd   | shamt |
| ---: | :---: | :---: | :---: | :---: | :---: |
|  bit |   6   |   5   |   5   |   5   |   5   | 5 |

Some instructions of type `R` are `add`, `sub`, `and`, `xor`, `sra`, `jal`.

### Encoding I (Immediate)

Here is finally the meaning of that *i*. In MIPS, instructions that have a space for a numeric constant are called *immediate*.

|      |  oc   |  rs   |  rt   |  imm  |
| ---: | :---: | :---: | :---: | :---: |
|  bit |   6   |   5   |   5   |  16   |

In this category also fall instructions that execute *conditional jumps*, i.e. those of the `branch` family (we will see them in a moment).

Some examples of instructions of type `I` are `addi`, `andi`, `slt`, `bne`, `lui`, `lw`, `sw`, `lb`.

### Encoding J (Jump)

Here we have unconditional jumps, i.e., those that are always executed, without the need to satisfy a condition.

|      |  oc   | dest  |
| ---: | :---: | :---: |
|  bit |   6   |  26   |

We have only two such instructions: `j` and `jal.`

Well, we got that part out of the way.

## Jumps.

We have just introduced them, and now we talk more about them.

As mentioned earlier, we have two types of jumps:

- Conditional: **branch**;
- Unconditioned: **jump**.

### Unconditioned jumps.

This type of jump is very simple. Basically, whenever an instruction of this type is encountered, we jump to the indicated destination.

The syntax is:

```
type_jump label
```

Where `label` is the label denoting the instruction to jump to. See:

| Jump type | To voice      | Usage       |
| --------- | ------------- | ----------- |
| `j`       | Jump          | `j label`   |
| `jal`     | Jump and Link | `jal label` |

The `jal` saves the contents of the `program counter (PC)` in `$ra`, and is used to call functions by saving the address from which to resume execution when finished. We will see more in the next article.

### Branch

Branches are those that in a high-level language correspond to **if**.

In MIPS we do not have the same flexibility of that type of construct, but it is much more rudimentary. The syntax here is:

```
type_branch $reg1, $reg2, label
```

In which the two registers are compared based on the type of *branch* operation, and if positive, execution jumps to the instruction denoted by `label`.

There are branches in which `0` is compared, and they have a more compact syntax.

| Branch type | To item                     | Usage                 | Jump if `op1 ? op2` |
| ----------- | --------------------------- | --------------------- | ------------------- |
| `beq`       | Branch if equal             | `beq $t0, $t1, label` | `==`                |
| `bne`       | Branch if not equal         | `bne $t0, $t1, label` | `!=`                |
| `blt`       | Branch if less than         | `blt $t0, $t1, label` | `<`                 |
| `ble`       | Branch if less or equal     | `ble $t0, $t1, label` | `<=`                |
| `bgt`       | Branch if greater than      | `bgt $t0, $t1, label` | `>`                 |
| `bge`       | Branch if greater or equal  | `bge $t0, $t1, label` | `>=`                |
| `beqz`      | Branch if equal to zero     | `beqz $t0, label`     |
| `bnez`      | Branch if not equal to zero | `bne $t0, label`      |                     |

* Can you think of a way to use a branch to always jump?

PRO TIP: Save the [MIPS Reference Card](http://www.cburch.com/cs/330/reading/mips-ref.pdf).

## Cycles

In MIPS we do not have instructions that allow us purely to create loops, but we can make them **combining banch and jump**.

Depending on the type of loop we want, we have patterns.

Some notes:

- I use `branch` as a placeholder for `beq`, `bne`, etc.;
- I use dummy labels (such as `do`, `while`, `for`) to show similarities, but you can use unique, context-appropriate labels.

#### Do-While

If at a high level we want:

```c
do {
  // stuff
} while (cond)
```

In MIPS we have:

```
do:
  # stuff
  branch $a, $b, do
```

#### While

High level:

```c
while (cond) {
  // stuff
}
```

MIPS:

```
while:
  branch (not_cond), endWhile.
  # stuff
  j while
endWhile:
# rest of program
```

Note that we need to negate the condition.

#### For

High level:

```c
for (i; i<n, i++) {
  // stuff
}
```

MIPS:

```
for:
  branch not_cond endFor.
  # stuff
  addi $i, $i, 1
  j for
endFor:
# rest of program
```

Also here from the conceptual point of view we have to negate the initial condition.

## Let's (again) take a break.

Before killing ourselves with vectors, it pays to pause and do some exercises.
Again, here are some ideas:

- **Calculator**: given as input two integers and an operation encoded as an integer, print the result of the calculation. *Then compare with [this example](https://github.com/davquar/Architetture2/blob/master/0.%20Simple%20things/add-or-mul-or-div-numbers.asm)*;
- **Cumulative Sum**:
  - Take as input a number `n`;
  - Take as input `n` integers, and as you go add them to register `$s0`;
  - Print the result.
- **strlen**: Print the length of a string taken as input;
- **str2int**:
  - Take a string and iterate over it;
  - For each character, prints its integer value *(hint: ASCII table)*;
  - If the character is uppercase *(same hint)*, add its *integer value* to a comulative sum;
  - When finished, print the comulative sum.

## Vectors and Matrices.

A vector is a data structure in which there is contiguous data of the same length. They are easily imagined in memory as a chopped segment.

What you need to know is that access to a vector is direct: it means that given a starting address (base) and an offset (offset), we access the position we are interested in directly, without going through the previous ones (which happens, for example, in lists).

For example, reasoning in C-like:

```c
type vector[2] <=> vector + 2*sizeof(type)
```

In MIPS assembly, then, we need only replicate the reasoning just seen: to access an element we must:

1. Get the current index *i*;
2. Multiply it by the size of the type;
3. Add it to the starting address of the vector.

As an example, suppose we want to access the position `2` of a vector of integers `V`:

```
.eqv $i, $t0 # index
.eqv $offs, $t1 # offset
.eqv $data, $t2 # data read

addi $i, $0, 2 # i=2
sll $offs, $i, 2 # offset=i*4
lw $data, V($offs) # data=read_mem(V+offset)
```

Note the `sll` line: we do the *shift left logical* by 2 positions because it is equivalent to multiplying the index by 4, which is the size in bytes of an integer (word).

If, on the other hand, we operate on a vector of bytes (a string, for example), there is no need to multiply.

The situation is a bit more complicated for **matrices**, because we have to forget about the very convenient access with rows and columns, because they do not exist. We have to imagine that we have a very long row that contains all the "rows" of the matrix.

We realize that to access an element we have to:

1. Take the base;
2. Calculate the vertical offset (row offset): `row*dimRow`;
3. Calculate the offset horizontally (column offset): `column*sizeof(type)`;
4. Sum the two offsets to the base.

Thus we have that:

```c
int M[X][Y];

M[x][y] <=> M + y*Y + x*sizeof(int)
```

*You have [some examples](https://github.com/davquar/Architetture2/tree/master/1.%20Vectors) available for both vectors and matrices.*

## Conclusions.

MIPS is nice, isn't it?

If you want to be cool, try doing some branch reduction if needed: restructure your code to avoid unnecessary branches.