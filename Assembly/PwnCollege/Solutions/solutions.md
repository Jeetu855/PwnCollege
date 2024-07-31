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

