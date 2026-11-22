# Operating Systems Lab

Lab records for the Operating Systems course: ten experiments covering Linux commands, shell scripting, text processing, memory management, and process creation in C.

## Experiments

| No. | Experiment | Topics | Write-up | Output |
| :-: | --- | --- | :-: | :-: |
| 1 | [Linux System Administration and Process Management](./Experiment%2001) | Files, permissions, pipes, `ps`, `kill`, `nice` | [Read](./Experiment%2001/Experiment%201.md) | [PDF](./Experiment%2001/Experiment1.pdf) |
| 2 | [Combined Redirection and the sed Stream Editor](./Experiment%2002) | Redirection, `sed` | [Read](./Experiment%2002/Experiment%202.md) | [PDF](./Experiment%2002/Experiment2.pdf) |
| 3 | [Bash Scripting, Conditionals, and Loops](./Experiment%2003) | Bash, `if`, loops | [Read](./Experiment%2003/Experiment%203.md) | [PDF](./Experiment%2003/Experiment3.pdf) |
| 4 | [User Session Duration, Line Ranges, and Text Processing](./Experiment%2004) | `who`, `sed`, `awk` | [Read](./Experiment%2004/Experiment%204.md) | [PDF](./Experiment%2004/Experiment4.pdf) |
| 5 | [String Length and Substring Extraction in Bash and C](./Experiment%2005) | Strings in Bash and C | [Read](./Experiment%2005/Experiment%205.md) | [PDF](./Experiment%2005/Experiment5.pdf) |
| 6 | [Dynamic Memory Allocation and Virtual Memory in C](./Experiment%2006) | `malloc`, `free`, virtual memory | [Read](./Experiment%2006/Experiment%206.md) | [PDF](./Experiment%2006/Experiment6.pdf) |
| 7 | [CPU-Bound and I/O-Bound Programs Using top](./Experiment%2007) | C, `top` | [Read](./Experiment%2007/Experiment%207.md) | [PDF](./Experiment%2007/Experiment7.pdf) |
| 8 | [Observing Process Memory Maps Using pmap](./Experiment%2008) | `pmap`, memory maps | [Read](./Experiment%2008/Experiment%208.md) | [PDF](./Experiment%2008/Experiment8.pdf) |
| 9 | [Process Identification, Creation, and Termination in C](./Experiment%2009) | `getpid`, `fork`, `exit` | [Read](./Experiment%2009/Experiment%209.md) | [PDF](./Experiment%2009/Experiment9.pdf) |
| 10 | [Parent-Child Synchronization Using fork() and wait()](./Experiment%2010) | `fork`, `wait` | [Read](./Experiment%2010/Experiment%2010.md) | [PDF](./Experiment%2010/Experiment10.pdf) |

## Repository layout

Each experiment has its own folder:

```
Experiment NN/
├── Experiment N.md   ← aim, theory, code, expected observations, conclusion
├── ExperimentN.pdf   ← output
└── PDF.md            ← download link for the PDF (Experiments 02–10)
```

## Environment

The commands and programs are meant for a Linux system with Bash and GCC. Compile a C program with:

```bash
gcc program.c -o program
./program
```
