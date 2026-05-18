---
title: Operating Systems
type: docs
---

## Booting

- Power-on & CPU reset
- **Firmware stage (BIOS/UEFI)**: Execute BIOS/UEFI firmware instructions, which are stored in flash (non-volatile) memory on the motherboard.
  - POST (Power-On Self-Test): hardware checks
  - Memory controller initialization: bring up DRAM (main memory) for later use
  - Peripheral discovery: load devices
  - **Bootloader selection: decide where to fetch the bootloader firmware, load it into memory, and jumps to it to execute**
    - The bootloader is usually stored in the reserved first sector of the disk - **MBR (Master Boot Record)**. If the system needs to boot from USB, it may be stored in USB memory.
- **Bootloader stage**
  - Set CPU mode
  - **Load OS kernel image from disk into main memory**
    - The kernel is fully loaded into RAM as a whole without swapping. The entirety has to fit into memory and have enough space left over for other stuff, e.g. user space.
    - It could also be a second-stage bootloader instead of OS kernel.
  - Transfer control to the OS kernel
  - The bootloader's memory is usually freed and can be overwritten from this point on.
- **OS kernel initialization**
  - **Memory management setup**: (kernel) page table & virtual memory initialization (map kernel image to virtual addresses)
  - Device driver initialization: disk, GPU, network, etc.
  - **Mount root filesystem (enable file access)**
  - Spawn the first user-space process: `init`
- **`init` - user space initialization**
  - Launch system services and background applications
  - Spawn user-space processes on start-up (**including the terminal/shell**)

Notes:

- **The kernel space mapping is shared across all processes**: each process has its own user space mappings, but they all share the same kernel space. This kernel space is initialized by the OS kernel and contains the kernel code, data and other content. It is protected by privilege levels (**only accessible with privilege escalation - kernel mode**).
- The amount of physical memory for kernel space is not fixed: it grows and shrinks dynamically as needed, and the kernel page table is updated accordingly.

## Running a Program


- **Creating the process** (`fork()`) - allocate process resources
  - Allocate a **Process Control Block (PCB)** for the new process - PID, parent PID, registers/program counter, scheduling info, file descriptor table, pointers.
    - This lives in kernel space together with other PCBs mapped to the top half of the virtual address space. The same applies to other kernel data mentioned below.
    - The file descriptor table is copied from the parent process, both pointing to the same open file table entries *(a system-wide table)*. The reference counts in the open file table are updated.
  - Create a **page table** for the new process. **The entries point to the same physical pages as the parent (copy-on-write)**, and will be copied into a new page/frame if writes occur. 
  - Allocate other kernel resources (e.g. kernel stack)
    - **Each process has its own kernel stack in the kernel space.** Instead of running in a specialized process, the kernel image works similarly as a "module" where functions are called upon usage and use the kernel stack in each process.
- **Switching to the new process** (`execve()`) - update process information
  - Tear down the old process's memory mappings (page table entries)
  - Create memory mappings in the virtual address space: **user-space** text (code), data (globals), bss (uninitialized globals), heap, stack
  - Load dynamic linker if needed
  - **Push arguments into the new user stack** (`argc`, `argv`, `envp`)
  - Update program counter and stack pointer to point to the new user space
  - Switch to user mode - start executing the new program


## System Call - `read()`

- Switch to kernel mode
  - Put arguments (**system call number**, arguments) into registers
    - Arguments will never spill onto the stack - system calls take no more than 6 arguments
  - Execute a **trap instruction** (`syscall`, `svc`): switch CPU to kernel mode, store program counter and (callee-saved, not arguments) registers on the kernel stack, jump to the system call entry point
    - Storing the program counter and callee-saved registers works the same as calling a function
    - Execution is inside the kernel, but still on behalf of the same process (using its kernel stack)
  - The kernel checks the system call number and dispatches to the corresponding handler
