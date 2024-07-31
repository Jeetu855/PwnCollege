All roads lead to cpu

![alt text](attachments/image.png)

![alt text](attachments/image-1.png)

![alt text](attachments/image-2.png)

![alt text](attachments/image-3.png)

![alt text](attachments/image-4.png)

Assembly is text representation of binary.

Assembly was assembled into binary code not compiled 
Assembly is direct translation of binary code ingested by the cpu, so its very cpu architechture dependent

Dialects of Assembly
- x86
- arm
- mips
- risc-v

```txt
OPERATION OPERAND OPERAND
```

Sub dialects of x86
- Intel
- AT&T

Intel literally made the x86 architecture so their syntax is better and prefered

Computers speak binary cause of the logic gates. "on" and "off" are relatively easy to check for as
we only have to check for 2 voltage levels

Text encoding we use it ASCII
ASCII has evolved into UTF-8 used on 98% of web
A = 0x41
a = 0x61
1 = 0x31

Words are grouping of bytes
Architecture defines the word width
Nibble = 4bits
Byte = 8 bits
HalfWord/Word = 16 bits
Double Word/dword = 32 bits
Quad Word/qword = 64 bits

A 64 bit machine can work with 64 bits at a time 

##### Expressing Negative Numbers

One Idea : sign bit
Use the leftmost bit as sign bit
b00000011 = 3
b10000011 = -3
Drawback 1, we have 2 ways of expressing 0
0 = b00000000 = b10000000
Drawback 2, arithematic operations have to be signed aware
unsigned : b00000000 - 1 = 0 - 1 = 255 = b11111111
signed : = b00000000 - 1 = 0 - 1 = -1 = b10000001 (first bit represents sign, rest is same as positive number)

Clever but crazy approach
Negative numbers are represented as large positive numbers that they would corelate to

0 - 1 = b11111111 = 255 = -1
-1 -1 = b11111110 = 254 = -2

The leftmost bit here is still the signed bit(for easy testing of negative numbers)
Advantage : Arithematic operations dont have to be signed aware
Smallest expressible negative number(8 bits) = b100000000 = -128
Largest expressible negative number(8 bits) = b01111111 = 127
Also only 1 representation of 0

#### Registers

Registers are very fast, temporary stores for data
Register are typically the same size as the word width of architecture
On a 64 bit architecture most registers will hold 64 bits

Registers live in our cpu
Some registers can be accessed partially

![alt text](attachments/image-5.png)

We load data into registers using 'mov' instruction
'mov' dosent move data, it copies it

Consider 
```asm
move eax, -1
```
eax is now 0xffffffff(both 4294967295 and -1) but
rax is now 0x00000000ffffffff(only 4294967295)
What if we wanted rax to also have the same sign, can we extend that sign?

```asm
mov eax, -1
movsx rax, eax
```

'movsx' does a sign-extending move, preserving the 2's complement value(i.e copies the top bit to the rest of the register)
'eax' is now 0xffffffff(both 4294967295 and -1)
rax is now 0xffffffffffffffff(both 4294967295 and -1)

![alt text](attachments/image-6.png)

Some registers are special
Example rip : we cannot read from or write to it directly
contains address of next instruction to be excuted(instruction pointer)
 
rsp : Stack pointer contains address of a region of memory to store temporary

#### Memory

Each memory address refrences one byte in memory 
Process memory are addressed linearly
From 0x10000 
To : 0x7fffffffffff(for architecture/OS purpose)(we have to remember we cannot access top 4 hex as those are non cannonical address)
So we can only access first 12 hex of RAM which are the cannonical address
This means 127 terabytes of addressable RAM! 
How can we have this much memory address if we dont have this much RAM
We dont have 127Terabytes of RAM.. but thats ok cause its all virtual

##### Stack
If we want to store a value in memory on stack for temporary storage, we can push it onto the stack
Values can be popped right back off
***Even on 64 bit systems we can only push 32 bit immediate values***

Pop actually does not remove the value from the stack, it actually still there, we just redifine where the stack ends

Addressing the Stack

Stack is somewhere in memory at an address. 
The cpu knows where the stack is because its address is stored in rsp
Stack is in very high memory because it grows backwards, it grows towards smaller memory address
Push subtracts 8 from rsp, Pop adds 8 to rsp

The equivalent of push would be 
```asm
sub rsp, 8
mov [rsp], ourData
```

Each memory location contains 1 byte

Memory Endianess
Data on most mordern systems is stored backwards, little endian : LSB stored at smallest memory address
Bytes are only shuffled for multi bytes stores and loads of registers to memory
Individual bytes never have their bits shuffled

Address Calculation

Use rax as an offset off some base address(in this case the stack)

```asm
mov rax, 0
mov rbx, [rsp+rax*8]
inc rax
mov rcx, [rsp+rax*8]
```

We can get the calculated address using lea(load effective address)

Address calculation has a limit
reg + reg*(2 or 4 or 8) + value is as good as it gets

**lea is one of the few instructions that can directly access the rip register**

#### Control Flow

![alt text](attachments/image-7.png)

CPU executes instruction in sequential manner until and unless told not to
One way to interupt a sequence is with 'jmp' jump instruction

Conditional Jumps

![alt text](attachments/image-8.png)

Conditional jumps check conditions stored in the "flags" register : rflags

Flags are updated by :
- most arithematic instructions
- comparions instruction cmp(sub, but discards the result)
- comparison instruction test(and, but discards the result)

