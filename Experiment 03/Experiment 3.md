# Experiment 3

## Bash Scripting, Conditionals, and Loops

### Aim

To understand Bash script execution, positional parameters, conditional tests, command substitution, and loops, and apply them to identifying path types and converting filenames to uppercase.

### Theory

#### 1. Shell scripts and interpreters

A shell script is a text file containing commands and control structures. Saving a sequence as a script makes an operation repeatable and allows its inputs to vary. The line `#!/bin/bash` identifies Bash as the interpreter when the script is executed directly. Run `chmod +x script.sh` before `./script.sh`, or invoke the interpreter explicitly with `bash script.sh`.

Bash includes extensions such as arithmetic loops and substring expansion. A script that uses these features should be run with Bash. Choosing the intended interpreter avoids dependence on what a system happens to provide as `sh`.

#### 2. Arguments, variables, and quoting

Command-line arguments make scripts reusable. For `./check_type.sh "Lab Notes"`, Bash makes the path available as `$1`.

| Parameter | Meaning |
| --- | --- |
| `$0` | Invocation name or path of the script |
| `$1`, `$2`, ... | Individual positional arguments |
| `$#` | Number of arguments supplied after the script name |
| `"$@"` | All positional arguments, each retained as a separate item |
| `$?` | Exit status of the most recently completed command |

Assign values without spaces around `=`, as in `target="$1"`. Quoting expansions such as `"$target"` preserves spaces and prevents filename-pattern expansion from changing the intended argument. In a loop, `for file in "$@"` processes each supplied path once, including a path containing spaces.

#### 3. Conditions and exit status

An `if` statement decides what to execute from a command's exit status. Status 0 means success, while a nonzero status means failure or a false test. The commands `test` and `[` evaluate conditions. In `[ ... ]`, spaces separate the command, operands, and closing bracket.

```bash
if [ -f "$target" ]; then
    printf '%s\n' 'Regular file'
elif [ -d "$target" ]; then
    printf '%s\n' 'Directory'
else
    printf '%s\n' 'Another type or a missing path'
fi
```

Common file tests include `-e` for existence, `-f` for a regular file, `-d` for a directory, `-s` for nonzero size, and `-r`, `-w`, and `-x` for access checks. These path tests generally follow symbolic links. An `-e` test may fail for a dangling symbolic link, so a separate `-L` test is useful when link identity matters.

Integer comparison uses `-eq`, `-ne`, `-lt`, `-le`, `-gt`, and `-ge`. String tests include `-z` for an empty value and `=` for equality. A script expecting one path should check `[ "$#" -eq 1 ]` before using that argument. `exit 1` can report invalid invocation, while `exit 0` indicates success.

#### 4. Command substitution and arithmetic

`$(command)` captures a command's standard output, with trailing newline characters removed. For example, `today=$(date)` stores formatted date information. Arithmetic expansion, `$((expression))`, calculates an integer result. Addition, subtraction, multiplication, division, and remainder are available. Integer division discards the fractional part.

These two expansion forms serve different purposes. Command substitution executes a command, while arithmetic expansion evaluates a numeric expression.

#### 5. Iteration

A `for` loop repeats its body for successive values. A `while` loop repeats as long as its condition succeeds. The loop variable must be initialized where appropriate, and the body must eventually change the condition if termination is required.

```bash
count=1
while (( count <= 5 )); do
    printf '%s\n' "$count"
    ((count++))
done
```

The example displays 1 through 5. Bash also supports lists such as `{1..5}`, filename patterns, and arithmetic `for` loops. For processing complete lines from a file, `while IFS= read -r line` preserves spaces more reliably than splitting the result of `$(cat file)` into words.

#### 6. Application to the lab tasks

The path-classification task first checks the number of arguments, then uses file tests to select the appropriate message. This separates invalid invocation from a valid invocation that names a missing path.

The uppercase-renaming task combines a loop, text conversion, and `mv`. The command `printf '%s' "$name" | tr '[:lower:]' '[:upper:]'` generates an uppercase name. The script should convert the final filename component while retaining the directory component, check that the source exists, and check for an existing destination. This avoids accidentally uppercasing the directory path or overwriting a different file. Use `mv -- "$old_path" "$new_path"` so a filename beginning with `-` is treated as a path.

### Programming code

Save each complete script under the filename shown and run it in Bash on Linux. The two task scripts below are transcribed from the manual; PDF page breaks have been removed.

#### Task 1: Check a file or directory — `check_type.sh`

```bash
#!/bin/bash
if [ $# -ne 1 ]; then
  echo "No Argument passed, Try Again with an Argument"
  exit 1
fi
target="$1"
if [ -f "$target" ]; then
  echo "$target is a regular file."
elif [ -d "$target" ]; then
  echo "$target is a directory."
elif [ -e "$target" ]; then
  echo "$target exists, but is neither a regular file nor a directory."
else
  echo "Error: $target does not exist."
fi
```

Run:

```bash
chmod +x check_type.sh
./check_type.sh /etc/passwd
./check_type.sh /var/log
```

The original argument-count message also appears when more than one argument is passed. The final missing-path branch prints an error message but does not explicitly return a nonzero exit status.

#### Task 2: Rename filenames to uppercase — `rename.sh`

```bash
#!/bin/bash
# Loop through all arguments
for file in "$@"
do
  # Convert filename to uppercase and rename
  mv "$file" "$(echo "$file" | tr '[:lower:]' '[:upper:]')"
done
```

Run on practice files in the current directory:

```bash
touch notes.txt report.txt
chmod +x rename.sh
./rename.sh notes.txt report.txt
```

This source version uppercases the entire argument. Use simple basenames in the current directory, with no existing uppercase destination names. Supplying a path also uppercases its directory components, and `mv` can replace an existing destination.

#### Additional loop and arithmetic examples

The following examples collect the manual's basic loop forms. `count=1` has been added to initialize the `while` example explicitly.

```bash
#!/bin/bash
count=1
while (( count <= 5 ))
do
    echo "$count"
    ((count++))
done

for name in Ali Rahul Aman
do
    echo "Hello $name"
done

for i in {1..5}
do
    echo "Count: $i"
done

for (( i=1; i<=5; i++ ))
do
    echo "Number: $i"
done

a=10
b=3
echo "Addition:       $((a + b))"
echo "Subtraction:    $((a - b))"
echo "Multiplication: $((a * b))"
echo "Division:       $((a / b))"
echo "Modulus:        $((a % b))"
echo "Exponentiation: $((a ** b))"
```

**Code source:** supplied Experiment 3 manual, pages 9–10 and 12–14.

### Expected observations

The path script selects the branch matching the supplied path type. Loops perform one iteration per supplied argument, and quoted arguments retain their boundaries. Renaming changes a file's directory entry, while its stored contents retain the same meaning and bytes.

### Conclusion

Bash combines commands with parameters, decisions, and iteration to automate file-system tasks. Correct quoting, input checks, and exit-status handling make the automation predictable.

### Reference

Supplied manual: *Experiment 3 Bash Scripting, Conditionals & Loops Manual.pdf*, pages 1-17.