- Call filesystem-specific read
  - Using information from PCB, the kernel finds the file descriptor table entry -> open file table entry -> offset, flags, file v-node
  - Call the read function of the filesystem using these parameters: **the file system fills a kernel buffer with data** (not the buffer in user space passed as an argument)
    - *How the file system fills the buffer is described below.*
  - Copy data to user space, update file offset
- Return to user mode
  - End of the trap instruction: set return value, restore registers and program counter from the kernel stack, switch CPU back to user mode

**What happens when the filesystem fills the kernel buffer with data?**

- **Check page cache**: The **kernel** maintains a cache of file contents in RAM, page by page. If the requested range of data is already cached, the kernel can just copy from cache - **the page cache is the kernel buffer**.
- **Cache miss - load from disk**: If data is not cached, the kernel asks the block device (disk) driver to read needed blocks into memory.
  - This is achieved via DMA transfer (write directly to a kernel buffer), and the kernel waits for a signal from the disk driver indicating that the transfer is complete.
  - The kernel buffer is later used for copying data to user space. **It is also used as the page cache.**

## Virtual Address Lookup

- **Translation Lookaside Buffer (TLB) in MMU**:
  - TLB: a hardware cache in MMU that maps VPN to PPN. Since the virtual space is process-specific, TLB entries need to be flushed upon context switches. Some TLB designs have tags to support multiple processes' translations.
  - TLB hit: directly return PPN + PPO
- **Page Table in DRAM**:
  - VPO is translated into PPO using multi-level page tables, where VPO is divided into multiple parts for each level
  - If found, return PPN + PPO, and TLB will be updated with the mapping for later use
  - If not found, page fault occurs
  - Some recently-accessed page table entries exist in the cache (SRAM). Such entries can be accessed more quickly than those in DRAM, improving translation speed
- **Page Fault - Swap Space (Disk)**:
  - The page needs to be loaded back into DRAM from the swap space on disk: pick a victim page, write it to swap space (if necessary), load the requested page into DRAM
  - Update page table and TLB, return PPN + PPO
  - Restart the instruction: page fault is transparent to the process


## Exceptions

- Exception handling:
    - CPU detects an exceptional condition and raises an exception
    - The exception frame is push onto the **kernel stack** of the process: program counter, registers, processor flags/status
    - Switch to privileged mode: flips into kernel mode, use kernel stack pointer
- Vector table lookup (maintained in OS):
    - The **exception vector number** comes from CPU hardware control logic
    - Use the exception vector number to index into the exception descriptor table, which holds the addresses of the handlers
- Exception handler:
    - PC is set to the handler address
    - Execution continues in kernel mode
- End of execution:
    - OS decides what to do next: resume the interrupted process, deliver a signal, terminate the process, schedule a different process
    - Use a special instruction to return from the exception handler: restore the program counter and other context

Notes:

- The exceptions in programming languages are **language-level exceptions**, which can be handled by user-defined handlers. If they are not handled, then the program terminates (or a default handler). **This works entirely in user space.**
- It is possible to have user-defined CPU exception handlers - they will be caught in OS and run in user space. Different OS has different ways of defining handlers. However, **it is dangerous to allow user-defined handlers at the CPU vector level.**


## Signals

- Signal generation:
    - Hardware exceptions are caught by the OS, and the kernel sends a signal to the process in the exception handler
    - Kernel events: timer, terminal input, child process exit
    - A process explicitly sends a signal (`kill()`)
- Trigger signal handler:
    - Each process has a **signal bitmap** maintained by the OS to keep track of pending signals
    - Each process has a **signal table**, which includes the signal handler addresses installed by the user via `signal()` or `sigaction()`
    - **The kernel modifies user space stack and program counter, with the same effect of setting up a stack frame of the handler** (old program counter, registers, etc.). When the program continues in user mode, the signal handler is executed
    - **The signal handler is run in user mode**
- End of handler:
    - Use a special instruction to return from the signal handler: restore the program counter and other context
