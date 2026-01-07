Custom Shell Implementation
A feature-rich Unix shell implementation in C++ with command completion, history management, pipelines, and I/O redirection.
Libraries Used
Standard C++ Libraries

<bits/stdc++.h> - Convenience header including most standard libraries
<filesystem> - File system operations (path manipulation, file existence checks)
<optional> - Optional return values for functions that may fail
<vector>, <string>, <algorithm> - Standard containers and algorithms

POSIX System Libraries

<unistd.h> - POSIX operating system API (fork, exec, dup2, close)
<sys/wait.h> - Process wait operations (waitpid)
<fcntl.h> - File control operations (open, O_WRONLY, O_CREAT flags)

GNU Readline Library

<readline/readline.h> - Interactive line editing with command history
<readline/history.h> - Command history management and persistence

Features
1. Built-in Commands
echo
Prints arguments to stdout with space separation.
bash$ echo Hello World
Hello World
pwd
Prints the current working directory.
bash$ pwd
/home/user/projects
cd
Changes the current directory. Supports ~ for home directory.
bash$ cd /tmp
$ cd ~
$ cd Documents
type
Identifies if a command is a builtin or external program.
bash$ type echo
echo is a shell builtin
$ type ls
ls is /usr/bin/ls
$ type nonexistent
nonexistent: not found
history
Displays or manages command history.
bash$ history          # Show all history
$ history 10       # Show last 10 commands
$ history -w file  # Write history to file
$ history -r file  # Read history from file
$ history -a file  # Append new commands to file
exit
Exits the shell with optional exit code.
bash$ exit
$ exit 42
2. External Command Execution
The shell searches for executables in directories listed in the PATH environment variable.
bash$ ls -la
$ cat file.txt
$ grep "pattern" file.txt
3. Tab Completion
Intelligent command completion system with the following behavior:

Single Tab Press:

Completes unique matches automatically
Completes to longest common prefix if multiple matches
Beeps if ambiguous


Double Tab Press:

Shows all possible completions
Redisplays current input



bash$ ec<TAB>          # Completes to "echo"
$ e<TAB><TAB>      # Shows: echo env exit ...
Completion sources:

Built-in commands
Executables in PATH directories

4. Command Pipelines
Chains multiple commands where stdout of one feeds into stdin of the next.
bash$ cat file.txt | grep "error" | wc -l
$ ls -l | sort | head -n 5
Implementation Details:

Creates pipes between consecutive commands
Forks separate processes for each command
Properly handles file descriptor management
Waits for all processes to complete

5. I/O Redirection
Standard Output Redirection
bash$ echo "Hello" > output.txt        # Truncate and write
$ echo "World" >> output.txt       # Append
$ echo "Explicit" 1> output.txt    # Explicit stdout
$ echo "Append" 1>> output.txt     # Explicit append
Standard Error Redirection
bash$ command 2> errors.txt            # Redirect stderr
$ command 2>> errors.txt           # Append stderr
Combined Redirection
bash$ command > output.txt 2> errors.txt
$ command >> out.txt 2>> err.txt
6. Quote Handling
Supports single quotes, double quotes, and escape sequences.
bash$ echo 'Single quotes preserve literals'
$ echo "Double quotes allow $variables"
$ echo "Escaped \"quotes\" inside"
$ echo 'Can'\''t without escaping'
$ echo Path\ with\ spaces
7. History Persistence
Automatically saves and loads command history using the HISTFILE environment variable.
bashexport HISTFILE=~/.shell_history

History persists across sessions
Incremental appending with history -a
Full write on exit

Code Workflow
1. Initialization Phase
cppint main() {
    rl_attempted_completion_function = completion;
    rl_completion_append_character = ' ';
    
    // Load history if HISTFILE is set
    const char *histfile = getenv("HISTFILE");
    if (histfile && *histfile) {
        read_history(histfile);
    }
    
    // Main loop starts...
}
2. Input Processing Loop
┌─────────────────────────────────────┐
│   readline("$ ") - Get user input   │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│  parse_command_line() - Tokenize    │
│  Handles quotes and escape sequences│
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│   Split by "|" for pipelines        │
└──────────────┬──────────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
   Pipeline?       Single Command
        │             │
        │             ▼
        │     ┌───────────────┐
        │     │ Parse I/O     │
        │     │ redirections  │
        │     └───────┬───────┘
        │             │
        │      ┌──────┴─────┐
        │      │            │
        ▼      ▼            ▼
  handle_    Builtin?   External
  pipeline_n            Command
        │      │            │
        └──────┴────────────┘
               │
               ▼
        Execute & Wait
3. Command Parsing
The parse_command_line() function tokenizes input while respecting:

Single quotes (literal strings)
Double quotes (allows escapes)
Backslash escapes
Whitespace separation

