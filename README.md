# minishell

A simplified Unix shell implementation written in C, developed as part of the 42 School curriculum. This project recreates core functionality of **bash**, handling command parsing, execution, redirections, pipes, environment variables, and built-in commands.

---

## Table of Contents

- [About](#about)
- [Features](#features)
- [Built-in Commands](#built-in-commands)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Usage](#usage)
- [Shell Grammar](#shell-grammar)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Technology Stack](#technology-stack)
- [Testing](#testing)
- [References](#references)
- [Authors](#authors)
- [Acknowledgments](#acknowledgments)

---

## About

**minishell** is a project from the 42 School common core that challenges students to build a functional shell from scratch in C. The goal is to gain a deep understanding of process creation, file descriptors, signal handling, and the inner workings of a Unix command-line interpreter.

The shell reads user input, parses it into tokens and an abstract syntax tree, and executes commands by forking child processes. It supports pipes, input/output redirections, here-documents, environment variable expansion, and a set of built-in commands.

---

## Features

- **Interactive prompt** with input line editing via `readline`
- **Command execution** by searching `PATH` or using absolute/relative paths
- **Pipes** (`|`) to chain commands, connecting stdout of one process to stdin of the next
- **Input redirection** (`<`) to read from a file
- **Output redirection** (`>`) to write to a file (truncate)
- **Append redirection** (`>>`) to write to a file (append)
- **Here-document** (`<<`) to read input until a specified delimiter
- **Environment variable expansion** (`$VAR`, `$?` for the last exit status)
- **Quoting**
  - Single quotes (`'`) prevent interpretation of all metacharacters
  - Double quotes (`"`) prevent interpretation of all metacharacters except `$`
- **Signal handling**
  - `Ctrl-C` displays a new prompt on a new line
  - `Ctrl-D` exits the shell
  - `Ctrl-\` does nothing in interactive mode
- **Exit status** tracking via `$?`

---

## Built-in Commands

The following commands are implemented directly within the shell (not via external binaries):

| Command | Description |
|---------|-------------|
| `echo`  | Print arguments to standard output. Supports the `-n` flag to suppress the trailing newline. |
| `cd`    | Change the current working directory. Supports relative paths, absolute paths, `-` (previous directory), and no argument (home directory). Updates `PWD` and `OLDPWD`. |
| `pwd`   | Print the absolute pathname of the current working directory. |
| `export`| Set or display environment variables. Without arguments, prints all exported variables in declaration format. |
| `unset` | Remove environment variables from the environment. |
| `env`   | Print the current environment variables. |
| `exit`  | Exit the shell with an optional numeric exit status. |

---

## Getting Started

### Prerequisites

- **Operating System**: Linux or macOS
- **Compiler**: `cc` (gcc or clang) with C99 or later support
- **Libraries**: `readline` (development headers required)
- **Build tool**: `make`

On Debian/Ubuntu systems, install the readline development library:

```bash
sudo apt-get install libreadline-dev
```

On macOS with Homebrew:

```bash
brew install readline
```

### Installation

Clone the repository and build the project:

```bash
git clone https://github.com/Czarslayer/MINI_SHELL-42.git
cd MINI_SHELL-42
make
```

This produces the `minishell` executable in the project root.

### Usage

Launch the shell:

```bash
./minishell
```

You will be presented with an interactive prompt. Enter commands as you would in bash:

```
minishell$ echo "Hello, World!"
Hello, World!
minishell$ ls -la | grep .c | wc -l
12
minishell$ cat < input.txt > output.txt
minishell$ export MY_VAR="some value"
minishell$ echo $MY_VAR
some value
minishell$ exit
```

---

## Shell Grammar

The shell processes input through the following stages:

1. **Lexical Analysis (Tokenization)**
   - The input string is split into tokens: words, operators (`|`, `<`, `>`, `>>`, `<<`), and quoted strings.
   - Quoting rules are applied to determine token boundaries.

2. **Syntax Analysis (Parsing)**
   - Tokens are organized into an abstract syntax tree (AST) or equivalent data structure.
   - The parser validates the grammar and constructs a representation of pipelines and redirections.

3. **Expansion**
   - Environment variables (`$VAR`) are expanded to their values.
   - `$?` is replaced with the exit status of the most recently executed foreground pipeline.

4. **Execution**
   - Simple commands are executed by forking a child process and calling `execve`.
   - Pipelines are constructed by connecting processes via `pipe()` file descriptors.
   - Redirections are applied using `dup2()` before execution.
   - Built-in commands are executed in the current process (or a child, depending on context).

---

## Architecture

The shell follows a modular architecture with clear separation of concerns:

```
+-----------+     +--------+     +-----------+     +-----------+
|   Input   | --> | Lexer  | --> |  Parser   | --> | Executor  |
| (readline)|     | (tokens)|    | (AST/tree)|     | (fork/exec)|
+-----------+     +--------+     +-----------+     +-----------+
                                                        |
                                              +---------+---------+
                                              |                   |
                                         +----------+      +-----------+
                                         | Builtins |      |  External |
                                         |          |      |  Commands |
                                         +----------+      +-----------+
```

**Key modules:**

- **Lexer** -- Tokenizes raw input into a stream of typed tokens
- **Parser** -- Validates grammar and builds a command tree from tokens
- **Expander** -- Resolves environment variables and performs quote removal
- **Executor** -- Manages process creation, piping, redirections, and built-in dispatch
- **Builtins** -- Implements shell-native commands (`cd`, `echo`, `export`, etc.)
- **Signals** -- Configures signal handlers for interactive and non-interactive contexts
- **Environment** -- Manages the shell's environment variable store

---

## Project Structure

```
MINI_SHELL-42/
|-- Makefile              # Build configuration
|-- includes/             # Header files
|   |-- minishell.h       # Main header with struct definitions and prototypes
|-- libft/                # 42 libft library (string/memory utilities)
|   |-- Makefile
|   |-- libft.h
|   |-- ...
|-- src/                  # Source files
|   |-- main.c            # Entry point, main loop, signal setup
|   |-- lexer/            # Tokenization
|   |-- parser/           # Syntax analysis and tree construction
|   |-- expander/         # Variable expansion and quote removal
|   |-- executor/         # Command execution, pipes, redirections
|   |-- builtins/         # Built-in command implementations
|   |-- signals/          # Signal handler configuration
|   |-- env/              # Environment variable management
|   |-- utils/            # Shared utility functions
```

> **Note**: The directory layout above represents the intended project structure. It may evolve as development progresses.

---

## Technology Stack

| Category        | Technology                          |
|-----------------|-------------------------------------|
| Language        | C (C99)                             |
| Build System    | GNU Make                            |
| Line Editing    | GNU Readline                        |
| Compiler        | `cc` (gcc / clang)                  |
| Platform        | Linux, macOS                        |
| C Library       | libft (42 standard utility library) |

### Key System Calls

The following POSIX system calls form the core of the shell's execution engine:

- `fork()` -- Create child processes
- `execve()` -- Execute programs
- `pipe()` -- Create inter-process communication channels
- `dup2()` -- Redirect file descriptors
- `waitpid()` -- Wait for child process termination
- `open()` / `close()` -- File descriptor management
- `signal()` / `sigaction()` -- Signal disposition
- `getcwd()` / `chdir()` -- Working directory operations
- `readline()` / `add_history()` -- Interactive input with history

---

## Testing

### Manual Testing

Run the shell and compare behavior against `bash`:

```bash
# Launch minishell
./minishell

# Run the same commands in bash and compare output/exit codes
echo $?
```

### Edge Cases to Verify

- Empty input and whitespace-only input
- Unclosed quotes (should display an error, not hang)
- Invalid redirections (`> | `, `< < `, `>>>`  )
- Multiple pipes (`cmd1 | cmd2 | cmd3 | cmd4`)
- Nested quotes and variable expansion within double quotes
- Signal behavior during command execution vs. at the prompt
- Here-document with and without quoted delimiters
- `$?` after various built-in and external commands
- Environment inheritance by child processes

### Third-Party Test Suites

Several community-developed test frameworks exist for minishell:

- [minishell-tester](https://github.com/LucasKuber/minishell_tester) -- Automated comparison against bash
- [42-minishell-tester](https://github.com/zstenger93/minishell_tester) -- Comprehensive test coverage

---

## References

- [The Open Group Base Specifications -- Shell Command Language](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html)
- [GNU Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html)
- [GNU Readline Library Documentation](https://tiswww.case.edu/php/chet/readline/rltop.html)
- [42 School](https://42.fr/en/homepage/)

---

## Authors

- **Mourad Abahani** -- [@Czarslayer](https://github.com/Czarslayer)

---

## Acknowledgments

- [42 Network](https://www.42network.org/) and [1337 Coding School](https://1337.ma/) for the project subject and learning environment
- The open-source community for test suites and reference implementations
- The GNU Project for the `bash` and `readline` documentation that served as behavioral references
