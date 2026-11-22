Experiment 1
Linux System Administration and Process Management
Aim
To understand the Linux command-line environment and use commands for directory navigation, file management, permissions, text processing, job control, process inspection, signals, and scheduling priority.

Theory
1. Linux, the shell, and the file system
Linux is a multiuser, multitasking operating system. Its kernel manages processor time, memory, devices, and access to files. A shell such as Bash accepts commands, expands their arguments, sets up input and output, and runs built-in commands or external programs. A terminal provides the interface through which a user interacts with the shell.

The Linux file system is organized as a hierarchy beginning at /. An absolute path starts at this root, while a relative path is interpreted from the current working directory. The symbols . and .. refer to the current and parent directories. Bash expands ~ to the user's home directory, while $HOME contains the configured home-directory path. Filenames are normally case-sensitive.

pwd displays the current directory, cd changes it, and mkdir creates directories. The option mkdir -p also creates missing parent directories. rmdir removes empty directories. Quoting paths, as in cd "Lab Files", keeps a path containing spaces together as one argument.

2. File operations and permissions
Files have metadata such as an owner, group, permissions, size, and timestamps. The command ls -l displays these attributes. Adding -a includes names beginning with a dot, and -h makes sizes easier to read.

Command	Purpose
touch notes.txt	Create an empty file if absent, or update timestamps if present
cp source.txt copy.txt	Copy a file
cp -r source_dir copy_dir	Copy a directory tree
mv old.txt new.txt	Move or rename a file
rm -i notes.txt	Request confirmation before removing the named file
cat notes.txt	Write a file's contents to standard output
head -n 5 notes.txt	Display the first five lines
tail -n 5 notes.txt	Display the last five lines
Permissions are assigned separately to the owner, group, and others. Their numeric values are read = 4, write = 2, and execute = 1. Thus, chmod 755 script.sh gives the owner all three permissions and gives the group and others read and execute permission. For a directory, read permits listing names, write permits changing directory entries subject to other checks, and execute permits traversal and lookup.

3. Standard streams and redirection
A process normally starts with three standard file descriptors.

Descriptor	Stream	Typical purpose
0	Standard input	Receive input
1	Standard output	Produce ordinary output
2	Standard error	Report diagnostics
The shell can connect these streams to files. > creates or truncates an output file, >> appends output, < supplies input from a file, and 2> redirects diagnostics. In Bash, &> redirects both standard output and standard error to one file. For example, cat first.txt second.txt > combined.txt concatenates two inputs into a destination file.

A pipe connects one command's standard output to the next command's standard input. In sort names.txt | uniq, sorting places equal lines together so uniq can remove adjacent duplicates. The command head -n 37 data.txt | tail -n 7 selects lines 31 through 37 when the input has at least 37 lines. tee copies a stream to both the terminal and a file. xargs converts input into command arguments, with -0 used for input containing null-delimited filenames.

4. Finding and editing information
grep selects lines by a pattern. Useful options include -i for case-insensitive matching, -v for nonmatching lines, -n for line numbers, -c for the number of matching lines, and -w for whole-word matches.

find traverses directory trees and evaluates conditions. Examples include find . -name '*.txt', find . -iname '*.txt', find . -user "$USER", and find . -empty. Quote filename patterns so the shell passes them to find. In GNU find, -size uses units such as c for bytes, k for KiB, M for MiB, and b for 512-byte blocks. A leading + or - selects sizes greater than or less than the specified value, with unit-rounding rules applying.

Vim uses modes. Press i to insert text and Esc to return to Normal mode, then use :w, :q, or :wq to save or quit. Nano presents shortcuts directly, including Ctrl+O to save and Ctrl+X to exit.

5. Processes and shell jobs
A process is an executing instance of a program with a PID, execution state, memory, and resources. Its parent is identified by a PPID. A shell job is a command or pipeline managed by that shell. A job number such as %1 is meaningful within that shell and differs from a system PID.

Adding & starts a job in the background. jobs lists the current shell's jobs. Ctrl+Z normally sends SIGTSTP to suspend the foreground job, bg %1 resumes it in the background, and fg %1 brings it to the foreground. Background execution allows the shell to accept more commands, but it does not itself reduce the job's CPU demand.

