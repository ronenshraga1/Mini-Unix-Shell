# Mini Unix Shell

A small C Mini Unix Shell focused on Unix process management. The main program, `myshell`, implements an interactive shell that parses command lines, starts foreground and background processes, tracks process state, supports basic redirection and one pipeline, and exposes built-in commands for history and signal-based process control.

## Features

- Interactive shell prompt that prints the current working directory.
- External command execution with `fork`, `execvp`, and `waitpid`.
- Foreground and background execution using `&`.
- Input and output redirection with `<` and `>`.
- Single pipeline support, for example `cmd1 | cmd2`.
- Command history with a fixed size of 10 entries.
- Process list tracking with running, suspended, and terminated states.
- Built-in process control commands based on Unix signals.
- Debug mode that prints child PID, command name, and execution mode.
- Standalone examples for pipes, process groups, and signal handling.

## Tech Stack

- Language: C
- Platform: Linux / Unix-like environment
- Build tool: `make`
- Compiler: `gcc`
- Target mode: 32-bit compilation through `gcc -m32`
- Development container: Ubuntu 20.04 with multilib GCC, GDB, Valgrind, NASM, Make, and common CLI tools

## Project Structure

```text
.
|-- .devcontainer/
|   |-- devcontainer.json    # VS Code dev container configuration
|   `-- dockerfile           # Ubuntu 20.04 32-bit-capable toolchain image
|-- LineParser.c             # Command-line parser implementation
|-- LineParser.h             # Parser data structures and function declarations
|-- myshell.c                # Main interactive shell implementation
|-- mypipeline.c             # Demonstration of a hard-coded pipe: ps -xl | grep 5
|-- mypipe.c                 # Simple parent/child pipe example
|-- looper.c                 # Infinite loop program with signal handlers
|-- Printers.c               # Process-group example that prints alternating numbers
|-- makefile                 # Build and clean commands
|-- in.txt                   # Sample input file
|-- out.txt                  # Sample output file
`-- README.md
```

Generated build artifacts such as `*.o`, `myshell`, `mypipeline`, `mypipe`, `looper`, and related binaries are ignored by `.gitignore`.

## Architecture

`myshell.c` is the main executable. It reads user input, uses `LineParser.c` to convert the input into a linked list of `cmdLine` structures, then dispatches either built-in shell commands or external programs.

The shell keeps two in-memory data structures:

- A process list that stores parsed command lines, process IDs, and process statuses.
- A circular history queue that stores the last 10 executed commands.

External command execution is handled with `fork()` and `execvp()`. Foreground commands are waited on immediately; background commands are added to the process list and checked later with `waitpid(..., WNOHANG | WUNTRACED | WCONTINUED)`.

## Installation

### Prerequisites

Use a Linux environment with:

- `gcc`
- `make`
- 32-bit development libraries compatible with `gcc -m32`

On Debian/Ubuntu, the required packages are similar to those installed by the dev container:

```sh
sudo apt-get update
sudo apt-get install build-essential gcc-multilib g++-multilib libc6-dev-i386 make
```

### Build

```sh
make
```

The makefile builds:

- `myshell`
- `mypipeline`

To remove generated build outputs:

```sh
make clean
```

## Configuration

No runtime environment variables are required by the programs.

The dev container sets:

```sh
CFLAGS=-m32
LDFLAGS=-m32
```

The makefile also passes `-m32` directly to `gcc`.

## Usage

Run the shell:

```sh
./myshell
```

Run the shell in debug mode:

```sh
./myshell -d
```

Example shell commands:

```sh
ls -l
sleep 10 &
cat < in.txt
echo hello > out.txt
ps -xl | grep 5
history
procs
quit
```

### Built-in Commands

| Command | Description |
| --- | --- |
| `cd <path>` | Change the shell's current working directory. |
| `history` | Print the command history. |
| `!!` | Re-run the most recent history entry. |
| `!<number>` | Re-run a command by its history number. |
| `procs` | Print tracked processes and their statuses. |
| `stop <pid>` | Send `SIGSTOP` to a process and mark it suspended. |
| `wakeup <pid>` | Send `SIGCONT` to a process and mark it running. |
| `ice <pid>` | Send `SIGINT` to a process and mark it terminated. |
| `nuke <pid>` | Send `SIGKILL` to the process group identified by `pid`. |
| `quit` | Free shell resources and exit. |

### Pipeline Demo

`mypipeline` runs a hard-coded two-process pipeline equivalent to:

```sh
ps -xl | grep 5
```

Run it with:

```sh
./mypipeline
```

### Additional Example Programs

`mypipe.c`, `looper.c`, and `Printers.c` are source examples, but they are not built by the current makefile.

To compile one manually:

```sh
gcc -m32 -Wall -g -o mypipe mypipe.c
gcc -m32 -Wall -g -o looper looper.c
gcc -m32 -Wall -g -o Printers Printers.c
```

## Scripts and Commands

| Command | Purpose |
| --- | --- |
| `make` | Build `myshell` and `mypipeline`. |
| `make clean` | Remove `myshell`, `mypipeline`, and object files. |
| `./myshell` | Start the interactive shell. |
| `./myshell -d` | Start the shell with debug output. |
| `./mypipeline` | Run the pipe demonstration program. |


## Testing

TODO: No automated test suite is present in the repository.

Suggested manual checks:

```sh
make
./myshell
```

Inside `myshell`, try:

```sh
pwd
echo hello
echo hello > out.txt
cat < out.txt
sleep 5 &
procs
history
ps -xl | grep 5
quit
```

## Deployment

There is no deployment pipeline or production deployment target. The repository is intended to be built and run locally or inside the provided VS Code dev container.

## Development Container

The `.devcontainer` setup builds an Ubuntu 20.04 image with the required 32-bit C toolchain and debugging utilities. Open the repository in VS Code with the Dev Containers extension and choose **Reopen in Container**.

The container validates the toolchain after creation by printing versions for GCC, Make, GDB, Valgrind, and NASM, then compiling a small 32-bit test program.

## Contributing

1. Keep changes focused on the lab programs and parser utilities.
2. Build with `make` before submitting changes.
3. Update this README when adding new commands, source files, build targets, or tests.
4. Do not commit generated binaries or object files.
