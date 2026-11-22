# Experiment 2

## Combined Redirection and the sed Stream Editor

### Aim

To display the current date while saving logged-in-user information to a file, and to use `sed` for selecting lines, deleting lines, substituting text, and rearranging captured words.

### Theory

#### 1. Combining commands and controlling output

The shell treats command sequencing and redirection as separate operations. In the following example, `date` writes to the terminal and `who` writes to the file.

```bash
date; who > logged_users.txt
```

The semicolon runs the commands sequentially regardless of the first command's exit status. With `date && who > logged_users.txt`, the second command runs only if `date` succeeds. The redirection belongs to `who`, so it does not also capture the date. To capture both commands, a grouped command can be redirected.

```bash
{ date; who; } > session_report.txt
```

`who` reads login-session records. Some containers or terminal environments provide no such records. `whoami` reports the current effective username and therefore answers a different question from listing all logged-in sessions.

#### 2. Stream editing

`sed` processes input as a stream. In its ordinary cycle, it reads a line into a working area called the pattern space, applies editing commands, writes the resulting line, and advances to the next line. This supports automated transformations without opening an interactive editor.

The basic form is `sed 'editing-command' input.txt`. Editing commands can include an address that selects a line number, a range, or a matching pattern. Single quotes keep the shell from expanding characters such as `$` and backslashes in the script.

#### 3. Selecting and deleting lines

| Command | Effect |
| --- | --- |
| `sed -n 'p' Names.txt` | Print all lines explicitly |
| `sed -n '2p' Names.txt` | Print line 2 |
| `sed -n '2,4p' Names.txt` | Print lines 2 through 4, inclusive |
| `sed '2d' Names.txt` | Omit line 2 from output |
| `sed '1d' Names.txt` | Omit the first line from output |
| `sed '$d' Names.txt` | Omit the last line from output |

The `-n` option suppresses normal automatic printing, and `p` prints explicitly. Combining them prevents unwanted lines or duplicate output. The command `d` discards the selected line and starts the next input cycle. These commands produce transformed output unless an option such as `-i` requests file modification.

#### 4. Substitution and regular expressions

Substitution has the form `s/pattern/replacement/flags`. Without `g`, the first matching occurrence on each line is replaced. With `g`, every matching occurrence on that line is replaced. For the input `John John`, `sed 's/John/Amit/'` produces `Amit John`, while `sed 's/John/Amit/g'` produces `Amit Amit`. A line containing `Rahul John` becomes `Rahul Amit` with either command because it has only one match. GNU sed manual

An initial `^` anchors a pattern to the beginning of a line, while `$` anchors it to the end. Thus, `s/^John/Amit/` affects a leading `John`, and `s/John$/Amit/` affects a trailing `John`. The pattern `.` matches a character, and `*` repeats the preceding expression zero or more times. Inside `[^ ]`, the caret negates the character set, so this expression matches a character other than a literal space.

For simple space-separated text, `sed 's/ /_/g' Names.txt` converts every space to an underscore. A literal-space pattern does not include tabs.

#### 5. Capturing and swapping words

Basic regular expressions use `\(...\)` for a capture group. The replacement can refer to captured text using `\1`, `\2`, and subsequent group numbers.

```bash
sed 's/^\([^ ]*\) \([^ ]*\)/\2 \1/' Names.txt
```

The expression captures the first two space-separated parts of a line and emits them in reverse order. For example, `John Doe` becomes `Doe John`. The initial `^` anchors the selection, the literal space separates the groups, and `\2 \1` determines their new order. Any text following the matched part remains after the replacement.

This example assumes a suitable two-word prefix separated by one space. Leading spaces, repeated spaces, and tabs require a more suitable whitespace pattern. The matched text and the replacement punctuation together determine the output. For example, `{\1}` inserts braces around group 1.

#### 6. Saving a transformation

Output can be redirected to a different file, or GNU `sed -i` can edit the input in place. Adding a backup suffix, as in `sed -i.bak 's/John/Amit/g' Names.txt`, retains an original backup. Using the same file as both ordinary input and a `>` destination causes the shell to truncate that file before `sed` reads it, so separate input and output filenames are required for ordinary redirection.

### Programming code and terminal commands

These are Bash and `sed` commands from the manual. Prepare `Names.txt` with sample names, then run each command separately. Commands without `-i` display transformed text without changing the input file.

#### Combined commands and redirection

```bash
date; who > logged_users.txt
```

The date appears on the terminal; only `who` writes to `logged_users.txt`. To save both outputs, the grouped form discussed in the theory is:

```bash
{ date; who; } > session_report.txt
```

#### Print or remove selected lines from the output

```bash
sed -n 'p' Names.txt
sed -n '2p' Names.txt
sed -n '2,4p' Names.txt
sed '2d' Names.txt
sed '1d' Names.txt
sed '$d' Names.txt
```

#### Substitution and anchors

```bash
sed 's/John/Amit/' Names.txt
sed 's/John/Amit/g' Names.txt
sed 's/^John/Amit/' Names.txt
sed 's/John$/Amit/' Names.txt
sed 's/ /_/' Names.txt
sed 's/ /_/g' Names.txt
```

Without `g`, substitution replaces the first match on each line, wherever that match occurs. For example, `Rahul John` becomes `Rahul Amit` with `s/John/Amit/`.

#### Regular expressions, captured groups, and word swapping

```bash
sed 's/.at/XXX/' file.txt
sed 's/[^ ]*/WORD/' Names.txt
sed 's/\([^ ]*\)/{\1}/' Names.txt
sed 's/\([^ ]*\) \([^ ]*\)/{\1}{\2}/' Names.txt
sed 's/^\([^ ]*\) \([^ ]*\)/\2 \1/' Names.txt
```

For a line such as `Amit Sharma`, the final command displays `Sharma Amit`. It assumes the first two fields are separated by one literal space.

#### Save a change with a backup

The manual includes the backup form below. It changes `Names.txt` and saves its previous contents in `Names.txt.bak`.

```bash
sed -i.bak 's/John/Amit/g' Names.txt
```

Code source: supplied Experiment 2 manual, pages 1–15. The grouped redirection example is an explanatory addition.

### Expected observations

The date appears on screen while session information is stored separately. Line addresses select the requested input, substitutions depend on both the pattern and flags, and capture groups reorder matched text. File contents change only when the selected workflow writes the transformation back.

### Conclusion

Shell redirection controls data destinations, while `sed` applies repeatable, pattern-based transformations to textual data.