ps gives a process snapshot. ps -e selects all processes, ps -ef includes parent and ownership information, and ps aux includes resource statistics. Use ps -p PID -f for a selected PID. top repeatedly refreshes process information, including CPU use, memory use, and state. Accumulated CPU time measures processor work and can be much smaller than elapsed time for a sleeping process.

6. Signals and priority
kill sends a signal. Its default is SIGTERM, which requests termination and allows a process with a suitable handler to perform cleanup. SIGKILL forces termination and cannot be caught or ignored. SIGSTOP suspends a process, and SIGCONT resumes it. Signal names make the intended action clear, for example kill -CONT PID.

For ordinary Linux scheduling, a nice value ranges from -20 to 19. A larger value generally reduces scheduling preference under competition. nice -n 10 command starts a command with a nice adjustment of +10, while renice 10 -p PID sets an existing process's nice value to 10. A nice value specifies scheduling preference rather than a fixed CPU percentage. Increasing scheduling preference may require additional privilege.

Programming code and terminal commands
This experiment uses Linux terminal commands rather than a standalone C program. The following examples are selected from the manual, with file-operation paths changed to a practice directory. Run commands individually in Bash and prepare the named sample text files before using them.

Directory and file operations
mkdir -p ~/os-lab-exp1/projects/app/src
cd ~/os-lab-exp1
pwd
ls -lah
touch notes.txt
cp -i notes.txt copy.txt
mv -i copy.txt renamed.txt
chmod 755 projects
cat notes.txt
cat -n notes.txt
head -n 5 notes.txt
tail -n 5 notes.txt
rmdir projects/app/src
rm -i renamed.txt
Use nano notes.txt or vim notes.txt to enter sample text. In Nano, save with Ctrl+O, then exit with Ctrl+X. In Vim, press i to insert text, then Esc and :wq to save and quit.

Redirection, pipelines, and searching
Each line below is a separate example using the input files named in the manual. > replaces an output file; >> appends to it.

cat data1 data2 > data3
grep "Doctor" users.csv > doctor_data
cat names.txt countries.txt | sort
cat names.txt | sort | uniq
cat data.txt | head -n 37 | tail -n 7
ls | tee test.txt
cat file_names.txt | xargs touch
grep -i "error" logs.txt
grep -v "error" logs.txt
grep -c "Doctor" users.csv
grep -w "Carly" users.csv
grep -n "Carly" users.csv
find . -size 4M
find . -name "a123"
find . -iname "a123"
find . -user root
find . -empty
The xargs touch example assumes simple whitespace-separated filenames; names containing spaces require a different delimiter convention. find . -size 4M uses sizes rounded up to 1 MiB units, rather than an exact byte-size equality.

Job control and process observation
sleep 100 &
jobs
fg
While the sleep command is in the foreground, press Ctrl+Z, then enter bg to resume it in the background. fg %1 or bg %1 selects job 1 if that is the job number reported by jobs.

ps
ps -e
ps -ef
ps aux
ps -o pid,ni,cmd
kill -l
top
Press q to leave top. The next example uses the manual's priority and signal commands, adding $! to capture the actual PID instead of copying an example PID from the PDF. Run it while the newly started sleep process is still active.

nice -n 10 sleep 300 &
lab_pid=$!
ps -p "$lab_pid" -f
renice 15 -p "$lab_pid"
ps -p "$lab_pid" -o pid,ni,cmd
kill -STOP "$lab_pid"
kill -CONT "$lab_pid"
kill -15 "$lab_pid"
Code source: supplied Experiment 1 manual, pages 1–15. Paths and PID capture have been adapted for the practice examples.

Expected observations
Directory and file commands change or reveal the selected file-system objects.
Redirection changes where output is recorded, and pipelines pass data between commands.
jobs shows shell-managed jobs, while process tools can inspect processes across the system.
Stopping, continuing, and terminating a process produce different observable states.
Priority and CPU-use readings depend on system load and the selected process.
Conclusion
These commands provide the foundations for Linux administration by combining file-system management, stream processing, and process control through a common shell interface.
