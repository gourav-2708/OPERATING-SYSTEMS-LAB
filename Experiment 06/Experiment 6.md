# Experiment 6

## Dynamic Memory Allocation and Virtual Memory in C

### Aim

To understand how a C program requests memory, causes pages to become resident by writing to them, holds the allocation for observation, and releases resources when it finishes.

### Theory

#### 1. Process address spaces

Each process operates within a virtual address space. The operating system and memory-management hardware translate virtual addresses into physical memory locations and enforce access permissions. Virtual addresses used by two processes can look alike while referring to different physical pages.

Dynamic allocation requests storage during program execution. The C library function `malloc(size)` returns a pointer to an allocated object on success and `NULL` on failure. The returned memory is initially uninitialized. The allocator may satisfy a request from an existing arena or obtain mappings from the operating system.

#### 2. Interpreting the size argument

The lab program accepts one size argument. In `main(int argc, char *argv[])`, `argc` includes the program name, so one supplied size gives `argc == 2`. `argv[1]` holds text that must be converted before arithmetic can be performed.

The requested byte count is calculated as follows.

```text
bytes = requested_size × 1024 × 1024
```

This unit is MiB, with 1 MiB equal to 1,048,576 bytes. A request for 500 MiB represents 524,288,000 bytes. Store sizes and allocation indices in `size_t`, validate a positive numeric argument, and check that multiplication fits in the type. A checked conversion such as `strtoull`, together with range checking, gives more reliable error reporting than `atoi`.

A `char *` makes the allocation convenient to inspect byte by byte. Pointer indexing advances by the size of the pointed-to type, and `sizeof(char)` is one C byte.

#### 3. Demand paging

Reserving an allocation and accessing every page are distinct events. Linux can postpone allocating physical backing for some anonymous pages until they are accessed. Writing to a previously untouched page can trigger a page fault, after which the kernel supplies appropriate backing and execution resumes. A handled page fault is part of normal memory management.

