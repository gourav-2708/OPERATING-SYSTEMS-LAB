# Experiment 4

## User Session Duration, Line Ranges, and Text Processing

### Aim

To understand shell scripts that calculate the elapsed duration of a user's login session, display a specified range of lines, and remove matching lines from several files.

### Theory

#### 1. Combining shell tools

A shell script can combine data collection, selection, calculation, and file editing. Variables store intermediate results, command substitution captures output, and pipelines pass records between tools. This experiment applies those mechanisms to three related tasks.

The session task uses `who`, `awk`, and `date`. The line-range task uses positional arguments and `sed`. The multiple-file task uses `shift` and a loop to apply a text operation to each requested file.

#### 2. Reading a login session

`who` reports recorded login sessions. In the format used by the supplied manual, the first field is the username, the second is the terminal, and the third and fourth contain the login date and time. AWK processes these rows as fields.

```bash
login_time=$(who | awk -v u="$user" '$1 == u {print $3, $4; exit}')
```

The `-v` option passes the Bash value into an AWK variable named `u`. The expression `$1 == u` selects an exact username match. The action prints the date and time and stops at the first matching record. Single quotes preserve AWK's `$1`, `$3`, and `$4` expressions while the shell processes the command line.

This calculation describes one selected login session. A user may have several simultaneous sessions, and the first returned record need not represent every session or the earliest login. Empty output indicates that no matching session record was found. The script must also account for environments without login records and for differences in the formatting of `who` output. Session duration includes idle periods and should be interpreted as elapsed login time.

#### 3. Converting times to a duration

Unix timestamps represent seconds relative to 1 January 1970 at 00:00:00 UTC. With GNU `date`, `date -d "$login_time" +%s` parses a date-time value into a timestamp, and `date +%s` reports the current timestamp. Their difference is an elapsed duration.

```text
elapsed_seconds = current_timestamp - login_timestamp
hours = elapsed_seconds / 3600
minutes = (elapsed_seconds % 3600) / 60
```

The divisions here are integer divisions. For example, 15,300 seconds gives 4 hours and 15 minutes. Including both the login date and time supports a session crossing midnight. Parsing failures, clock adjustments, and the precision of the source login record affect the reliability of the result. Both timestamps must refer to the intended time zone and date interpretation.

#### 4. Selecting an inclusive line range

A range-selection script accepts a filename, a starting line number, and an ending line number. After validation, it can construct a numeric `sed` address.

```bash
sed -n "${start},${end}p" "$file"
```

The double quotes allow Bash to expand `start` and `end`. The `-n` option disables automatic output, while `p` prints the selected range. Line numbering starts at 1, and both endpoints are included. For a range 3 through 6, the intended output contains four lines if all are present.

The script should require three arguments, a readable file, positive integer endpoints, and an ending value at least as large as the starting value. Validating numeric arguments also prevents unexpected characters from becoming part of the `sed` program. A range extending beyond the end of the file prints only the selected lines that actually exist.

#### 5. Processing several files with shift

For an invocation of the form `script.sh pattern file1 file2`, the first argument identifies the selection pattern. The script stores this argument and calls `shift`. After shifting, the remaining positional arguments are the file paths, so `for file in "$@"` can process them individually.

In `sed '/Aniket/d' names.txt`, the address `/Aniket/` selects lines containing the pattern and `d` removes those lines from the output. GNU `sed -i.bak` writes the result to the input file while keeping an original backup. Each file should be checked before modification.

A `sed` pattern is a regular expression. For example, a dot in the pattern means any matching character rather than necessarily a literal dot. The manual's `/$word/d` form can also match a substring within a longer word. To accept arbitrary literal text, the script must escape regular-expression characters and the chosen delimiter, or use a tool mode designed for fixed strings. Whole-word matching requires an explicit word-boundary policy.

### Programming code

The manual labels its three tasks as internal Experiments 1–3; all three belong to this repository's **Experiment 04**. Save each script separately and run it using Bash and GNU command-line tools on Linux.

#### Task 1: User session duration — `working_period.sh`

```bash
#!/bin/bash

read -p "Enter username: " user

login_time=$(who | awk -v u="$user" '$1 == u {print $3, $4; exit}')

if [ -z "$login_time" ]
then
    echo "User is not currently logged in."
else
    echo "Login time: $login_time"
    login_seconds=$(date -d "$login_time" +%s)
    current_seconds=$(date +%s)
    total_seconds=$((current_seconds - login_seconds))
    hours=$((total_seconds / 3600))
    minutes=$(((total_seconds % 3600) / 60))
    echo "Total working time: $hours hours $minutes minutes"
fi
```

Run:

```bash
bash working_period.sh
```

Enter a username listed by `who`. This script measures the elapsed time of the first matching login session; its output label “working time” does not establish active work time. It assumes a `who` date format understood by GNU `date -d` and does not check date-conversion errors.

#### Task 2: Display a line range — `lines.sh`

```bash
#!/bin/bash

if [ "$#" -ne 3 ]
then
    echo "Usage: $0 filename start_line end_line"
    exit 1
fi

file="$1"
start="$2"

end="$3"

sed -n "${start},${end}p" "$file"
```

Run with a readable sample file and positive line numbers, where the start does not exceed the end:

```bash
bash lines.sh names.txt 3 6
```

The source checks the number of arguments but does not validate the file or numeric range.

#### Task 3: Delete matching lines from several files — `delete_lines.sh`

```bash
#!/bin/bash

if [ "$#" -lt 2 ]
then
    echo "Usage: $0 word file1 [file2 ...]"
    exit 1
fi

word="$1"
shift

for file in "$@"
do
    sed -i "/$word/d" "$file"
done
```

Run on practice copies of the files:

```bash
bash delete_lines.sh Aniket names.txt students.txt teachers.txt
```

The original `sed -i` command edits each file in place and creates no backup. Its argument is a regular-expression pattern, not a guaranteed literal word match; `/` and other special characters require appropriate escaping. The example `Aniket` matches any line containing that sequence, including within a longer word.

**Code source:** supplied Experiment 4 manual, pages 1, 7–9. Filenames above identify the three extracted scripts.

### Expected observations

- The session script reports a duration for the selected recorded login, or identifies the absence of a matching record.
- A valid line interval displays exactly the available lines within the inclusive range.
- `shift` removes the pattern argument from the list processed as filenames.
- The deletion operation removes complete matching lines according to the selected pattern semantics.

### Conclusion

Shell variables, field processing, timestamp arithmetic, and stream editing can be combined to produce small administrative utilities. The interpretation of session records, line numbers, and text patterns determines whether their output answers the intended question.

### Reference

Supplied manual: *Experiment 4 Shell Scripting User Period, Line Range & Text Processing Manual.pdf*, pages 1-10.
