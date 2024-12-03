# Development Tools Demonstrations

## Git Version Control

### Basic Git Workflow
```bash
# Initialize a new repository
git init

# Add files and make first commit
git add README.md
git commit -m "Initial commit"

# Create and switch to a feature branch
git checkout -b feature/new-feature

# Make changes and commit
git add changed_file.cpp
git commit -m "Implement new feature"

# Push changes to remote
git push origin feature/new-feature
```

### Advanced Git Features
```bash
# Interactive rebase to clean up commits
git rebase -i HEAD~3

# Cherry-pick a specific commit
git cherry-pick abc123def

# Create and apply patches
git format-patch -1 HEAD
git am < patch_file.patch
```

## GDB Debugging

### Basic Debugging Session
```bash
# Compile with debugging symbols
g++ -g program.cpp -o program

# Start GDB
gdb ./program

# Common GDB commands
(gdb) break main
(gdb) run
(gdb) next
(gdb) step
(gdb) print variable
(gdb) backtrace
```

### Advanced GDB Features
```bash
# Set watchpoint
(gdb) watch variable

# Examine memory
(gdb) x/10x &variable

# Custom pretty printers
(gdb) info pretty-printer
```

## Valgrind Memory Analysis

### Memory Leak Detection
```bash
# Basic memory check
valgrind --leak-check=full ./program

# Detailed memory analysis
valgrind --tool=memcheck --leak-check=full --show-leak-kinds=all ./program
```

### Cache Profiling
```bash
# Cache usage analysis
valgrind --tool=cachegrind ./program

# Analyze results
cg_annotate cachegrind.out.12345
```

## LLDB (Alternative Debugger)

### Basic LLDB Usage
```bash
# Start debugging session
lldb ./program

# Common commands
(lldb) breakpoint set --name main
(lldb) run
(lldb) thread step-over
(lldb) thread step-in
(lldb) frame variable
```

## Compiler Tools

### Clang Static Analyzer
```bash
# Run analysis
scan-build g++ program.cpp

# Generate HTML report
scan-build -V g++ program.cpp
```

### Address Sanitizer
```bash
# Compile with sanitizer
g++ -fsanitize=address program.cpp

# Run with sanitizer
./a.out
```

## Build Systems

### Make
```makefile
# Example Makefile
CXX = g++
CXXFLAGS = -Wall -g

program: main.o utils.o
    $(CXX) $(CXXFLAGS) -o program main.o utils.o

clean:
    rm -f *.o program
```

### CMake
```cmake
# Example CMakeLists.txt
cmake_minimum_required(VERSION 3.10)
project(MyProject)

add_executable(program main.cpp utils.cpp)
target_compile_features(program PRIVATE cxx_std_17)
```

## IDE Features

### VSCode Integration
```json
// Example launch.json for debugging
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Program",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/program",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb"
        }
    ]
}
```

## Performance Profiling

### perf
```bash
# Record performance data
perf record ./program

# Analyze results
perf report
```

### gprof
```bash
# Compile with profiling
g++ -pg program.cpp

# Run program
./a.out

# Analyze profile data
gprof ./a.out gmon.out
```

Each of these demonstrations shows practical usage of essential development tools you'll need when working with Bitcoin Core. Try them out in your local environment to get hands-on experience. 