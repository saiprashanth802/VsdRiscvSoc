# 1. RISC-V Toolchain Setup Log

**Target**: Ubuntu 64-bit  
**Toolchain**: `riscv32-unknown-elf`  
**Purpose**: Setup, environment config, and sanity checks.

---

<details>
<summary><strong>[1] Decompress the Toolchain Archive</strong></summary>


```bash
cd ~/Downloads
tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz
```
</details>
<details> <summary><strong>[2] Modify the User Environment</strong></summary>
Edit your shell config (~/.bashrc or ~/.zshrc) and append:

```bash
export PATH="$HOME/Downloads/riscv/bin:$PATH"
```
</details>
<details> <summary><strong>[3] Sanity Check: Toolchain Version Output</strong></summary>

  Run the following:


```bash
riscv32-unknown-elf-gcc --version
```

 EXPECTED OUTPUT:

![WhatsApp Image 2025-06-07 at 16 54 44_2b2e332f](https://github.com/user-attachments/assets/f2cfe348-55d3-4c0b-83e5-81c00e08fc22)


```bash
riscv32-unknown-elf-objdump --version
```

EXPECTED OUTPUT:

![WhatsApp Image 2025-06-07 at 16 54 45_0301d28b](https://github.com/user-attachments/assets/89ba740d-d822-4c3a-8f90-323563417d19)



```bash
riscv32-unknown-elf-gdb --version
```
 EXPECTED OUTPUT:


![WhatsApp Image 2025-06-07 at 16 54 45_15d6d697](https://github.com/user-attachments/assets/fce4f32e-4a3a-4e47-9fbd-1aa17ebf3fea)

</details>

---

# 2. Compile “Hello, RISC-V”

The goal is to write a minimal C "Hello, World!" program, compile it using the RISC-V toolchain for the RV32IMC architecture, and verify the output ELF file.

---

<details>
<summary><b>Step 1: Write a minimal C program</b></summary>

Create a file named `hello.c`:
```c
gedit hello.c
```
Paste this code inside:

```c
#include <stdio.h>

int main() {
    printf("hello world\n");
    return 0;
}
```
Save and exit the editor

</details>

<details> <summary><b>Step 2: Compile using the RISC-V cross-compiler</b></summary>
Run this command in the terminal:

```bash

riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o hello_O0.elf hello.c
```
 If you get an error like "cannot find suitable multilib", it means your toolchain doesn't support rv32imc. You can try replacing with:

```bash
riscv32-unknown-elf-gcc  -o hello_O0.elf hello.c
```
</details>

<details> <summary><b>Step 3: Verify the output ELF file</b></summary>
Check the type of the file using:

```bash

file hello_O0.elf
```
Expected output (also displaying the possible error you may encounter):


![WhatsApp Image 2025-06-07 at 16 55 07_314669fe](https://github.com/user-attachments/assets/dd394606-7f8c-48a7-908e-5d013793558b)

</details>

---

# 3. From C to Assembly

### Goal
Take the same `hello.c` program and generate the assembly code (`.s` file), then understand what's happening in the function prologue/epilogue.

---

### Step-by-Step Instructions (Terminal Only)

<details>
<summary><strong> 1. Ensure your C file still exists</strong></summary>

Check your C source file:

```bash
cat hello.c

```

![image](https://github.com/user-attachments/assets/45adb943-8257-48d0-a8e0-3f666e3fd4a7)

</details>

<details> <summary><b>Step 2: Generate the assembly code</b></summary>
Use the compiler to generate a .s file (assembly code) from the C source:

```bash
riscv32-unknown-elf-gcc -S -O0 hello.c
```
This will produce a file named hello.s.

</details>

<details> <summary><b>Step 3: View the generated assembly</b></summary>
View the output using:

```bash

cat hello.s
```
You should see something like this:

![Screenshot from 2025-06-08 11-20-10](https://github.com/user-attachments/assets/d8a34c36-05a5-4e1f-8c9a-c0a1d75543d0)


</details>

<details> <summary><b>Step 4: Understand each assembly instruction</b></summary>

![image](https://github.com/user-attachments/assets/efa29835-f2a8-4a90-b82e-c2fc675b60cc)

The first two correspond to the prologue and the bottom three to the epilogue

 These instructions are automatically inserted to manage function call/return safely.

</details>

---
  
# Task 4: Hex Dump & Disassembly

The goal is to turn your compiled RISC-V ELF file (`hello.elf`) into:
1. A human-readable disassembly (addresses + instructions).
2. A raw hexadecimal file (Intel HEX format) that can be used for flashing or simulation.

---

<details>
<summary><b>Step 1: Disassemble the ELF file</b></summary>

Use `objdump` to convert your ELF binary into readable RISC-V assembly with addresses:

```bash
riscv32-unknown-elf-objdump -d hello.elf > hello.dump
```
To view it:

```bash
cat hello.dump
```
PARTIAL OUTPUT:

![image](https://github.com/user-attachments/assets/017bbe38-8dba-4ca5-8a5a-98a0a438b485)



100b4          -> Memory address(hex)  
1141           -> Raw machine code  
addi           -> mnemonic  
sp,sp,-16      -> operands  
addi sp,sp,-16 -> opcode  

</details>

<details> <summary><b>Step 2: Generate the raw hex format</b></summary>
  
Use objcopy to convert the ELF file into Intel HEX format:

```bash

riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
```

To view the hex contents:

```bash
cat hello.hex
```

PARTIAL OUTPUT:
![Screenshot from 2025-06-08 11-23-56](https://github.com/user-attachments/assets/65c95d81-46c0-40f0-9022-ad96235a329b)


This file represents your binary's memory layout and can be loaded into emulators or flashed to hardware.

</details>
</details> 

---

# Task 5: ABI & Register Cheat-Sheet



<details><summary><b>Step 1: List all 32 RV32 integer registers with their ABI names</b></summary><br>


| Register | ABI Name | Typical Role                      |
|----------|----------|---------------------------------|
| x0       | zero     | Hard-wired zero (always 0)      |
| x1       | ra       | Return address                  |
| x2       | sp       | Stack pointer                  |
| x3       | gp       | Global pointer                 |
| x4       | tp       | Thread pointer                 |
| x5       | t0       | Temporary / caller-saved       |
| x6       | t1       | Temporary / caller-saved       |
| x7       | t2       | Temporary / caller-saved       |
| x8       | s0/fp    | Saved register / frame pointer |
| x9       | s1       | Saved register                 |
| x10      | a0       | Function argument / return     |
| x11      | a1       | Function argument / return     |
| x12      | a2       | Function argument              |
| x13      | a3       | Function argument              |
| x14      | a4       | Function argument              |
| x15      | a5       | Function argument              |
| x16      | a6       | Function argument              |
| x17      | a7       | Function argument              |
| x18      | s2       | Saved register                 |
| x19      | s3       | Saved register                 |
| x20      | s4       | Saved register                 |
| x21      | s5       | Saved register                 |
| x22      | s6       | Saved register                 |
| x23      | s7       | Saved register                 |
| x24      | s8       | Saved register                 |
| x25      | s9       | Saved register                 |
| x26      | s10      | Saved register                 |
| x27      | s11      | Saved register                 |
| x28      | t3       | Temporary / caller-saved       |
| x29      | t4       | Temporary / caller-saved       |
| x30      | t5       | Temporary / caller-saved       |
| x31      | t6       | Temporary / caller-saved       |

</details>

<details><summary><b>Step 2: Calling Convention Summary</b></summary>

- **a0–a7 (x10–x17):** Used to pass function arguments and return values.  
- **s0–s11 (x8, x9, x18–x27):** Callee-saved registers. The function must save and restore these if it modifies them.  
- **t0–t6 (x5–x7, x28–x31):** Caller-saved temporaries. The caller must save these if needed after a function call.  
- **ra (x1):** Return address. Used to store the address to return to after a function call.  
- **sp (x2):** Stack pointer. Points to the current top of the stack.  
- **gp (x3):** Global pointer. Used for accessing global variables in some environments.  
- **tp (x4):** Thread pointer. Used for thread-local storage.  
- **zero (x0):** Always zero. Writes have no effect.

</details>

---

# Task 6: Emulating and Debugging hello.elf in QEMU using GDB


  
 <details> <summary><strong>Step 1: Start QEMU with debugging enabled</strong></summary>

```bash
qemu-system-riscv32 -nographic -machine sifive_e -kernel hello.elf -S -gdb tcp::1234
```
</details>

 <details> <summary><strong>Step 2: Connect GDB to QEMU</strong></summary>

  Open another terminal and run:

```bash
riscv32-unknown-elf-gdb hello.elf
```

Then inside GDB:

```gdb

(gdb) target remote :1234
```
You should see:

```cpp

Remote debugging using :1234
0x00001000 in ?? ()
```

</details> 

<details> <summary><strong>Step 3: Set breakpoint at <code>main</code> and continue</strong></summary>
  
```gdb
(gdb) break main
Breakpoint 1 at 0x1016a: file hello.c, line 5.
(gdb) continue
```
 Problem:
At this point, QEMU consistently froze after hitting "continue".

Attempting to interrupt using Ctrl+C resulted in:

```ruby
Program received signal SIGINT, Interrupt.
0x00000000 in ?? ()
```
Stepping through with stepi stayed stuck at 0x00000000:

```scss
(gdb) stepi
0x00000000 in ?? ()
```

This behavior indicates that debugging failed due to missing debug symbols or QEMU not progressing correctly.

</details>

<details><summary><b> MY OUTPUT: </b></summary>

![Screenshot from 2025-06-07 19-33-56](https://github.com/user-attachments/assets/ae517ac2-0b94-4ee1-984b-d7dea849bbb8)


![Screenshot from 2025-06-07 19-26-57](https://github.com/user-attachments/assets/0bb55233-e1e0-4e24-9c73-b9163a2276f6)



</details>

---

# Task 7: Running Under an Emulator

<b>Goal</b>

Run a bare-metal RISC-V ELF program using an emulator (Spike or QEMU) and see output via UART console.  
This is essential if you do not have access to real hardware yet.

---

<details>
<summary><b>Step 1: Compile your bare-metal program with debug symbols</b></summary>

Use the following command to compile your C program (`hello.c`) with the linker script (`linker.ld`) and include debug info:

```bash
riscv32-unknown-elf-gcc -g -nostdlib -nostartfiles -T linker.ld -o hello.elf hello.c
```
</details>

<details> <summary><b>Step 2: Run the ELF using QEMU</b></summary>
Use QEMU's RISC-V system emulator to run your ELF and get UART output:

```bash

qemu-system-riscv32 -nographic -machine sifive_e -kernel hello.elf
```
</details>

<details> <summary><b>Step 3: Debugging using GDB with QEMU</b></summary><br>
  
Start QEMU with GDB server enabled:

```bash
qemu-system-riscv32 -nographic -machine sifive_e -kernel hello.elf -S -gdb tcp::1234
```
-S tells QEMU to start paused (waits for GDB)

-gdb tcp::1234 opens TCP port 1234 for GDB remote debugging

In another terminal, start GDB:

```bash

riscv32-unknown-elf-gdb hello.elf
```

Connect to QEMU's GDB server:

```gdb
(gdb) target remote :1234
```

Use GDB commands:

- info registers
Shows the current values of CPU registers (e.g., ra, sp, gp, a0, etc.)

![image](https://github.com/user-attachments/assets/b48bd0f8-32a7-4dd7-9e36-831b55ee741f)


</details>

---

# Task 8: Exploring GCC Optimisation

**Question:**  
*“Compile the same file with -O0 vs -O2. What differences appear in the assembly and why?”*

---

### Goal

To observe how GCC optimizations affect assembly output by comparing two compiler optimization levels:  
`-O0` (no optimization)  
`-O2` (aggressive optimizations)




<details>
<summary><strong> Step 1: Write a test C program</strong></summary>

Use a slightly optimized example to make the changes clearly visible:

```c
// File: main.c
int square(int x) {
    return x * x;
}

int unused_function() {
    int y = 100;
    return y;
}

int main() {
    int a = 10;
    int b = square(a);
    return b;
}
```

</details>

<details> <summary><strong> Step 2: Compile with -O0 (no optimization)</strong></summary>

  
```bash
riscv32-unknown-elf-gcc -S -O0 main.c -o main_O0.s
```

This generates a verbose .s file with all functions and minimal optimization.

</details>

<details> <summary><strong> Step 3: Compile with -O2 (optimized)</strong></summary>

```bash
riscv32-unknown-elf-gcc -S -O2 main.c -o opt_O2.s
```

This will inline small functions, eliminate unused ones, and use registers more efficiently.
---![image](https://github.com/user-attachments/assets/a3e2f364-5292-445e-9eed-4df40d10cadf)

</details>

<details> <summary><strong> Step 4: Compare the two versions</strong></summary>

```bash
diff main_O0.s main_O2.s
```

You should see:

- unused_function removed

- square inlined (i.e., no separate function exists)

- Tighter, shorter assembly


| Feature                     | `-O0` Output (Verbose, No Optimizations) | `-O2` Output (Optimized, Compact) |
|----------------------------|-------------------------------------------|----------------------------------|
| Function: `square`         | Present as a standalone function          | **Removed, inlined into `main`** |
| Function: `unused_function`| Present in full                           | **Removed entirely**             |
| `main` function            | Calls `square` via `jal`                  | Does the multiplication inline   |
| Registers                  | Loads/stores to stack                     | Uses fewer registers             |
| Constant `int a = 10`      | Loaded into memory, then used             | Constant used directly           |
</details>

---

# Task 9: Inline Assembly Basics

<strong> Goal</strong>

  To write a function in C that returns the current value of the RISC-V cycle counter by accessing CSR register 0xC00 using inline assembly.

  ---

<details> <summary><strong>C Function Code</strong></summary><br>
  
![Screenshot from 2025-06-08 12-13-57](https://github.com/user-attachments/assets/3922cab3-ee3e-4057-8ff3-dfcfdfe15991)



</details>

<details> <summary><strong> Explanation of Each Part</strong></summary><br>
 
  <details> <summary><strong>static inline uint32_t rdcycle(void)</strong></summary><br>

- static inline: Makes the function inlineable and avoids multiple-definition errors.

- uint32_t: Returns a 32-bit unsigned integer.

- rdcycle: Custom function name.
  
  </details>  


 <details><summary><strong> asm volatile ("csrr %0, cycle" : "=r"(c));</strong></summary>

- asm volatile
   - asm: Embed assembly code.
   - volatile: Prevent compiler from optimizing this line away.
- "csrr %0, cycle"
   - csrr: Control and Status Register Read instruction.
   - cycle: Name for CSR 0xC00 (cycle counter).
   - %0: Placeholder for the first operand (here, the output variable c).
- : "=r"(c)
   - "=r": Output operand (= means write-only), stored in a general-purpose register (r).
   - (c): The C variable that receives the result.

</details>

</details>  

---

# 10. Memory-Mapped I/O Demo

<b>Problem Statement:</b>

“Show a bare-metal C snippet to toggle a GPIO register located at 0x10012000. How do I prevent the compiler from optimizing the store away?”

---

<details><summary><strong>Code Snippet</strong></summary><br>

![image](https://github.com/user-attachments/assets/86b52711-b5e3-4ce3-a357-2c6d03194443)


</details>

 <details> <summary><strong> Explanation</strong></summary><br>
   
- volatile:
  - Tells the compiler not to optimize access to the variable.
  - Required when accessing hardware registers or memory-mapped I/O.
  - Without it, compiler might skip the write because it sees no effect in program logic.

- (uint32_t*)0x10012000:
  - Casts the GPIO memory address to a pointer of type uint32_t*.
  - This assumes the hardware register is 4 bytes wide.

- *gpio = 0x1;:
  - Writes the value 0x1 to the register.
  - This could, for example, turn on a specific output pin.

- Alignment:
The address 0x10012000 is 4-byte aligned (divisible by 4), which is necessary for uint32_t access on many systems.

</details>

# 11. Linker Script 101

<strong> Problem Statement</strong>
“Provide a minimal linker script that places .text at 0x00000000 and .data at 0x10000000 for RV32IMC.”

---

</details> <details> <summary><strong>Minimal Linker Script</strong></summary><br>

![image](https://github.com/user-attachments/assets/525677bf-44ad-4195-88d0-bd40c79209a6)



</details> 

<details><summary><strong> Why Flash and SRAM Addresses Differ</strong></summary><br>
  
- Flash (e.g., 0x00000000):

  - Non-volatile memory.

  - Stores program code that must survive resets/power cycles.

- SRAM (e.g., 0x10000000):

  - Volatile memory used for runtime data.

  - Faster and writable, unlike Flash.

 During startup, boot code usually copies .data from Flash to SRAM and zeros .bss, so variables are correctly initialized.

</details>

---

# 12. Start-up Code & crt0

<strong>Question</strong><br>

“What does crt0.S typically do in a bare-metal RISC-V program and where do I get one?”

---

<details><summary><strong>What is `crt0.S`?</strong></summary><br>
  
- crt0.S is the "C Runtime Zero" file — the very first code that runs before main() in a bare-metal program.

- It's written in assembly and is architecture-specific.

- It performs system-level initialization that C programs assume has already been done.

</details> 

<details><summary><strong>Key Responsibilities of `crt0.S`</strong></summary><br>
  
- Set the Stack Pointer (sp)

  - Stack is essential for calling C functions.

  - crt0 sets sp to a known safe memory location (usually top of RAM).

- Zero out .bss section

  - .bss holds uninitialized global/static variables, which should start as 0.

  - crt0 clears this region with a loop.

- Copy .data from Flash to RAM

  - .data holds initialized globals.

  - crt0 copies them from ROM (Flash) to SRAM.

- Call main()

  - After setting everything up, crt0 branches to your main program.

  - Infinite Loop After main Returns

  - If main() returns, crt0 halts or spins forever to prevent undefined behavior.

</details> <details> <summary><strong>Where to Get `crt0.S`</strong></summary>

- Option 1: Write Your Own

  - Very short (usually ~30–50 lines of RISC-V assembly).

- Option 2: Use from a Runtime Library

- Newlib and libc implementations often include crt0.S.

- You can extract one from:

  - newlib source

  - SiFive’s freedom-e-sdk

  - Minimal RISC-V templates on GitHub (bare-metal-riscv)

</details> 

<details> <summary><strong>Minimal Example of `crt0.S`</strong></summary><br>

![image](https://github.com/user-attachments/assets/eeb2cf9c-6e64-4058-b81e-a6ddeb137c1f)


</details>

---

# 13. Machine Timer Interrupt (MTIP) Setup</strong></summary><br>

<b>Question</b>

“Demonstrate how to enable the machine-timer interrupt (MTIP) and write a simple handler in C/asm.”

---

<details><summary><b>Method</b></summary>

- Write to mtimecmp

- Set mie (enable machine timer interrupt)

- Set mstatus (enable global interrupt)

- Point mtvec to a valid handler

- Use __attribute__((interrupt)) in C

<b>Full Code (with Comments)</b><br>
![image](https://github.com/user-attachments/assets/44d49987-9faf-43c1-a652-5b8f47c7fe0a)


```c
   // File: mtip_example.c
#include <stdint.h>

// Assume these are memory-mapped addresses for your hardware
#define MTIMECMP_ADDR 0xBFF00000
#define MTIME_ADDR    0xBFF00008
#define COUNTER_ADDR  0xBFF10000

// Function to read a CSR (helper)
static inline uint32_t csr_read(uint32_t csr) {
    uint32_t value;
    asm volatile ("csrr %0, %1" : "=r"(value) : "i"(csr));
    return value;
}

// Function to write a CSR (helper)
static inline void csr_write(uint32_t csr, uint32_t value) {
    asm volatile ("csrw %0, %1" : : "i"(csr), "r"(value));
}

// MTIP handler
void mtip_handler(void) {
    volatile uint32_t *counter = (uint32_t *)COUNTER_ADDR;
    volatile uint32_t *mtime = (uint32_t *)MTIME_ADDR;
    volatile uint32_t *mtimecmp = (uint32_t *)MTIMECMP_ADDR;

    // Increment counter
    (*counter)++;

    // Reset mtimecmp for the next interrupt
    uint32_t current_time = *mtime;
    *mtimecmp = current_time + 1000000; // Trigger after 1M ticks
}

// Initialization function
void init_mtip(void) {
    // Set mtvec to point to the handler
    csr_write(0x305, (uint32_t)&mtip_handler); // mtvec CSR

    // Enable machine timer interrupt (MTIE, bit 7) in mie
    csr_write(0x304, (1 << 7)); // mie CSR

    // Enable global machine-mode interrupts (MIE, bit 3) in mstatus
    uint32_t mstatus = csr_read(0x300); // mstatus CSR
    mstatus |= (1 << 3);
    csr_write(0x300, mstatus);

    // Set mtimecmp to trigger after 1M ticks
    volatile uint32_t *mtimecmp = (uint32_t *)MTIMECMP_ADDR;
    *mtimecmp = 1000000;
}

int main(void) {
    init_mtip();
    while (1) {
        // Main loop; interrupt handler runs asynchronously
    }
    return 0;
}

```
</details>

---

# 14. rv32imac vs rv32imc – What’s the “A”?</strong>

### ❓ Question  
**“Explain the ‘A’ (atomic) extension in rv32imac. What instructions are added and why are they useful?”**

---

### What is the “A” Extension?

- The **“A” stands for Atomic** — it adds **atomic read-modify-write** instructions.
- These instructions allow for **safe, hardware-supported access to shared memory**.
- It is used when multiple cores/threads need to access the same variable without conflicts.

---

###  Instructions Introduced by the "A" Extension

| Instruction   | Meaning                             | Purpose                            |
|---------------|--------------------------------------|-------------------------------------|
| `lr.w`        | Load-Reserved word                   | Starts an atomic load-store block   |
| `sc.w`        | Store-Conditional word               | Stores only if no one else touched |
| `amoadd.w`    | Atomic add                           | Adds a value atomically             |
| `amoswap.w`   | Atomic swap                          | Atomically swaps memory             |
| `amoand.w`    | Atomic AND                           | Useful for bitmask flags            |
| `amoor.w`     | Atomic OR                            | Used in setting flags atomically    |

---

###  Use Cases

- **Operating Systems**: Manage shared memory between tasks/interrupts.
- **Multithreading**: Protect shared counters or variables.
- **Lock-Free Data Structures**: Like queues or stacks, to avoid performance penalties from locking.

---

###  Analogy

Imagine you're writing on a shared whiteboard:

- `lr.w` → You reserve a spot to write.  
- `sc.w` → You try to write, but **only succeed if no one else has written there** since you reserved it.  
- `amoadd.w` → You **add a number** to the current value, and it's done **safely**, even if others are trying at the same time.

---

### Summary

- The “A” extension makes **safe concurrent memory access** possible in RISC-V.
- It is essential for low-level OS and embedded systems programming.
- It **distinguishes `rv32imac` (which supports atomics)** from `rv32imc` (which does not).

---

# Task 15: Atomic Test Program

### Question:

“Provide a two-thread mutex example (pseudo-threads in main) using lr.w/sc.w on RV32.”

---

Method:
- Simulate two threads using functions in main.

- Use lr.w/sc.w instructions in inline assembly for spinlock mutex.

- Demonstrate protection of a shared variable (shared_counter) from race conditions.

---

# Full Code with Explanation:

![image](https://github.com/user-attachments/assets/3a19f37a-e082-4533-9234-539ef2d64fe9)

```c
#include <stdio.h>
#include <stdint.h>

// Simulated mutex variable (0 = unlocked, 1 = locked)
volatile uint32_t mutex = 0;

// Simulated thread IDs for demonstration
#define THREAD_1 1
#define THREAD_2 2

// Function to simulate acquiring the mutex using lr.w and sc.w
int acquire_mutex(uint32_t thread_id) {
    uint32_t temp;
    int success;
    do {
        // Load-Reserved: Load mutex value
        asm volatile ("lr.w %0, (%1)" : "=r"(temp) : "r"(&mutex));
        // Check if mutex is already locked
        if (temp == 1) {
            printf("Thread %d: Mutex already locked, retrying...\n", thread_id);
            continue;
        }
        // Store-Conditional: Try to set mutex to 1 (locked)
        asm volatile ("sc.w %0, %2, (%1)" : "=r"(success) : "r"(&mutex), "r"(1));
    } while (success != 0); // Retry if store-conditional failed
    printf("Thread %d: Mutex acquired\n", thread_id);
    return 1;
}

// Function to release the mutex
void release_mutex(uint32_t thread_id) {
    // Simply set mutex to 0 (unlocked)
    asm volatile ("sw zero, (%0)" : : "r"(&mutex));
    printf("Thread %d: Mutex released\n", thread_id);
}

// Simulate critical section
void critical_section(uint32_t thread_id) {
    printf("Thread %d: Entering critical section\n", thread_id);
    // Simulate some work
    for (volatile int i = 0; i < 100000; i++);
    printf("Thread %d: Exiting critical section\n", thread_id);
}

// Simulate Thread 1 behavior
void thread1_simulation() {
    printf("Thread 1: Starting\n");
    acquire_mutex(THREAD_1);
    critical_section(THREAD_1);
    release_mutex(THREAD_1);
    printf("Thread 1: Finished\n");
}

// Simulate Thread 2 behavior
void thread2_simulation() {
    printf("Thread 2: Starting\n");
    acquire_mutex(THREAD_2);
    critical_section(THREAD_2);
    release_mutex(THREAD_2);
    printf("Thread 2: Finished\n");
}

int main() {
    printf("Main: Starting two-thread mutex simulation\n");
    
    // Simulate two threads running "concurrently"
    thread1_simulation();
    thread2_simulation();
    
    printf("Main: Simulation complete\n");
    return 0;
}
```
---

Expected Output:<br>

```vbnet
Shared counter: 3
```
Each function locks the variable, modifies it safely, and releases the lock.

---

### Concepts
- Spinlock: A busy-wait loop that tries acquiring a lock until it succeeds.

- lr.w / sc.w: RISC-V atomic pair that guarantees safe update without interference.

- Volatile: Prevents compiler from caching or optimizing memory accesses.

- Pseudo-threads: We simulate threads using plain functions to show locking logic.

 --- 

 # 16 Using Newlib printf Without an OS

---

## Question

“How do I retarget `_write` so that `printf` sends bytes to my memory-mapped UART?”

---

## Answer Outline

- Implement `_write(int fd, char* buf, int len)` that loops over bytes to `UART_TX`.
- Re-link with `-nostartfiles` plus custom `syscalls.c`.
- Run in QEMU and verify UART output.

---



<details><summary><strong> linker.ld file:</strong></summary><br>

![unnamed](https://github.com/user-attachments/assets/b4449d2f-4f7a-41ec-8a0e-30900ca92b48)

</details>

# Task 17: Endianness & Struct Packing

**Question:**  
Is RV32 little-endian by default? Show how to verify byte ordering with a union trick in C.

---

<details>
<summary><b>Steps to check endianness in C</b></summary><br>

![image](https://github.com/user-attachments/assets/4f937562-8ff4-4b5e-98c5-839c2b0e2ceb)

---

</details>

---

## Summary: What is RISC-V?

RISC-V is an **open Instruction Set Architecture (ISA)**. It is not a microprocessor, not Verilog, and not a chip.

### Key Clarifications:

- **RISC-V is an ISA**: It defines the set of instructions a processor can execute (e.g., `add`, `load`, `branch`, etc.).
- **It is not a processor**: It is the blueprint or specification, not the hardware implementation.
- **It is not Verilog**: RISC-V processors can be implemented using Verilog, but RISC-V itself is language-agnostic.
- **It is not a chip**: Many chips implement RISC-V, but RISC-V itself is only the architectural specification.

This separation is critical: Just as a programming language is distinct from the compiler that implements it, RISC-V is distinct from the hardware or software built on top of it.

---

## Why We Did These Tasks

The weekly tasks were designed to build our understanding of embedded systems using RISC-V from the ground up. Each step removed layers of abstraction that modern operating systems usually hide from us.

| Task | Concept |
|------|---------|
| **Toolchain Setup** | We learned how to compile C code into RISC-V ELF binaries using a cross-compiler. |
| **Disassembly & `info reg`** | We analyzed how our C code translates into assembly and how CPU registers change at runtime. |
| **Memory-Mapped I/O** | We saw how to control hardware directly by writing to specific memory addresses. |
| **Linker Scripts** | We controlled where code and data sections are placed in memory, crucial for bare-metal systems. |
| **Start-up Code (`crt0.S`)** | We understood what happens before `main()` runs: setting the stack, zeroing `.bss`, and transferring control. |
| **Interrupts** | We configured and handled timer interrupts, learning how CPUs respond to asynchronous events. |
| **Atomic Operations & Endianness** | We explored concurrency control using `lr/sc` and verified byte ordering in memory. |
| **Retargeting `printf` via UART** | We enabled output without an OS by writing our own `_write` function to communicate with a memory-mapped UART. |

---

## Final Reflection

By completing these tasks, we developed a full-stack understanding of what goes into bringing up and controlling a bare-metal RISC-V system. We wrote programs that could boot without an OS, interact directly with memory and hardware, handle interrupts, and even print messages through a custom UART.

Through this, we didn’t just learn about RISC-V as an architecture—we experienced what it means to build and control an entire system based on it. This foundation is critical not only for embedded systems development but also for understanding operating systems, device drivers, and processor design.




