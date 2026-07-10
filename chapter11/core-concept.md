# Chapter 11: Introduction to Shell Scripts

## Structural Outline and Core Concepts

### 11.1 Shell Script Basics
A shell script is a file containing a series of commands designed to be executed by the shell, starting with a "shebang" (e.g., `#!/bin/sh`) indicating the interpreter. Scripts require read and execute permissions (`chmod +rx`) to run, simplifying and automating repetitive tasks at the shell prompt. However, they are limited and less suitable for complex string manipulation or heavy computation compared to languages like Python or Perl.

### 11.2 Quoting and Literals
Quoting is essential for creating literals to prevent the shell from incorrectly interpreting or expanding characters (like `$` or `*`). Single quotes (`'`) strictly prevent any substitution, ensuring the entire enclosed string is treated as a single parameter. Double quotes (`"`) act similarly but allow the shell to expand variables within them, providing flexibility when combining static text and variable data.

### 11.3 Special Variables
Scripts interact with arguments and the environment through special, read-only variables. Positional parameters like `$1` and `$2` capture individual arguments, while `$#` counts them and `$@` represents the complete list. Other useful variables include `$0` for the script's name, `$$` for the process ID, and `$?` for the last command's exit code.

### 11.4 Exit Codes
Every Unix program returns an exit code to its parent process upon completion, stored in the `$?` variable. A zero exit code typically indicates successful execution, while a non-zero value indicates an error or a specific condition (e.g., `grep` finding no match). Properly evaluating and returning exit codes is crucial for robust script flow control and error handling.

### 11.5 Conditionals
The Bourne shell uses `if`/`then`/`elif`/`else` structures, often paired with the `test` command (or `[`) to evaluate exit codes. Tests can evaluate file properties (e.g., checking if a file exists with `-f`), compare strings (`=`), or perform arithmetic comparisons (`-eq`). The `case` construct is another conditional tool optimized for pattern matching strings without evaluating exit codes, serving as an efficient alternative to chained `if` statements.

### 11.6 Loops
Loops allow repeated execution of commands. The `for` loop iterates over a defined list of space-delimited values, assigning each to a variable per iteration. The `while` loop continues executing as long as a specified command returns a successful (zero) exit code, and can be terminated early using the `break` statement.

### 11.7 Command Substitution
Command substitution allows the standard output of a command to be captured and used as an argument or stored in a variable. The preferred syntax is `$()`, which executes the inner command and replaces it with the resulting output on the command line. While powerful, overuse of command substitution can impact performance compared to built-in shell expansions.

### 11.8 Temporary File Management
Scripts often require temporary files to collect and pass output between commands. It's critical to generate unique filenames to avoid conflicts, typically using tools like `mktemp`. Additionally, robust scripts use signal handlers (`trap`) to ensure temporary files are cleaned up even if the script is abruptly terminated by the user.

### 11.9 Here Documents
Here documents allow you to feed large blocks of text directly into the standard input of a command without using multiple `echo` statements. Enclosed between `<<EOF` and a terminating `EOF` marker, they keep multi-line formatting intact. Shell variables inside here documents are still expanded, making them ideal for generating dynamic reports.

### 11.10 Important Shell Script Utilities
Several external programs are staples in shell scripting for manipulating text and files. Tools like `basename` strip paths and extensions, while `awk` and `sed` offer powerful stream editing and text extraction capabilities. When dealing with too many files for a single command buffer, `xargs` bridges the gap by building manageable command-line arguments from standard input.

### 11.11 Subshells
Subshells are independent, temporary shell environments spawned by enclosing commands in parentheses. They inherit a copy of the parent shell's environment, allowing for local changes to paths or directories that do not affect the main script once the subshell exits. This is heavily used to group pipeline operations or isolate environmental variables.

### 11.12 Including Other Files in Scripts
The dot operator (`.`) sources another file directly into the current shell process. Unlike executing a script as a separate command, sourcing runs the commands in the current environment, allowing the script to read and retain variable definitions and configurations established in the sourced file.

### 11.13 Reading User Input
The `read` built-in command pauses script execution to capture a line of text from standard input and assign it to a variable. This is primarily used to build interactive prompts or require confirmation before executing dangerous operations. 

### 11.14 When (Not) to Use Shell Scripts
While shell scripts excel at managing files and gluing commands together, they can become unwieldy for complex logic. If a script requires convoluted string parsing or heavy arithmetic, it is often a sign to transition to a more capable scripting language like Python.


## Commands & Utilities

- `chmod`: Changes file permissions; `chmod +rx script` makes a script readable and executable.
- `shift`: Built-in command that shifts positional parameters to the left (e.g., `$2` becomes `$1`), useful for parsing arguments in a loop.
- `[` or `test`: Evaluates conditional expressions for files, strings, and arithmetic, returning an exit code of 0 if true.
- `mktemp`: Generates a unique temporary file (or directory) safely, preventing naming conflicts and race conditions.
- `trap`: Intercepts system signals (like `SIGINT` from CTRL-C) to execute cleanup code, such as deleting temporary files.
- `basename`: Strips directory paths and optional suffixes from filenames.
- `awk`: A programming language used in scripts primarily for extracting specific fields from text streams (e.g., `awk '{print $5}'`).
- `sed`: A stream editor used to automatically perform search-and-replace or line deletions on an input stream.
- `xargs`: Reads items from standard input and executes a specified command, bypassing command-line length limits.
- `expr`: A basic command for evaluating arithmetic operations or simple string matching.
- `exec`: Replaces the current shell process with a new command, saving resources but preventing the original script from resuming.
- `read`: Built-in command that accepts user input from the terminal and stores it in a specified variable.
