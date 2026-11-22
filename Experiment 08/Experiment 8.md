# Experiment 8

## Observing Process Memory Maps Using pmap

### Aim

To examine the virtual-memory mappings of the CPU-bound and file-operation programs from Experiment 7 and interpret mapped size, resident memory, dirty pages, permissions, and mapping names.

### Theory

#### 1. The process memory map

A process uses a virtual address space divided into mapped regions. Each mapping describes an address range, its permissions, and its association with an executable, library, file, or anonymous memory. A mapping is a virtual-memory region rather than a direct display of physical RAM addresses.

The command `pmap PID` summarizes the mappings of a selected process. `pmap -x PID` adds extended memory statistics. These tools rely on information exposed by Linux through `/proc`; permission restrictions and process termination can affect whether the information is available.

#### 2. Common regions

| Region | Role |
| --- | --- |
| Executable code | Machine instructions generated from the program |
| Read-only data | Constants and other data mapped without write permission |
| Data and BSS | Storage for relevant global and static objects |
| Heap or anonymous allocations | Memory managed for dynamic allocation and runtime needs |
| Stack | Storage associated with calls and some local objects |
| Shared libraries | Library code and data, such as the C runtime |
| Dynamic loader and other mappings | Support for loading and running the process |

These conceptual regions need not correspond one-to-one with rows in `pmap`. One executable or library can have several mappings with different permissions. Some allocations appear as anonymous regions, and a compiler may keep individual local variables in registers rather than placing them on the stack.

The programs in Experiment 7 contain different instructions but use a similar C runtime. Their maps can therefore have similar kinds of regions. Library operations may allocate memory even when the program's source contains no explicit `malloc` call.

#### 3. Interpreting pmap -x

| Column | Interpretation |
| --- | --- |
| `Address` | Starting virtual address of the region |
| `Kbytes` | Total mapped size of the region |
| `RSS` | Pages of that region currently resident in physical memory |
| `Dirty` | Reported memory associated with modified, dirty pages |
| `Mode` | Permissions and mapping attributes |
| `Mapping` | Associated file or descriptive region name |

Mapped size and resident size answer different questions. A region can cover a large address range while only some of its pages are resident. RSS can also include shared pages. Adding RSS values across processes can count the same physical pages more than once.

The dirty count concerns modified pages, including anonymous-memory pages. It should not automatically be interpreted as a quantity of user-file data waiting to be saved. File-backed and anonymous pages have different backing and writeback behavior.

#### 4. Permissions and isolation

The letters `r`, `w`, and `x` represent read, write, and execute access. Typical executable instructions appear in a read-and-execute region such as `r-x--`. Writable data and the stack commonly appear with a mode such as `rw---`. A read-only region may appear as `r----`.

These permissions help the operating system enforce appropriate access. An attempt to access memory contrary to its mapping can generate a fault and terminate the process. Identical-looking virtual addresses in two processes do not by themselves establish that the processes share physical storage.

#### 5. Reading an example row

The following row is illustrative and is adapted from the supplied manual.

```text
Address           Kbytes   RSS  Dirty Mode  Mapping
000076ec1b228000     1632  1056      0 r-x-- libc.so.6
```

This region begins at the displayed virtual address and maps 1632 KiB of the C library. Of that region, 1056 KiB is reported as resident, and zero dirty memory is reported. Its mode permits reading and executing. The final totals row sums the reported quantities for the displayed process mappings.

Addresses and quantities vary with executable layout, library versions, address-space randomization, allocation, and current memory use. Compare categories and measured quantities rather than expecting the example's exact addresses to reappear.

#### 6. Relationship to the preceding experiments

Experiment 6 changes allocation and touches memory to make residency observable. Experiment 7 changes CPU activity and waiting behavior. Experiment 8 reveals where memory is mapped for the running programs and how much is resident.

A process performing many calculations can use little memory, while another process can hold a large allocation and spend most of its time asleep. Memory-map size alone therefore does not predict processor demand. Record `pmap` and `top` measurements as separate observations of the same processes.

### Programming code and terminal commands

The supplied Experiment 8 manual introduces no new C source. It reuses `io_bound.c` and `cpu_bound.c` from Experiment 7 and adds `pmap` commands to inspect their memory mappings. Copy the two program listings from [Experiment 7.md](../Experiment%2007/Experiment%207.md) into files in the working directory before continuing.

#### Prepare the two processes

```bash
gcc -Wall -Wextra io_bound.c -o io_bound
gcc -Wall -Wextra cpu_bound.c -o cpu_bound
./io_bound &
io_pid=$!
./cpu_bound &
cpu_pid=$!
```

The `$!` assignments capture each PID immediately after launch. They replace the manual's example PID placeholders with values from the current run.

#### Inspect the mappings

```bash
pgrep -af 'io_bound|cpu_bound'
pmap "$cpu_pid"
pmap -x "$cpu_pid"
pmap "$io_pid"
pmap -x "$io_pid"
```

Compare address ranges, permissions, mapping names, virtual mapping size, RSS, and dirty memory. Actual addresses and totals vary across runs and systems. Memory-map similarity does not imply similar CPU usage.

#### Finish the observation

In the same shell, while these are still the running processes:

```bash
kill "$io_pid" "$cpu_pid"
```

**Code source:** supplied Experiment 8 manual, pages 1–2 and 13, reusing the Experiment 7 programs. Compilation, PID capture, and cleanup are included to make the observation sequence complete.

### Expected observations

Both programs should normally show executable and library mappings and a stack. Their total mapped memory, RSS, and dirty counts may differ, but high CPU usage does not require a larger address space. Record actual PIDs and totals while the programs are running, then stop the lab processes after observation.

### Conclusion

`pmap` connects a program's memory model to observable Linux mappings. Mapped size, residency, permissions, and backing information together explain how a process's virtual address space is organized.

### Reference

Supplied manual: *Experiment 8 Observing Process Memory Maps Using pmap Manual.pdf*, pages 1-13.
