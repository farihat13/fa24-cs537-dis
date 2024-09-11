# Discussion 01

## Project 01 
- Git clone 
    - https://git.doit.wisc.edu/cdis/cs/courses/cs537/fall24/public/p1
- Play the game - board1.txt and test 1.in
    - https://letterboxed.io/home
- Write something simple - e.g. a program that just prints "Correct" should pass the first test.
    - ./run-tests.sh -v
- Write a new test - just choose an error message from the spec
    - ./run-tests.sh -t 4

## Project 02 (xv6) 


### Makefile

<!-- - Explain the basic mechanics of a Makefile.  -->

- A special file used to automate the process of compiling and building programs
- Makefiles usually have a bunch of definitions followed by target rules
- Rules specify how to build your project making it easier to compile large projects or recompile only the parts that have changed. 

**Variables:** e.g. CC, CFLAGS-common, CFLAGS-dbgs
- CC: specifies the compiler to be used
- CFLAGS-common: common compilation flags
    
    * -std=c17 tells the compiler to use the C17 standard.
    
    * Enable warnings (-Wall, -Wextra) and treat them as errors (-Werror)

    * Debug flags with lower optimization (-Og) and debugging information (-g).
    
- TARGET: The name of the program you want to create (letter-boxed)

- SRC: The source file to be compiled (letter-boxed.c)

**Rules:** tells how to build the target files from source files. 

- Each rule has the following structure:
    ```makefile
    target: dependencies
        commands
    ```
- Indentations need to be tabs
- `$@': target being generated
- `$<': first dependency
- Phony Targets (e.g. clean)


### Makefile for xv6

- https://git.doit.wisc.edu/cdis/cs/courses/cs537/fall24/public/discussion_material.git


1. Object Files (OBJS): compiled object files (.o files) that make up the kernel. 
    ```makefile
    OBJS = \
        bio.o\
        console.o\
        exec.o\
        ...
    ```

2. Conditional Tool Setup (TOOLPREFIX): if TOOLPREFIX is not defined, it tries to detect and set the correct prefix based on the environment. The TOOLPREFIX is used to support cross-compiling (e.g., compiling on a Mac for a different architecture like i386).
    ```makefile
    # TOOLPREFIX = i386-jos-elf
    ifndef TOOLPREFIX
    TOOLPREFIX := $(shell ...)
    endif
    ```

3. Compiler and Flags (CC, CFLAGS, LDFLAGS):
    ```makefile
    CC = $(TOOLPREFIX)gcc
    CFLAGS = -fno-pic -static -fno-builtin ...
    ```

4. Building Targets (xv6.img): builds the xv6.img disk image by combining the bootloader (bootblock) and kernel into a single disk image.

    ```makefile
    xv6.img: bootblock kernel
        dd if=/dev/zero of=xv6.img count=10000
        dd if=bootblock of=xv6.img conv=notrunc
        dd if=kernel of=xv6.img seek=1 conv=notrunc
    ```

5. Bootloader Compilation (bootblock): compiles the bootloader, which is the first code that runs when the system boots.
    ```makefile
    bootblock: bootasm.S bootmain.c
        $(CC) $(CFLAGS) -fno-pic -O -nostdinc -I. -c bootmain.c
        $(LD) $(LDFLAGS) -N -e start -Ttext 0x7C00 -o bootblock.o ...
    ```

6. Kernel Linking (kernel): links all the object files ($(OBJS), entry.o, etc.) and creates the kernel binary. The linker script (kernel.ld) controls the memory layout of the kernel.

    ```makefile
    kernel: $(OBJS) entry.o entryother initcode kernel.ld
        $(LD) $(LDFLAGS) -T kernel.ld -o kernel entry.o $(OBJS) ...
    ```    

7. Phony Targets (clean)

8. Running the Kernel (qemu): This target runs the compiled kernel inside the QEMU emulator. QEMU is a tool that allows you to emulate different hardware and run your operating system in a virtual environment. 
    ```makefile
    qemu: fs.img xv6.img
        $(QEMU) -serial mon:stdio $(QEMUOPTS)
    ```    

### QEMU
 <!-- What is QEMU? Important escape sequences to know about QEMU. Most important one to know about is Ctrl+a followed by x to terminate the qemu session. -->


QEMU is an open-source emulator and virtualizer. It allows you to create virtual machines on your computer. It can simulate a wide range of hardware systems (like different types of processors, storage devices, and networks), allowing you to run operating systems and software in a controlled environment without needing the actual physical hardware.

For example, if you're developing a custom operating system (like in the context of xv6 or a Linux kernel), you can use QEMU to test and run that operating system on your computer without installing it directly on your machine. This makes development safer and more convenient.

Key features:
- Emulator: It simulates hardware, allowing you to run software for different architectures (e.g., running ARM software on an x86 PC).
- Virtualizer: It can run virtual machines at near-native performance by using your system’s hardware features (like Intel VT-x or AMD-V).


### XV6 

- Go through `user.h` to explain the user-level functions available.
- Explain the flow of control that takes place during the syscall. Important files to show would be `trap.c`, `uSyS.s`, `syscall.c`,`sysproc.c`.
- Go through struct proc defined in `proc.c` and explain the relevant members as needed for this project.
- If you have extra time, you can consider introducing docker.