cppvector<string> tokens = parse_command_line(line);
// Input:  echo "Hello World" 'test' arg\ with\ space
// Output: ["echo", "Hello World", "test", "arg with space"]
4. Pipeline Execution
cppvoid handle_pipeline_n(const vector<vector<string>> &pipeline) {
    // For each command in pipeline:
    // 1. Create pipe (except for last command)
    // 2. Fork child process
    // 3. Setup stdin from previous pipe
    // 4. Setup stdout to next pipe
    // 5. Execute command
    // 6. Parent closes unused pipe ends
    // 7. Wait for all children
}
Example: cat file.txt | grep "error" | wc -l
Process 1: cat file.txt
   stdout → pipe1
   
Process 2: grep "error"
   stdin  ← pipe1
   stdout → pipe2
   
Process 3: wc -l
   stdin  ← pipe2
   stdout → terminal
5. External Command Execution
cppvoid execute_external(...) {
    pid_t pid = fork();
    if (pid == 0) {  // Child process
        // Setup I/O redirections
        if (redirect_stdout) {
            int fd = open(output_file, flags, 0644);
            dup2(fd, STDOUT_FILENO);
        }
        
        // Execute program
        execv(path, args);
    } else {  // Parent process
        waitpid(pid, &status, 0);
    }
}
6. Path Resolution
cppoptional<string> find_in_path(const string &cmd) {
    // Get PATH environment variable
    // Split by ':' (or ';' on Windows)
    // For each directory:
    //   - Check if cmd exists
    //   - Check if executable
    //   - Return full path if found
    // Return nullopt if not found
}
7. Tab Completion System
cppchar **completion(const char *text, int start, int end) {
    // Only complete at start of line (command position)
    if (start != 0) return nullptr;
    
    // Gather matches from:
    // 1. builtin_generator() - built-in commands
    // 2. external_generator() - PATH executables
    
    // Single match: return it
    // Multiple matches:
    //   - First tab: complete to LCP, beep if ambiguous
    //   - Second tab: show all matches
}
8. I/O Redirection for Builtins
Built-in commands handle redirection by:

Saving original file descriptors
Opening target files with appropriate flags
Duplicating to stdout/stderr
Executing command
Restoring original file descriptors

cppint saved_stdout = dup(STDOUT_FILENO);
int fd = open(output_file, O_WRONLY | O_CREAT | O_TRUNC, 0644);
dup2(fd, STDOUT_FILENO);

// Execute builtin command

dup2(saved_stdout, STDOUT_FILENO);
close(saved_stdout);
Usage Examples
Basic Commands
bash$ pwd
/home/user

$ cd /tmp
$ pwd
/tmp

$ echo Testing shell features
Testing shell features
Pipeline Example
bash$ cat access.log | grep "ERROR" | wc -l
42
Redirection Example
bash$ echo "Log entry" > log.txt
$ cat log.txt
Log entry

$ echo "Another entry" >> log.txt
$ cat log.txt
Log entry
Another entry
History Management
bash$ export HISTFILE=~/.myshell_history
$ echo "command 1"
$ echo "command 2"
$ history 5
    1  export HISTFILE=~/.myshell_history
    2  echo "command 1"
    3  echo "command 2"
    4  history 5

$ history -w backup.hist  # Save to file
$ history -a backup.hist  # Append new commands
Complex Example
bash$ cat data.txt | sort | uniq | grep "pattern" > results.txt 2> errors.txt
This command:

Reads data.txt
Sorts the lines
Removes duplicates
Filters for "pattern"
Writes matches to results.txt
Writes errors to errors.txt

Compilation
bashg++ -std=c++17 main.cpp -o shell -lreadline
Required flags:

-std=c++17: For filesystem library
-lreadline: Link GNU readline library

Environment Variables

PATH: Colon-separated directories to search for executables
HISTFILE: File path for persistent command history
HOME: Home directory for cd ~

Technical Notes
Memory Management

Uses strdup() for readline completions (caller must free)
Properly closes file descriptors after use
Waits for all child processes to prevent zombies

Process Model

Parent shell process manages lifecycle
Each external command runs in forked child
Pipeline commands run concurrently with proper synchronization

Error Handling

Uses error_code for filesystem operations
perror() for system call failures
Graceful fallback for missing commands

Limitations

No background job control (&)
No command substitution ($(...))
No shell variables or parameter expansion
No glob pattern expansion (*.txt)
Limited quote handling in completion
No signal handling (Ctrl+C, Ctrl+Z)

Future Enhancements

Job control (background processes, fg/bg commands)
Shell scripting support (if/while/for loops)
Environment variable expansion
Glob pattern matching
Command aliases
Signal handling and process groups
Configuration file support (~/.shellrc)
