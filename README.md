# **Project Overview**

This repository is focused on **enhancing and extending** the educational operating system **xv6**.  
**xv6** (or **XV6**) is a reimplementation of Dennis Ritchie and Ken Thompson's **Unix Version 6 (v6)**, developed at MIT for teaching operating system concepts. It provides a simple yet realistic implementation of a Unix-like system that can run on modern hardware while maintaining the essential architectural elements of Unix.

These projects systematically modify and extend xv6's functionality across multiple core subsystems. The work begins with **basic system call implementations** and **command modifications**, progresses through **process scheduling algorithms**, explores **virtual memory management** with lazy allocation techniques, and culminates in extending the **file system** with support for larger files and symbolic links.

### **Goals**

1. **Understand the internals of a Unix-like operating system**  
   Gain deep familiarity with the architecture and implementation details of process management, memory allocation, device management, file systems, and the relationships between these subsystems.

2. **Hands-on experience with system call implementation**  
   Learn the complete lifecycle of system calls from user space to kernel and back, including argument passing, privilege transitions, and error handling.

3. **Explore process scheduling algorithms**  
   Implement and compare multiple scheduling algorithms (default FIFO, SJF, lottery, priority) and understand their performance characteristics and implementation complexity.

4. **Implement virtual memory concepts**  
   Understand page tables, address translation, trap handling, and advanced memory management techniques like on-demand (lazy) allocation with locality awareness.

5. **Extend file system capabilities**  
   Modify the existing file system to support larger files (multi-level indirection), implement symbolic links, and add file seek capabilities (e.g., `lseek`).

6. **Develop practical OS development skills**  
   Gain experience with low-level debugging techniques, system-level programming, and complex interactions between hardware and software.

7. **Learn to debug and test complex system-level code**  
   Master techniques for verifying correctness across different execution paths.

---

# **Project Structure**

