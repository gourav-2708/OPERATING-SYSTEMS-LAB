# Experiment 7

## CPU-Bound and I/O-Bound Programs Using top

### Aim

To compare the CPU utilization, process states, and accumulated CPU time of a computation-heavy C program and a C program that repeatedly performs file operations and sleeps.

### Theory

#### 1. CPU bursts and waiting

A process alternates between executing instructions and waiting for events. A CPU burst is a period of processor activity. Waiting can occur for input, storage operations, timers, or synchronization. The scheduler chooses runnable tasks and shares available processor time among them.

A CPU-bound workload spends much of its time performing computation and is limited mainly by processor throughput. An I/O-bound workload spends substantial time waiting for input or output. Workload behavior is observed over time, since a process may move between computation and waiting during execution.

#### 2. The computation-heavy program

The manual's CPU-bound program repeatedly increments a counter in an infinite loop and occasionally prints its value. Most iterations perform arithmetic and a remainder test, keeping the process runnable for long periods. Printing only after many iterations limits the relative amount of terminal I/O.

The scheduler may run this program continuously when a logical CPU is available, or share the CPU with competing tasks. Its accumulated CPU time should grow quickly. For a sustained demonstration, the counter type and loop design must avoid signed integer overflow, and the compiler must retain the intended computation. A finite workload or an unsigned counter can make the arithmetic behavior well-defined.

#### 3. The file-operation program

The second program repeatedly opens `dummy.txt`, writes a message, closes the stream, and calls `sleep(1)`. Opening with mode `"w"` creates the file if necessary and truncates existing contents. The program must check the result of `fopen` before using the stream, and writing or closing can also report errors.

`fprintf` sends data through a C stream. `fclose` flushes buffered stream output and closes the file, but storage caching can delay a physical-device write. Small repeated writes may be handled largely through memory caches.

The deliberate one-second sleep explains much of this example's low CPU consumption. It is therefore a useful illustration of computation mixed with waiting, although the observed delay is largely a timer wait. A true storage-limited workload would require measurements that establish storage as the limiting resource.

#### 4. Concurrent execution

Appending `&` starts each executable as a background job. Both processes then remain available for observation while the shell accepts further commands. `$!` captures the PID of the most recently started background job in the shell. Process tools can also identify the running commands.

```bash
./io_bound &
io_pid=$!
./cpu_bound &
cpu_pid=$!
top -p "$io_pid,$cpu_pid"
```

These commands assume the two executables have already been compiled. Their PIDs should be recorded for the current run because process identifiers change between executions.

#### 5. Reading top

`top` refreshes its process display periodically. Its output includes processor usage, memory use, process state, and accumulated CPU time.

| Field | Meaning for the comparison |
| --- | --- |
| `PID` | Identifies the monitored process |
| `%CPU` | Processor use during the measurement interval |
| `S` | Current process-state indicator |
| `TIME+` | Accumulated CPU time used by the process |
| `COMMAND` | Program or command name |

State `R` indicates a process running or ready to run. State `S` indicates interruptible sleep, which is commonly seen during `sleep(1)`. State `D` indicates uninterruptible sleep, often associated with certain kernel or I/O waits. A sleeping process can still show nonzero `%CPU` because the percentage summarizes activity over an interval.

With common `top` settings, a single busy thread may approach 100% of one logical CPU. Display mode, CPU allocation, competition, and virtual-machine limits affect this value. CPU percentage should therefore be interpreted together with elapsed time and `TIME+`.

#### 6. Comparing the observations

Take several samples at the same refresh interval. The counter program should normally show higher processor use and a faster increase in `TIME+`. The file-operation program should often appear sleeping, with CPU time increasing more slowly. Exact percentages and states are measurements to record during the actual lab run.

After collecting observations, terminate the specific processes started for the experiment so their loops finish. Exiting `top` closes the monitor and does not itself stop the monitored programs.

### Programming code

Save the two C programs below in a practice directory on Linux. They are transcribed from the manual. The file-I/O program creates or overwrites `dummy.txt` in its working directory.

#### Program 1: File I/O with a sleep — `io_bound.c`

```c
#include <stdio.h>
#include <unistd.h>
int main()
{
    FILE *fp;
    printf("Starting I/O task...\n");
    while (1)
    {
        fp = fopen("dummy.txt", "w");
        if (fp != NULL)
        {
            fprintf(fp, "Writing disk data\n");
            fclose(fp);
            printf("File operation done\n");
        }
        sleep(1);
    }
    return 0;

}
```

#### Program 2: Continuous computation — `cpu_bound.c`

```c
#include <stdio.h>
int main()
{
    long long counter = 0;
    while (1)
    {
        counter++;
        if (counter % 100000000 == 0)
        {
            printf("CPU task running: %lld\n", counter);
        }
    }
    return 0;
}
```

#### Compile, run, and monitor

The manual's background-launch commands are augmented with `$!` to record the actual PIDs for observation and cleanup.

```bash
gcc -Wall -Wextra io_bound.c -o io_bound
gcc -Wall -Wextra cpu_bound.c -o cpu_bound
./io_bound &
io_pid=$!
./cpu_bound &
cpu_pid=$!
pgrep -af 'io_bound|cpu_bound'
top -p "$io_pid,$cpu_pid"
```

Press **q** to leave `top`, then stop the two programs in the same terminal:

```bash
kill "$io_pid" "$cpu_pid"
```

The file workload spends much of its time sleeping, so low CPU utilization alone does not prove a physical-disk bottleneck. Both programs loop indefinitely. The source's signed counter would have undefined behavior if it eventually overflowed; the exercise is intended as a short observation.

**Code source:** supplied Experiment 7 manual, pages 1–5. PID capture and the filtered `top` invocation are convenience additions.

### Expected observations

| Property | Counter program | File-operation and sleep program |
| --- | --- | --- |
| Main activity | Repeated arithmetic | File operations followed by sleep |
| Typical CPU use | Higher | Lower |
| Common sampled state | `R` | `S` during timer sleep |
| CPU-time growth | Faster | Slower |

### Conclusion

The comparison shows how computation and waiting affect scheduling and CPU accounting. Low processor use must be interpreted in the context of the program's actual activity, including deliberate sleep and buffered I/O.

### Reference

Supplied manual: *Experiment 7 CPU-Bound vs I_O-Bound C Programs Manual.pdf*, pages 1-6.
