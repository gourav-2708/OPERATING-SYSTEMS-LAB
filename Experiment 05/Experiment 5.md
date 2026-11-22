# Experiment 5

## String Length and Substring Extraction in Bash and C

### Aim

To understand how Bash and C represent and process strings, calculate string length, extract a substring from a selected starting position, and compare execution using the `time` command.

### Theory

#### 1. Strings and indexing

A string is a sequence of characters. Both lab implementations use zero-based indexing, so the first position is 0. A substring is a consecutive portion selected by a starting position and a requested length.

For the ASCII string `OperatingSystem`, the length is 15. The letter `S` begins at index 9, so selecting six characters from position 9 produces `System`. The space in `hello world` is also part of the string and contributes to its length.

#### 2. String operations in Bash

Bash stores the input as a shell variable. The command `IFS= read -r str` reads a line while preserving leading and trailing whitespace and treating backslashes literally. A prompt may be supplied with the Bash-specific `-p` option.

| Expansion | Meaning |
| --- | --- |
| `${#str}` | Length of the value stored in `str` |
| `${str:start:length}` | Substring beginning at `start` with the requested `length` |

```bash
str='OperatingSystem'
start=9
length=6
printf 'Length: %s\n' "${#str}"
printf 'Substring: %s\n' "${str:start:length}"
```

The example produces a length of 15 and the substring `System`. Parameter expansion performs these operations within Bash. The lab should validate nonnegative numeric values for `start` and `length` to define a consistent input policy. Negative values have additional meanings in Bash substring expressions and fall outside this lab's simple indexing rule.

#### 3. Strings in C

C commonly represents a string as a character array terminated by the null character `\0`. The terminator marks the end of the string and is separate from the printable characters. The array capacity includes space for that terminator.

For `char str[100]`, `fgets(str, sizeof str, stdin)` reads at most 99 input bytes and adds a null terminator on success. If a newline is read before the capacity is reached, it is retained in the array. The return value must be checked before the array is used. Longer input may leave additional characters in the input stream.

The manual's counting loop advances until it reaches either `\n` or `\0`. Stopping at the newline gives the length of the entered text rather than counting the Enter key as part of that text. A `strlen` call, by comparison, counts every byte before `\0`, including a retained newline.

#### 4. Extracting the substring in C

The program uses array indexing to print the selected part of the string. It first determines the string length, then prints while the index is below both `start + length` and the counted length. The manual's implementation does not validate the starting position. A robust version should validate the input before computing the available suffix as follows.

```text
available = count - start
characters_to_print = minimum(requested_length, available)
```

This calculation assumes `0 <= start <= count` and a nonnegative requested length. Starting at `count` gives an empty substring. Negative indices or indices beyond the allocated object are invalid. Computing the available length first also avoids depending on an unchecked `start + length` sum that could overflow.

The manual prints one character at a time with `%c`. If the substring is instead copied into another array, that destination needs enough capacity for all copied characters plus a terminating `\0`.

#### 5. Character-count assumptions

The supplied examples use ASCII, where one character occupies one byte. A C loop over `char` elements counts bytes. Bash can count characters according to the active locale. Multibyte text can therefore produce different results or be split incorrectly by a byte-based C implementation. Using the same ASCII input gives a clear comparison for this experiment.

#### 6. Comparing execution time

The C program is compiled into an executable using GCC. Bash interprets the shell script while it runs. The `time` command or Bash keyword can report three useful measurements.

| Field | Interpretation |
| --- | --- |
| `real` | Elapsed wall-clock time, including time spent waiting |
| `user` | CPU time spent executing user-space instructions |
| `sys` | CPU time spent in the kernel on behalf of the program |

Typing interactively contributes to elapsed time and can dominate a short experiment. For a useful comparison, provide identical prepared input, compile the C program before timing it, and repeat the measurement. Process startup, shell startup, terminal output, caching, scheduling, and virtual-machine load can outweigh the substring operation itself. A single short run does not establish a general speed advantage.

### Programming code

The manual calls these files `exp7.sh` and `exp7.c`, although its topic and title are Experiment 5. The filenames below are normalized to `exp5.sh` and `exp5.c`; the program logic is transcribed from the supplied manual.

#### Shell script — `exp5.sh`

```bash
#!/bin/bash
read -p "Enter a string: " str
echo "Length of string: ${#str}"
read -p "Enter starting position: " start
read -p "Enter number of characters: " length
echo "Substring: ${str:$start:$length}"
```

#### C program — `exp5.c`

```c
#include <stdio.h>
int main()
{
    char str[100];
    int start, length, i, count = 0;
    printf("Enter a string: ");
    fgets(str, 100, stdin);
    while (str[count] != '\0' && str[count] != '\n')
    {
        count++;
    }
    printf("Length of string: %d\n", count);
    printf("Enter starting position: ");
    scanf("%d", &start);
    printf("Enter number of characters: ");
    scanf("%d", &length);
    printf("Substring: ");

    for (i = start; i < start + length && i < count; i++)
    {
        printf("%c", str[i]);
    }
    printf("\n");
    return 0;
}
```

#### Compile and run

```bash
bash exp5.sh
gcc -Wall -Wextra exp5.c -o exp5
./exp5
```

For both programs, enter `OperatingSystem`, start position `9`, and length `6`. The expected substring is `System`; positions begin at zero.

The C source assumes successful input, nonnegative integers, and a short string. It does not check `fgets` or `scanf`, reject negative indices, or guard `start + length` against overflow. Use valid small values for this source example; input validation described in the theory is needed before extending it to arbitrary input.

To compare timings using the same prepared input rather than typing at different speeds:

```bash
printf 'OperatingSystem
9
6
' > input.txt
time bash exp5.sh < input.txt
time ./exp5 < input.txt
```

The prepared-input commands are an added reproducibility aid. A single short run is not a reliable performance benchmark.

**Code source:** supplied Experiment 5 manual, pages 1 and 4–5; compilation and timing are discussed on pages 8–9.

### Expected observations

Both implementations should give the same length and substring for valid ASCII inputs. Timing values may vary between runs, and CPU time can be smaller than elapsed time when input or scheduling delays occur. Record measured times rather than treating example times as experimental results.

### Conclusion

Bash expresses string operations through parameter expansion, while C exposes array storage, terminators, and bounds directly. The comparison connects high-level scripting with lower-level memory representation and execution-time measurement.

### Reference

Supplied manual: *Experiment 5 String Operations in Shell Script & C Manual.pdf*, pages 1-9. The manual's example names `exp7.sh` and `exp7.c` refer to this Experiment 5 task.
