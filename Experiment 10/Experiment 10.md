# Experiment 10

## Parent-Child Synchronization Using fork() and wait()

### Aim

To explain the existing Experiment 10 C program, which creates one child, lets it complete its work, and makes the parent report that child's PID after collecting its termination.

### Theory

#### 1. The synchronization problem

Parent and child processes can run independently after `fork()`. Their scheduling order is determined by the operating system. If the parent needs the child's work to be complete before continuing, it requires a synchronization operation.

The Experiment 10 program uses `wait(NULL)` for this purpose. Its child prints its PID, pauses with `sleep(2)`, prints a termination message, and calls `exit(0)`. The parent prints a waiting message, calls `wait(NULL)`, and reports the returned PID. The two-second pause makes the waiting stage easier to observe.

#### 2. Required interfaces

| Header | Relevant interface |
| --- | --- |
| `<stdio.h>` | `printf()` and diagnostic output |
| `<stdlib.h>` | `exit()` and termination constants |
| `<unistd.h>` | `fork()`, `getpid()`, and `sleep()` |
| `<sys/wait.h>` | `wait()`, `waitpid()`, and status macros |

These process interfaces belong to the Unix/POSIX environment used by the Linux lab. The program is compiled with GCC and executed as a Linux process.

#### 3. Selecting the correct branch

The variable receiving `fork()` has type `pid_t`. A negative return indicates failure and must lead to error reporting and termination of the original process. A zero return selects the child branch. A positive return selects the parent branch and contains the child's PID.

The child's own PID is obtained with `getpid()`. A successful creation produces different parent and child PIDs, while the parent's positive `fork()` result identifies that particular child.

#### 4. Blocking rather than repeatedly checking

When no eligible child has yet terminated, a blocking wait allows the caller to sleep until an appropriate event occurs. The parent does not need a loop that repeatedly checks a variable, and it does not use a full processor simply to await completion.

`sleep(2)` pauses the child for a time interval. `wait(NULL)` synchronizes the parent with child termination. Changing the child's workload or delay changes how long the parent may wait, while the completion relationship remains the purpose of `wait()`.

#### 5. Completion and status collection

When the child calls `exit(0)`, its normal exit processing runs and the kernel records its completion. The successful `wait()` both identifies the finished child and collects its retained termination record. This prevents an uncollected zombie record from persisting under the normal status-handling configuration used here.

For this single-child program, the PID returned by a successful wait should equal the positive PID obtained by the parent from `fork()`. If the child finishes before the parent reaches `wait()`, the completion is retained for collection and the wait can return immediately.

#### 6. PID and exit status

The wait function's return value is a PID. To obtain the encoded termination status as well, the caller can pass the address of an integer, as in `wait(&status)`. Check `WIFEXITED(status)` before reading `WEXITSTATUS(status)` for normal completion. For signal termination, use `WIFSIGNALED(status)` and then `WTERMSIG(status)`.

The `NULL` argument in the current lab program discards that status value. It still allows the parent to collect the child and obtain its PID. When a program has several children and needs one specific child, `waitpid(child_pid, &status, 0)` selects it explicitly. [Linux wait manual](https://man7.org/linux/man-pages/man2/waitpid.2.html)

#### 7. Ordering and error handling

The child and parent messages before the wait can appear in either order. The parent's message after a successful wait indicates that the child has already terminated. Output buffering can affect when text becomes visible, especially when output is redirected, so visible line timing should be interpreted with that in mind.

A `-1` return from `wait()` indicates an error. `EINTR` can occur if a caught signal interrupts the wait, in which case a suitable implementation retries. `ECHILD` indicates that there is no eligible child to collect. Reporting a returned PID without checking success can therefore produce a misleading completion message.

### Conceptual execution flow

```text
Original process
    |
    +-- fork failure --> report error and finish
    |
    +-- child --> print own PID --> sleep --> exit
    |
    +-- parent --> wait for child --> collect PID --> report completion
```

### Programming code

This C program is transcribed from the screenshot in the existing [Experiment10.pdf](./Experiment10.pdf), page 1. Save it as `process_creation_wait.c` and compile on Linux.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main()
{
    pid_t p = fork();

    if (p < 0)
    {
        printf("Fork failed!\n");
        return 1;
    }

    if (p == 0)
    {
        printf("Child PID: %d\n", getpid());
        sleep(2);
        printf("Child terminated.\n");
        exit(0);
    }
    else
    {
        printf("Parent waiting for child...\n");
        pid_t pid = wait(NULL);
        printf("Child PID %d terminated.\n", pid);
    }

    return 0;
}
```

#### Compile and run

```bash
gcc -Wall -Wextra process_creation_wait.c -o exp10
./exp10
```

The program checks for `fork()` failure, as shown in the PDF. It does not check the return value of `wait()` before printing the termination message; the error-handling discussion in the theory explains that remaining limitation. The child's “terminated” message is printed just before `exit(0)`, while the parent's message follows successful collection of the child's termination.

**Code source:** existing repository file `Experiment10.pdf`, page 1. Formatting has been normalized for Markdown.

### Expected observations

The child reports an actual runtime PID. The parent may appear paused while the child sleeps and works. After successful collection, the parent reports the same child PID. The total runtime includes the child delay and normal scheduling overhead, so it need not be exactly two seconds.

### Conclusion

The experiment demonstrates how a parent coordinates with a child's lifecycle. Waiting provides completion synchronization and collects the child's termination record, while the returned PID identifies the child that finished.

### Source basis

This theory follows the program already stored in [Experiment10.pdf](./Experiment10.pdf) and its accompanying folder description. No separate Experiment 10 manual was included in the supplied attachments. The attached Experiment 9 manual covers the same family of process functions, so the two experiments share foundational concepts.