Main conditional flags
Carry flag : was the 65th bit 1
Zero flag : was the result 0
Overflow flag : did the result wrap between positive adn negative
Signed flag : was the results signed bit set(i.e was it negative)

Thanks to 2's complement, only jumps themselves have to bee signedness-aware

ja : jump if above(unsigned)
jg : jump if greater than(signed)

Same thing, its just 1 is signed and other is unsigned

Looping : with our conditional jumps, we can implement looping 

```asm
mov rax, 0
LOOP_HEADER
inc rax
cmp rax, 10
jb LOOP_HEADER // jump to the label if value is below 10
```

This is a program to count till 10

Function Calls

Assembly code is split into functions with 'call' and 'ret'
'call' pushes 'rip' and jumps away
'ret' pops 'rip' and jumps to it

![alt text](attachments/image-9.png)

Registers are shared between functions so calling conventions should agree on what registers are protected

#### System Calls

How do we interact with the outside world?
Even something as simple as quitting the program?

We use system call to exit
**System call : Its an instruction that makes a call into the operating system**

```asm
syscall
```

syscall triggers a system call specified by the value in rax
arguments in rdi, rsi, rdx, r10, r8, r9
return value in rax

![alt text](attachments/image-10.png)


The c code is equivalent to the assembly code, first move the arguements into the registers, then the syscall number in rax
Some system calls take string argument(for example file paths)
A string is a bunch of contiguous blokcs in memory followed by a null byte

![alt text](attachments/image-13.png)

![alt text](attachments/image-12.png)

Some system calls require constants as an argument

Finally we can quit

![alt text](attachments/image-15.png)

#### Building Programs

We have to tell the assembler what syntax we are using
".intel_syntax"
"noprefix" : this tells the assembler we will not prefix all register names with %

```asm
.global _start
_start:
.intel_syntax noprefix
mov rdi, 42 #our programs return code
mov rax, 60 #system call number of exit()
syscall # do the system call
```

```sh
gcc -nostdlib -o quitter quitter.s
```

![alt text](attachments/image-16.png)

We can view our programs return code using 
```sh
echo $?
```

To get the assebly back

```sh
objdump -M intel -d quitter
```

![alt text](attachments/image-17.png)

There is an instruction "int3", it itself is a breakpoint, it triggers a debugger with a breakpoint

Steps to solve the dojo

1) "as -o asm.o asm.S"   (run these commands in terminal without the quotes, asm.s is the file we wrote our code in)
2) "objcopy -O binary --only-section=.text asm.o asm.bin"
3) "cat ./asm.bin | /challenge/run"


```sh
as -o asm.o asm.S && objcopy -O binary --only-section=.text ./asm.o ./asm.bin && cat ./asm.bin | /challenge/run
```


Many instructions exist in x86 that allow you to do all the normal
math operations on registers and memory.

For shorthand, when we say A += B, it really means A = A + B.

Here are some useful instructions:

```asm
  add reg1, reg2       <=>     reg1 += reg2
  sub reg1, reg2       <=>     reg1 -= reg2
  imul reg1, reg2      <=>     reg1 *= reg2
```

***All 'regX' can be replaced by a constant or memory location***

f(x) = mx + b, where:
    m = rdi
    x = rsi
    b = rdx

there is an important difference between mul (unsigned
multiply) and imul (signed multiply) in terms of which
registers are used

```asm
    intel_syntax noprefix

    .global _start
    
    start:
    
    imul rdi, rsi
    add rdi, rdx
    mov rax, rdi
```

Division in x86 is more special than in normal math. Math in here is
called integer math. This means every value is a whole number.

As an example: 10 / 3 = 3 in integer math.

Why?

Because 3.33 is rounded down to an integer.

The relevant instructions for this level are:
  mov rax, reg1; div reg2

Note: div is a special instruction that can divide
a 128-bit dividend by a 64-bit divisor, while
storing both the quotient and the remainder, using only one register as an operand.
How does this complex div instruction work and operate on a
128-bit dividend (which is twice as large as a register)?

For the instruction: ***div reg***, the following happens:
  ***rax = rdx:rax / reg***
  ***rdx = remainder***
***rdx:rax means that rdx will be the upper 64-bits of the 128-bit dividend and rax will be the lower 64-bits of the 128-bit dividend.***
**You must be careful about what is in rdx and rax before you call div.**

Please compute the following:
  speed = distance / time, where:
    distance = rdi
    time = rsi
    speed = rax

distance will be at most a 64-bit value, so rdx should be 0 when dividing.h

```asm
.intel_syntax noprefix

.global _start

_start:

      mov rax, rdi
      div rsi
```

x86 allows you to get the remainder after a div operation.
For instance: 10 / 3 -> remainder = 1

The remainder is the same as modulo, which is also called the "mod" operator.

```asm
.intel_syntax noprefix

.global _start

_start:
      
      mov rax, rdi
      div rsi
      mov rax, rdx # rdx stores the remainder of the division operation
```

Another cool concept in x86 is the ability to independently access to lower register bytes.

We can also access the lower bytes of each register using different register names.

For example the lower 32 bits of rax can be accessed using eax, the lower 16 bits using ax,
the lower 8 bits using al.

MSB                                    LSB
+----------------------------------------+
|                   rax                  |
+--------------------+-------------------+
                     |        eax        |
                     +---------+---------+
                               |   ax    |
                               +----+----+
                               | ah | al |
                               +----+----+

Lower register bytes access is applicable to almost all registers.

Using only one move instruction, please set the upper 8 bits of the ax register to 0x42.



