1. [**Basic System Calls and Commands**](#project-1-basic-system-calls-and-commands)  
2. [**Syscalls and Schedulers**](#syscalls-and-schedulers)  
3. [**Virtual Memory**](#virtual-memory)  
4. [**File Systems**](#file-systems)  

Each project builds upon previous ones, creating a progressively more capable and sophisticated operating system.

---

## ** Basic System Calls and Commands**

Focuses on **fundamental operations** in xv6, introducing modifications to existing commands and implementing new system calls.

### **Data Structures**

- **Process control block** (`struct proc` in `proc.h`): maintains process state, process ID, kernel stack.  
- **File metadata structures** (`struct stat` in `stat.h`): used by the `ls` command to distinguish between files and directories.  
- **Syscall function pointer array** (`syscalls[]` in `syscall.c`): maps system call numbers to their implementations.  
- **System call argument handling** structures for parameter passing from user space to kernel.

### **Algorithms**

- **Command-line argument parsing** for the `sleep` command (using `argv` array).  
- **File/directory traversal** for the `ls` command (using `struct dirent`).  
- **System call registration/dispatch** (transition from user space to kernel space).  
- **Process sleep/wakeup** mechanism for `sleep`.

### **Challenges & Fixes**

- Modifying multiple files (`syscall.h`, `syscall.c`, `user.h`, `usys.S`) for new system calls.  
- Handling hidden files (dotfiles) in `ls` while preserving original functionality.  
- Implementing directory indicators (`/`) for `ls`.  
- Ensuring proper error handling for `sleep` with no arguments.  
- Updating the **Makefile** to include new user programs.

---

## **Syscalls and Schedulers**

Extends **syscall functionality** and implements **advanced scheduling algorithms** (SJF, lottery, priority).

### **Data Structures**

- **Extended process control block**: tracks scheduling info (predicted job length, ticket count, priority level, runtime stats).  
- **Scheduling queues**: separate lists of processes at different priority levels.  
- **Scheduler-specific context structures**: maintain algorithm state across context switches.

### **Algorithms**

1. **Shortest Job First (SJF)**
   - Job length prediction (exponential average or random).  
   - Selects the runnable process with the shortest predicted running time.

2. **Lottery Scheduling** (optional)
   - Assigns tickets to processes.  
   - Uses pseudo-random selection to pick the next process.  
   - CPU time is proportional to ticket counts.

3. **Multi-level Priority Scheduling** (optional)
   - Multiple priority queues.  
   - Priority boosting to prevent starvation.  
   - Round-robin within each priority level.

4. **`uniq` command**  
   - Adjacent line deduplication (`-c`, `-u`, `-w` flags).

5. **`find` command**  
   - Recursive search with filename matching, file type filtering, inode comparisons, custom formatting.

### **Challenges & Fixes**

- **Timer interrupt handling** changes for new schedulers.  
- Tracking **execution time** across scheduling decisions.  
- **Compile-time scheduler selection** via Makefile.  
- Managing **scheduler state** during process creation/termination.  
- Handling edge cases in `uniq` and `find`.

---

## **Virtual Memory**

Explores **memory management** in xv6, shifting from eager to **lazy allocation**.

### **Data Structures**

- **Page tables** (multi-level structures mapping virtual to physical addresses).  
- **`proc->sz`**: process memory size tracking.  
- **Page fault info** from trap frames.  
- **Allocation metadata** for pages.

### **Algorithms**

1. **Trap Handling for Page Faults**  
   - Distinguishes invalid memory accesses from other traps.  
   - Allocates physical memory **on-demand**.

2. **Modified `sbrk()`**  
   - Adjusts `proc->sz` without immediately allocating memory.

3. **Lazy Page Allocation**  
   - Catches page faults.  
   - Allocates a new physical page.  
   - Updates page tables.  
   - Resumes execution transparently.

4. **Locality-aware Allocation** (optional)  
   - Allocates faulting page plus adjacent pages.  
   - Reduces fault frequency.

### **Challenges & Fixes**

- Understanding the **relationship** between `sbrk()`, `growproc()`, and physical allocation.  
- Correctly **inserting page fault handling** in `trap.c`.  
- Differentiating **legitimate page faults** from illegal accesses.  
- Handling **compile-time selection** of allocation strategies.  
- Ensuring **memory protection** in page tables.

---

## **File Systems**

Extends xv6 file system with **larger file support** and **symbolic links**.

### **Data Structures**

- **Inode structure** modifications (`struct dinode` in `fs.h`) for double-indirect blocks.  
- **File descriptor table** entries track offsets for `lseek`.  
- **Symbolic link metadata** to store target paths.  
- **Block allocation tables** and **buffer cache** for disk blocks.

### **Algorithms**

1. **File Seeking & Offset Management**  
   - `lseek(fd, offset)`: Reposition file offsets.  
   - Handles out-of-bound seeks (hole filling).

2. **Block Mapping (`bmap()`)**  
   - **Direct blocks** (first 10-11).  
   - **Single-indirect** (next 128).  
   - **Double-indirect** (up to 128×128).

3. **Symbolic Links**  
   - `symlink(target, path)`.  
   - Recursive link resolution with cycle detection.  
   - Respecting `O_NOFOLLOW` flag.

4. **Block Allocation/Freeing**  
   - For **large files** with multi-level indirection.  
   - **itrunc()** releases all blocks properly.

### **Challenges & Fixes**

- Maintaining **backward compatibility** with the same inode size.  
- Balancing **direct/indirect/double-indirect** blocks for performance.  
- Handling **symbolic link cycles** and recursion depth limits.  
- Zero-filling for sparse files created by `lseek`.  

---

# **Key Architectural Decisions**

## **System Call Implementation**

- **Integration** via `syscall.h`, `syscall.c`, `user.h`, `usys.S`.  
- **Error handling** standardized (negative return values).  
- **User-space** vs. **kernel-space** separation with well-defined interfaces.

## **Scheduler Design**

- **Modular** scheduler implementation with a clear separation between policy and mechanism.  
- **Compile-time** selection via Makefile flags.  
- **Backward compatibility** with existing process management.  
- **Process statistics tracking** (e.g., `ticks_running`).  
- **Priority inversion** prevention with boosting.

## **Memory Management**

- **Lazy allocation** that **transparently** catches page faults.  
- **Locality-aware** allocation for performance.  
- **Trap handling** that differentiates legitimate page faults from illegal accesses.  
- **Compile-time** strategy selection (e.g., `ALLOCATOR=LAZY` or `ALLOCATOR=LOCALITY`).

## **File System Extensions**

- **Backward-compatible inode extensions** (double-indirect blocks).  
- **Symbolic links** stored in inode data blocks.  
- **`lseek`** integrated with `read/write` ops.  
- **Sparse file** support with zero-filling.  
- **Multi-level block mapping** (`bmap`) for large files.

---

# **Functions and Algorithms**

### **Examples**

- **`hello()`**: Prints a kernel message (`"Hello from the Kernel!"`).  
- **`ls`** (modified): Hides dotfiles unless `-a` flag is present; appends `/` for directories.  
- **`sleep n`**: Pauses execution for `n` ticks.  
- **`ticks_running(pid)`**: Returns process running time in ticks or `-1` if nonexistent.  
- **`uniq`**: Removes adjacent duplicate lines, supports `-c`, `-u`, `-w`.  
- **`find`**: Recursive file search with flags like `-type`, `-inum`, `-printi`.

### **Schedulers**

- **SJF**: Predicted job length, picks shortest job next.  
- **Lottery**: Assign tickets, random draw.  
- **Priority**: Multiple priority levels, round-robin within each level.

### **Lazy Allocation**

- **Modified `sbrk(n)`**: Updates process size, no immediate allocation.  
- **Page fault handler**: Allocates physical memory on first access.  
- **Locality**: Allocates the faulting page plus neighbors.

### **File System**

- **`lseek(fd, offset)`**: Reposition file offset.  
- **`symlink(target, path)`**: Create symbolic link.  
- **`open`** (enhanced): Follows links unless `O_NOFOLLOW`.  
- **`bmap()`**: Handles direct, single-indirect, and double-indirect blocks.  
- **`itrunc()`**: Frees all blocks (direct, indirect).

---

# **Building and Running**

```bash
# Clean previous builds
make clean

# Build xv6 with default configuration
make

# Run xv6 in QEMU with GUI
make qemu

# Run xv6 in QEMU (console-only)
make qemu-nox

# Scheduler selection
make qemu SCHEDULER=DEFAULT
make qemu SCHEDULER=SJF
make qemu SCHEDULER=LOTTERY
make qemu SCHEDULER=PRIORITY

# Memory allocator selection
make qemu ALLOCATOR=LAZY
make qemu ALLOCATOR=LOCALITY
```

---

# **Specific Commands**

### ****

```bash
hello            # Outputs "Hello Xv6!" and "Hello from the Kernel!"
ls               # Lists directory contents
ls -a            # Includes hidden files
sleep 10         # Sleeps for 10 ticks
sleep 100        # Sleeps for 100 ticks (about 10 seconds)
```

### **Syscalls and Schedulers**

```bash
uniq filename.txt            # Filters adjacent repeated lines
uniq -c filename.txt         # Counts occurrences
uniq -u filename.txt         # Shows only unique lines
uniq -w 5 filename.txt       # Compares first 5 characters
cat file.txt | uniq          # Reads from stdin

find / -name file.txt        # Searches for file.txt from root
find . -name "*.c"           # Finds .c files in current directory
find /etc -name passwd -type f
find /home -name Documents -type d
find / -name file.txt -printi

ticks_running_test           # Display running time of processes
simple_scheduler_test        # Tests SJF behavior
advanced_scheduler_test      # Tests lottery or priority
set_lottery_tickets 50       # Assign 50 tickets to current process
get_lottery_tickets 2        # Get ticket count for PID 2
set_sched_priority 2         # Set priority to level 2
get_sched_priority 3         # Get priority for PID 3
```

### **Virtual Memory & File Systems**

```bash
# Virtual memory tests
vm_test
vm_locality_test

# File system commands
lseek_test
symlink_test
largefiles_test

# Symlink example
symlink /etc/passwd passwd_link
cat passwd_link
open passwd_link O_NOFOLLOW
```

---

# **Future Work**

### **Potential Improvements**

1. **Scheduler Enhancements**
   - Dynamic priority adjustments (aging mechanism).  
   - Completely Fair Scheduler (CFS) approach.  
   - Real-time constraints (deadline scheduling).  
   - Scheduler activations for user-level threading.

2. **Memory Management**
   - Swapping to disk for better memory utilization.  
   - Shared memory regions for IPC.  
   - Demand paging from executables.  
   - Copy-on-write `fork()`.  
   - Huge pages for TLB optimization.

3. **File System**
   - Journaling for crash resilience.  
   - File system compression.  
   - Access Control Lists (ACLs).  
   - More sophisticated block allocation strategies.  
   - File system quotas.  
   - Extended attributes.

### **Known Limitations**

- **Schedulers**: No real-time constraints, no CPU affinity, limited performance metrics, possible starvation in SJF.  
- **Memory**: No swapping, limited protection, no shared memory, local allocations may waste space.  
- **File System**: Max file size ~16MB, no journaling, limited directory performance, no atomic rename.

---

## **Conclusion**

These projects provide a solid foundation for **understanding operating system concepts** by directly modifying xv6. Each module builds upon the last, culminating in a more **robust and feature-rich system**. However, there remain **ample opportunities** for further enhancements and advanced feature implementations, offering a rich learning platform for aspiring systems programmers.

---

**Thank you for exploring this xv6-based Operating Systems project!** If you have any questions or suggestions, feel free to open an issue or submit a pull request.
