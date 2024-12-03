# Development Tools Exercises

## Git Version Control

### Exercise 1: Basic Git Workflow
1. Create a new directory and initialize it as a Git repository
2. Create a README.md file with a project description
3. Make your first commit
4. Create a new branch called `feature/calculator`
5. Add a simple calculator program in C++
6. Commit your changes with a meaningful message
7. Switch back to main branch and merge your feature branch

### Exercise 2: Conflict Resolution
1. Create a new branch called `feature/add-multiply`
2. Modify the calculator to add multiplication
3. Switch to main and create another branch `feature/add-division`
4. Modify the same file to add division
5. Try to merge both branches and resolve the conflicts

### Exercise 3: Git History Management
1. Make several commits on your feature branch
2. Use interactive rebase to squash commits
3. Write a good commit message that summarizes all changes
4. Create a patch from your changes
5. Apply the patch to a different branch

## GDB Debugging

### Exercise 1: Basic Debugging
```cpp
// Debug this program using GDB
#include <iostream>

int factorial(int n) {
    if (n < 0) return -1;
    if (n == 0) return 1;
    return n * factorial(n - 1);
}

int main() {
    int result = factorial(5);
    std::cout << "Factorial: " << result << std::endl;
    return 0;
}
```
1. Set a breakpoint at the factorial function
2. Step through the recursion
3. Print variable values at each step
4. Use backtrace to understand the call stack

### Exercise 2: Memory Bug Detection
```cpp
// Find and fix the memory bug using GDB
#include <iostream>

int main() {
    int* arr = new int[5];
    for (int i = 0; i <= 5; i++) {
        arr[i] = i;
    }
    delete[] arr;
    return 0;
}
```

## Valgrind Memory Analysis

### Exercise 1: Memory Leak Detection
```cpp
// Find and fix all memory leaks
#include <iostream>

class Resource {
    int* data;
public:
    Resource() { data = new int[100]; }
    void process() { /* some work */ }
};

int main() {
    Resource* r1 = new Resource();
    Resource* r2 = new Resource();
    r1->process();
    return 0;
}
```

### Exercise 2: Cache Performance
1. Write a matrix multiplication program
2. Use Valgrind's cachegrind to analyze cache performance
3. Optimize the program based on the analysis
4. Compare performance before and after optimization

## Build System Practice

### Exercise 1: Makefile Creation
1. Create a project with the following structure:
```
project/
├── src/
│   ├── main.cpp
│   ├── calculator.cpp
│   └── calculator.h
└── Makefile
```
2. Write a Makefile that:
   - Compiles with debugging symbols
   - Uses compiler warnings
   - Has clean and install targets
   - Supports different build types (Debug/Release)

### Exercise 2: CMake Configuration
1. Convert the above project to use CMake
2. Add external dependency (e.g., Boost)
3. Configure different build types
4. Add custom build targets

## IDE Integration

### Exercise 1: VSCode Setup
1. Set up a C++ project in VSCode
2. Configure:
   - IntelliSense
   - Build tasks
   - Debug configuration
   - Code formatting
3. Add useful extensions for C++ development

### Exercise 2: Debugging Configuration
1. Create launch configurations for:
   - Debug build
   - Release build
   - Running with arguments
2. Add compound launch configurations
3. Configure task dependencies

## Performance Analysis

### Exercise 1: Profiling Practice
```cpp
// Profile and optimize this code
#include <vector>
#include <algorithm>

void inefficientSort(std::vector<int>& vec) {
    for (size_t i = 0; i < vec.size(); i++) {
        for (size_t j = i + 1; j < vec.size(); j++) {
            if (vec[j] < vec[i]) {
                std::swap(vec[i], vec[j]);
            }
        }
    }
}

int main() {
    std::vector<int> numbers(10000);
    // Fill with random numbers
    for (int i = 0; i < 10000; i++) {
        numbers[i] = rand();
    }
    inefficientSort(numbers);
    return 0;
}
```
1. Use gprof to profile the program
2. Identify performance bottlenecks
3. Optimize the code
4. Compare performance metrics

### Exercise 2: Memory Profiling
1. Write a program that processes large datasets
2. Profile memory usage with Valgrind
3. Optimize memory usage
4. Document improvements

## Challenge Exercise: Complete Development Workflow

Create a small project that demonstrates your mastery of all tools:

1. Initialize a Git repository
2. Set up a CMake build system
3. Implement a feature with intentional bugs
4. Use GDB to find and fix the bugs
5. Use Valgrind to check for memory issues
6. Profile the code and optimize performance
7. Document your process and findings

Submit your solution by:
1. Creating a new branch
2. Adding your implementation
3. Creating a detailed commit message
4. Preparing a patch file

This comprehensive exercise will help you practice using all development tools in a realistic scenario. 