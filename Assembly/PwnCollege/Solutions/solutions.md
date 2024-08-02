```sh
as -o asm.o asm.S && objcopy -O binary --only-section=.text ./asm.o ./asm.bin && cat ./asm.bin | /challenge/run
```

Level 4:

```asm
    intel_syntax noprefix

    .global _start
    
    start:
    
    imul rdi, rsi
    add rdi, rdx
    mov rax, rdi
```

Level 5:

```asm
.intel_syntax noprefix

.global _start

_start:

      mov rax, rdi
      div rsi
```

Level 6:

```asm
.intel_syntax noprefix

.global _start

_start:
      
      mov rax, rdi
      div rsi
      mov rax, rdx # rdx stores the remainder of the division operation
```

Level 7:

```asm
.intel_syntax noprefix

.global _start

_start:

      mov ah, 0x42
```

Level 8:

```asm
.intel_syntax noprefix

.global _start

_start:
      
      mov al, dil
      mov bx, si
```

Level 9:

```asm
.intel_syntax noprefix

.global _start

_start:
      
      shr rdi, 32
      shl rdi, 56
      shr rdi, 56
      mov rax, rdi
```

Level 10:

```asm
.intel_syntax noprefix

.global _start

_start:
      
     xor rax, rax # rax becomes 0
     xor rax, rdi # rax now holds the value of rdi
     and rax, rsi # performent and since rax = rdi, and we do rax & rsi

```

Level 11:

```asm
.intel_syntax noprefix

.global _start

_start:
      
      xor rax, rax
      and rdi, 1
      xor rdi, 1
      xor rax, rdi

```

Level 12:

```asm
.intel_syntax noprefix

.global _start

_start:
      
      mov rax, [0x404000]
```

Level 13:

```asm
.intel_syntax noprefix

.global _start

_start:
      
      mov [0x404000], rax
```
Level 14:

```asm
.intel_syntax noprefix

.global _start

_start:
      
      mov rax, [0x404000]
      add qword ptr [0x404000], 0x1337
```

Level 15:

```asm
.intel_syntax noprefix

.global _start

_start:
      
      xor rax, rax
      mov al, [0x404000]
```

Level 16:

```asm
.intel_syntax noprefix

.global _start

_start:
      
      mov al, [0x404000]
      mov bx, [0x404000]
      mov ecx, [0x404000]
      mov rdx, [0x404000]
```

Level 17:

```asm
.intel_syntax noprefix

.global _start

_start:
      
      mov rax, 0xdeadbeef00001337
      mov [rdi], rax 
      mov rax, 0xc0ffee0000 
      mov dword ptr [rsi], eax # Move the lower 32 bits of rax to the memory location pointed by rsi
      #Move the higher 32 bits of rax to the memory location pointed by rsi+4
      shr rax, 32
      mov dword ptr [rsi + 4], eax
```

Level 18:

```asm
.intel_syntax noprefix

.global _start

_start:
      
      mov rax, [rdi]
      mov rdx, [rdi+8] # the 8th byte from the the base of rdi
      add rax, rdx
      mov [rsi], rax
```


Level 19:
```asm
.intel_syntax noprefix

.global _start

_start:
      
      pop rax
      sub rax, rdi
      push rax
```

Level 20:

```asm
.intel_syntax noprefix

.global _start

_start:
      
      push rdi
      push rsi
      pop rdi
      pop rsi

```

