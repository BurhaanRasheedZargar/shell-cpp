# shell-cpp

A custom Unix-like shell implemented in C++ that supports command parsing, built-in functions, external program execution, piping, I/O redirection, and advanced terminal features like auto-completion and history management.

## Features

### 1. Complex Command Parsing
The shell uses a custom tokenizer to handle various argument formats:
* **Single Quotes (`'...'`)**: Preserves every character literally.
    * *Example*: `$ echo 'plain text'` prints `plain text`.
* **Double Quotes (`"..."`)**: Preserves literal values but allows escaping for `"` and `\`.
    * *Example*: `$ echo "He said \"Hi\""` prints `He said "Hi"`.
* **Backslash Escapes (`\`)**: Escapes the next character.
    * *Example*: `$ echo Space\ Between` prints `Space Between`.

### 2. Built-in Commands
The shell natively executes several standard commands:
* **`echo`**: Displays text to the terminal.
    * *Example*: `$ echo hello world`
* **`cd`**: Changes the current working directory. It supports the `~` shortcut for the home directory.
    * *Example*: `$ cd ~/Documents`
* **`pwd`**: Prints the absolute path of the current directory.
    * *Example*: `$ pwd`
* **`type`**: Identifies if a command is a shell builtin or an external executable.
    * *Example*: `$ type ls` outputs `ls is /usr/bin/ls`.
* **`history`**: Displays or manages the command history.
    * *Example*: `$ history 5` shows the last 5 commands.
* **`exit`**: Terminates the shell, optionally with a specific exit code.
    * *Example*: `$ exit 0`

### 3. External Executables
The shell searches the system's `PATH` environment variable to find and run external programs.
* *Example*: `$ ls -la` or `$ cat file.txt`

### 4. Command Pipelines
You can chain multiple commands using the pipe (`|`) symbol. The standard output of one command is passed as standard input to the next.
* *Example*: `$ cat main.cpp | grep "int" | wc -l`

### 5. I/O Redirection
The shell supports redirecting standard output and standard error to files:
* **Standard Output (Overwrite)**: `$ command > file.txt` or `$ command 1> file.txt`.
* **Standard Output (Append)**: `$ command >> file.txt` or `$ command 1>> file.txt`.
* **Standard Error (Overwrite)**: `$ command 2> error.log`.
* **Standard Error (Append)**: `$ command 2>> error.log`.

### 6. Auto-completion (Tab Completion)
Integrated with the GNU Readline library, the shell provides intelligent tab-completion for built-in commands and external binaries found in your `PATH`.
* *Usage*: Type `p` and press `Tab` twice to see options like `pwd`.

### 7. Persistent History
The shell reads from and writes to a history file (specified by the `HISTFILE` environment variable), allowing you to access previous commands across sessions.

---

## Workflow

1.  **Prompt**: The shell displays a `$ ` prompt and waits for input via `readline()`.
2.  **Parsing**: The input string is broken down into tokens, respecting quotes and escape characters.
3.  **Pipeline Detection**: If pipes (`|`) are present, the shell creates necessary pipes and forks multiple child processes.
4.  **Execution**:
    * **Builtins**: Executed directly within the shell process unless part of a pipeline.
    * **External**: The shell forks a child process, sets up redirection/pipes, and uses `execv()` to run the program.
5.  **Clean up**: The shell waits for all processes to finish and resets file descriptors before the next prompt.

## Compilation

### Prerequisites
* C++17 compatible compiler (e.g., `g++`).
* GNU Readline library (`libreadline-dev` on Linux).

### Build Command
```bash
g++ -std=c++17 main.cpp -lreadline -o shell-cpp
