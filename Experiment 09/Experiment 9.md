# Experiment 9

## Process Identification, Creation, and Termination in C

### Aim

To understand process identifiers, create a child process with `fork()`, distinguish the parent and child execution paths, and use `wait()` to observe child termination.

### Theory

#### 1. Programs and processes

A program is a stored set of instructions. A process is a running instance with an execution context, virtual memory, open resources, and an identifier. Several processes can execute the same program simultaneously.

Linux assigns each live process a PID within its PID namespace. PIDs can be reused after processes finish, so an example PID is meaningful only for the associated run. The parent process is identified by a PPID.

`getpid()` returns the calling process's PID, and `getppid()` returns its current parent's PID. These functions are declared in `<unistd.h>`. The type `pid_t` is intended for process identifiers. A portable teaching example can print one by converting it to `long` and using `%ld`.

#### 2. Creating a child

`fork()` creates a child process from the calling process. Both continue from the point immediately after the call, and the return value tells each process which branch to execute.

| Return value | Meaning |
| --- | --- |
| Negative, normally `-1` | Creation failed and no child was created |
| `0` | Execution is in the child |
| Positive PID | Execution is in the parent, and the value identifies the child |

The failure branch must be checked separately before treating a nonzero return as successful parent execution. Resource limits or insufficient resources can make creation fail.

#### 3. Separate execution contexts

The parent and child have separate private address spaces. At creation their private memory contents correspond, but later ordinary writes in one do not update the other's private variables. Linux normally implements this efficiently using copy-on-write, postponing physical copying until a shared private page needs to be written. [Linux fork manual](https://man7.org/linux/man-pages/man2/fork.2.html)

The child inherits copies of open file descriptors. Corresponding descriptors can refer to the same underlying open file description, which can share an offset. This differs from ordinary private-variable storage. Explicit shared-memory mechanisms are needed when the program intends to communicate through memory.

#### 4. Concurrent execution and output order

After a successful fork, scheduling determines when each runnable process executes. The parent may print first in one run and the child may print first in another. On a multicore system they may execute concurrently on different logical CPUs.

A short delay can make the behavior easier to observe, but a delay alone does not establish a dependable ordering relationship. Synchronization is needed when the parent must continue only after the child completes.

#### 5. Normal termination

Returning from `main` or calling `exit(0)` performs normal program termination, with zero conventionally indicating success. The C runtime performs its normal exit processing, including flushing standard I/O streams. The kernel releases the exiting process's address space and open-descriptor references.

Under the ordinary child-status handling used in this lab, the kernel retains a small termination record until the parent collects it. An exited child with an uncollected record is called a zombie. This state preserves information about the completed process rather than continuing its computation.

#### 6. Waiting and collecting the child

The call `wait(NULL)` collects termination information for one eligible child and returns that child's PID. If a child has already terminated, it can return immediately. Otherwise, it blocks the caller until a child terminates or an error or interruption occurs. `NULL` means that the caller does not request the encoded termination status. [Linux wait manual](https://man7.org/linux/man-pages/man2/waitpid.2.html)

With exactly one successfully created child, the positive PID returned by `fork()` in the parent should match the successful return from `wait()`. The child's `getpid()` should report the same identifier. Check for a `-1` return before reporting successful completion.

### Execution sequence

1. The original process obtains or displays its PID.
2. A successful `fork()` creates a child and produces two execution paths.
3. The child displays its own PID, performs its work, and exits.
4. The parent waits and collects the child's termination record.
5. The parent displays the returned child PID and finishes.

### Programming code

The following three C programs follow the manual's progression from retrieving a PID to creating and waiting for a child. Save each listing separately and compile on Linux, where `fork()` and `wait()` are available.

#### Step 1: Display the current PID — `step1_pid.c`

```c
#include <stdio.h>
#include <unistd.h>
int main() {
    // getpid() returns the PID of the currently running process
    printf("I am a simple process. My PID is: %d\n", getpid());
    return 0;
}
```

```bash
gcc -Wall -Wextra step1_pid.c -o step1_pid
./step1_pid
```

#### Step 2: Create a child — `step2_fork.c`

```c
#include <stdio.h>

#include <unistd.h>
int main() {
    pid_t p = fork();
    // From this point onward, both processes execute the same code
    if (p == 0) {
        // Only the child executes this branch
        printf("I am the CHILD process! My own PID: %d\n", getpid());
    } else {
        // Only the parent executes this branch
        printf("I am the PARENT process! fork() gave me child's PID: %d\n", p);
    }
    return 0;
}
```

```bash
gcc -Wall -Wextra step2_fork.c -o step2_fork
./step2_fork
```

#### Step 3: Wait for child termination — `process_creation_wait.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
int main() {
    pid_t p = fork();
    if (p == 0) {
        // --- CHILD BRANCH ---

        printf("[Child] Work started. My PID: %d\n", getpid());
        sleep(2); // Pause so students can clearly observe the parent waiting
        printf("[Child] Work finished! Exiting now.\n");
        exit(0);
    }
    else {
        // --- PARENT BRANCH ---
        printf("[Parent] Child started. Now waiting for it to finish...\n");
        // wait(NULL) halts the parent and returns the terminated child's PID
        pid_t terminated_pid = wait(NULL);
        printf("[Parent] Child with PID %d has terminated!\n", terminated_pid);
    }
    return 0;
}
```

```bash
gcc -Wall -Wextra process_creation_wait.c -o exp9
./exp9
```

These are the manual's original examples. Steps 2 and 3 assume `fork()` succeeds: their `else` branch also receives a failure return of `-1`, so a robust implementation must test `p < 0` separately. Step 3 also assumes `wait(NULL)` succeeds. Apply the failure checks explained in the theory when extending these examples. PIDs shown by a run will differ from the manual's sample output.

**Code source:** supplied Experiment 9 manual, pages 1–3. Page breaks have been removed and indentation retained.

### Expected observations

The parent and child have different own PIDs. The value returned to the parent by `fork()` matches the child's `getpid()`. Early messages may appear in different orders, but the parent's successful completion message after `wait()` occurs after the child has terminated. Record actual identifiers rather than reusing those shown in the manual.

### Conclusion

The experiment connects process identity, creation, independent execution, termination, and collection of child status. `fork()` produces the two execution paths, while `wait()` allows the parent to observe completion.

### Reference

Supplied manual: *Experiment 9 Process Creation & Termination Using C Manual.pdf*, pages 1-4.
