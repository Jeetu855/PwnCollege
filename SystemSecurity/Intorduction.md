![alt text](attachments/image.png)

![alt text](attachments/image1.png)

## Binary Files

ELF : Executable and Linkable Format, binary file format on Linux
It contains 
- Program and its data
- Describes how the program should be loaded(program/segment header)
- Contains metadata describing programs components

ELF Program Headers

Program headers specify information needed to prepare the program for execution. Most important entry types
- INTERP : defines the library that should be used  to load this ELF into memory
- LOAD : defines a part of the file that should be loaded into memory

Program headers are source of information when loading the file into the memory

To see program headers of a binary, we can use the 'readelf' utility

```sh
readelf -a binary
```

ELF Section Headers

A different view of the ELF with useful information for introspection, debugging etc.

Important Sections

- .text : executable code of our program 
- .plt & .got : used to resolve and dispatch library calls
- .data : used for preinitialized global writable data (such as global arrays with initial values)
- .rodata : used for global read only data(such as string constants)
- .bss : used for uninitialized global writable data(such as global array without initial values)

**Section headers are not necessary part of ELF, only segments(defined via program headers) are needed for loading and operation. Section headers are just metadata.**

Symbols 

Binaries and libraries that use dynamically loaded libraries rely symbols(names) to find libraries, resolve function calls into those libraries etc.

---

## Linux Process Loading

cat file.txt
1) A process is created
2) Cat is loaded
3) Cat is initialized
4) Cat is launched
5) Cat reads the envrionment and arguments
6) Cat does its things 
7) Cat terminates

We are interested in first 3 parts


##### Portrait of a Process

Every linux process has :

- state (running, waiting, stoppped, zombie)
- priority (and other scheduling information)
- parent, siblings, children
- shared resources (files, pipes, sockets)
- virtual memory space
- security context ( effective uid and gid, saved uid and gid, capabilities )

**But where do these process come from**

In linux, process propogate by mitosis.

fork and more recently clone are system calls that create a nearly exact copy of calling process, a parent and a child
Later the child process ususally uses the 'execve' syscall to replace itself with another process
Example: 
- we type /bin/cat in bash 
- bash forks itself into the old parent process and the child process
- the child process 'execve' /bin/cat, becoming /bin/cat

***In the loading process, we need to know, can we load?***

Before anything is loaded, the kernel checks for executable permissions. If file is not executable, 'execve' will fail

***If we have correct permissions, the kernel has to decide What to load?***

To figure out what to load, the linux  kernel reads the beginning of file(ie /bin/cat) and makes the descision

1) If the file starts with '#!'(shbang, the kernel will treat this as a script file), the kernel extracts the interpreter from the rest of that line and executes this interpreter with the original file as an argument 
2) If the file matches a format in '/proc/sys/fs/binfmt_misc', the kernel executes the interpreter specified for that format with the original file as an argument 
3) If the file is dynamically linekd ELF, the kernel reads the interpreter/loader defined in the ELF, loads the interpreter and original file and lets the interpreter take control.
4) If the file is a statically linked ELF, the kernel will load it
5) Other legacy file formats are checked for.

These can be recurssive!

**The file extension does not matter to know what type of file it is, only the beginning of file matters to figure out the type of file.**

Interpreter/loader can also be modified using a tool called patchelf
```sh
patchelf --set-interpreter /some/interpreter /binary/which/requires/new/interpreter
```

#### Dynamically Linked ELF: The loading process

1) The program and its interpreter are loaded by the kernel
2) The interpreter locates the libraries
    - LD_PRELOAD environment variable and anything in /etc/ld.so.preload
    - LD_LIBRARY_PATH environment variable (can be set in the shell)
    - DT_RUNPATH or DT_RPATH specified in the binary file(both can be modified with patchelf)
    - system-wide configuration(/etc/ld.so.conf)
    - /lib and /usr/lib
3) The interpreter loads the libraries
    - these libraries can depend on other libraries, causing more to be loaded
    - relocations updated


'strace' goes throgh a binary and prints out all the system calls

***Where is all this getting loaded to?***

Each linux process has a virtual memory space. It contains:
    - the binary
    - the libraries
    - the heap(for dynamically allocated memory)
    - the stack(for functions and local variables)
    - any memory specifically mapped to the program
    - some helper regions
    - kernel code in the "upper half" of memory (above 0x8000000000000000 on 64 bit architechtures)

Virtual memory is dedicated to our process
Physical memory shared amongst whole system

We can see this whole space by looking at /proc/self/maps

***The Standard C Library***

Libc is a library full of helper functions that is linked by almost every process
Provides functionality like 
- printf()
- scanf()
- socket()
- atoi()
- malloc()
- free()

#### Statically Linked ELF: The loading process

1) The binary is loaded

```sh
gcc -static file.c -o file
```

Statically linked files take up lot more space that dynamically linked files because all the libraries are inside the binary also bugs from libraries statically linked in will have to be patched by the application developer, and not the library developer. This often means that 0-day patch releases to vulnerable libraries will not cover applications statically linked to those libraries.

Also when compiling with '-static' flag, by default the program is loaded into the same address when its ran, so no PIE(Position Independent Executable)
To have statically linked file and also have PIE protection on, we need to compile with the flag '-static-pie'


### Process Initialization

After everything is loaded into memory, it is initialized
Every LEF binary can specify 'constructors' which are functions that run before the program is actually launched
'Constructos' run before program execution starts, before main is executed

We can specify our own constructor

```c
__attribute __((constructor)) void haha(){
    write(1,"haha!\n",6); // 1: this is file descriptor for stdout(standard output)
    // next is the string literal
    // 6 : number of bytes to write
}
int main(int argc, char **argv){
    printf("Hello world");
    return 0;
}
```
In this code, the constructor will be executed first and then our main function

---

## Program Interaction