Linux can also allow allocations under an overcommit policy. Consequently, a successful allocation does not establish that every requested byte is already resident or that later accesses will always succeed under memory pressure. [Linux malloc manual](https://man7.org/linux/man-pages/man3/malloc.3.html)

#### 4. Touching the allocation

The manual writes one byte at successive 4096-byte intervals. On a system with 4096-byte pages, these writes touch the allocation at page-sized intervals and cause its backing to be used. The actual page size is platform-dependent and can be obtained using `sysconf(_SC_PAGESIZE)` or `getconf PAGESIZE`.

The number of virtual pages intersecting an allocation also depends on its starting alignment. A complete page-touching implementation should include the final byte so that a trailing partial page is touched. Optimizations and allocator behavior must be considered when designing a measurement program, since the accesses need to remain observable in the compiled program.

Touching the pages typically increases the process's resident memory. The operating system can subsequently reclaim or swap eligible pages under pressure, so residency describes a condition at a particular moment.

#### 5. Observing memory use

The program remains alive using a loop containing `sleep(1)`. Sleeping keeps its allocation available for inspection while allowing other runnable tasks to use the processor. In another terminal, `free -h` displays system memory information and `watch -n 1 free -h` refreshes it approximately once per second.

`free` reports system-wide counters. Its values also reflect other processes, caches, and background activity. The `available` estimate is useful when considering how much memory new applications may use. A process-specific RSS measurement can complement the system totals. Record a baseline, a reading after page writes, and a reading after termination, while expecting some system-wide variation.

#### 6. Releasing memory and handling termination

`free(memory)` returns the object to the allocator. The allocator may retain the underlying pages for later allocations, so a `free` call does not always cause an immediate equal drop in system memory use. After the object is freed, the program must stop accessing it.

An infinite loop prevents cleanup placed after that loop from being reached normally. Ctrl+C normally sends SIGINT to the foreground process group. With the default termination action, the kernel reclaims the process's address space when the process exits.

For explicit application cleanup, a signal handler can set a `volatile sig_atomic_t` flag. The main loop observes the flag, leaves the loop, calls `free`, and exits. Functions such as `printf` and `free` should be called from ordinary program flow because they are not async-signal-safe. This pattern makes interruption and resource cleanup easier to reason about.

### Programming code

These are the two complete C examples from the manual, with page breaks removed. Save them separately on Linux. Both retain the source's `atoi` conversion and fixed 4096-byte stride; use a modest positive integer for this lab demonstration.

#### Basic allocation program — `memory.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
int main(int argc, char *argv[])
{
    int mb;
    if (argc != 2)
    {
        printf("Usage: %s <memory_in_MB>\n", argv[0]);
        return 1;
    }
    mb = atoi(argv[1]);

    char *memory = malloc(mb * 1024 * 1024);
    if (memory == NULL)
    {
        printf("Memory allocation failed\n");
        return 1;
    }
    for (int i = 0; i < mb * 1024 * 1024; i += 4096)
    {
        memory[i] = 1;
    }
    printf("%d MB memory allocated.\n", mb);
    printf("Program is running... Press Ctrl+C to exit.\n");
    while (1)
    {
        sleep(1);
    }
    return 0;
}
```

Compile and start a small allocation:

```bash
gcc -Wall -Wextra memory.c -o memory
./memory 16
```

The argument `16` requests 16 MiB, despite the source's “MB” label. The manual demonstrates `500`; a smaller allocation is shown here to make the same observation with less memory. In a second terminal:

```bash
free -h
watch -n 1 free -h
```

Press **Ctrl+C** in the program's terminal when finished. The basic version has no explicit `free()` path; the operating system reclaims its mappings on termination.

#### Manual's signal-handling extension — `memory_signal.c`

The following is a faithful transcription of the manual's extension, included for comparison. Its handler calls `free`, `printf`, and `exit`, which are not async-signal-safe. For a corrected implementation, use the flag-based cleanup approach described in the theory and perform allocation cleanup outside the handler.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
char *memory;
void handle_sigint(int sig)
{
    free(memory);
    printf("\nMemory released.\n");
    exit(0);
}
int main(int argc, char *argv[])
{
    int mb;
    if (argc != 2)
    {
        printf("Usage: %s <memory_in_MB>\n", argv[0]);
        return 1;
    }
    signal(SIGINT, handle_sigint);
    mb = atoi(argv[1]);
    size_t size = (size_t)mb * 1024 * 1024;
    memory = malloc(size);
    if (memory == NULL)
    {
        printf("Memory allocation failed.\n");
        return 1;
    }

    for (int i = 0; i < mb * 1024 * 1024; i += 4096)
    {
        memory[i] = 1;
    }
    printf("Memory allocated successfully.\n");
    printf("Press Ctrl+C to release memory and exit.\n");
    while (1)
    {
        sleep(1);
    }
    return 0;
}
```

The original examples also need robust numeric conversion, overflow checks, and a system-derived page size before accepting arbitrary allocation requests. Their signed `int` multiplication and loop bounds can overflow for large inputs. These limitations belong to the source examples and are not resolved by removing PDF formatting.

**Code source:** supplied Experiment 6 manual, pages 1–2 and 11–12. The example run size has been reduced to 16 MiB.

### Expected observations

An untouched allocation can produce a smaller immediate RSS increase than its requested size. Writing through the allocation generally increases residency, and terminating the process releases its address space. Exact changes depend on page size, allocator behavior, caches, memory pressure, and other system activity.

### Conclusion

Dynamic allocation, physical residency, and release are separate stages. The experiment connects C pointers and allocation functions with virtual-memory translation, demand paging, and operating-system accounting.

### Reference

Supplied manual: *Experiment 6 Memory Allocation & Virtual Memory in C Manual.pdf*, pages 1-13.
