---
layout: post
title:  "Reducing Dependency and Improving Build Process in a VSCode Extension"
date:   2023-05-13 20:28:00 +0800
categories: github vs-code-extension cyclomatic-complexity c-parser clang
---

In the process of maintaining and improving a VSCode extension for calculating cyclomatic complexity in C code, I've encountered and resolved several issues. This blog post will walk you through these challenges and how they were addressed, providing insights into the journey of refining the extension.

## Problem: Dependency on an External Python Script

The initial implementation of the extension relied on an external Python script to calculate cyclomatic complexity. This dependency posed problems as it required users to have Python installed and set up correctly on their systems, which could potentially create barriers for users.

## Solution: Shifting to a Precompiled Binary

To remove the Python dependency, I decided to replace the Python script with a precompiled binary. This would allow the complexity calculation to be executed directly without needing Python. The replacement was a C++ program that used Clang's AST (Abstract Syntax Tree) traversal, which is better suited for parsing C code than regex-based solutions.

## Problem: Build Issues with the C++ Program

Upon embarking on creating the C++ program, I encountered various build issues. These issues posed a significant challenge and added complexity to the development process. I'm on MacOS, and used Homebrew to install LLVM. Despite this, I faced issues in accessing `llvm-config`.

## Solution: Resolving Build Issues

Firstly, to ensure that `llvm-config` was accessible in my PATH, I ran the following command:

```bash
echo 'export PATH="/usr/local/opt/llvm/bin:$PATH"' >> ~/.zshrc
```

After sourcing the updated bash profile, I was able to use `llvm-config` in my terminal.

The build issues were resolved by executing the command:

```bash
g++ ccc.cpp -o ccc $(llvm-config --cxxflags) $(llvm-config --ldflags) -lclang $(llvm-config --libs --system-libs)
```

This command uses `llvm-config` to get the appropriate flags and libraries required to compile the C++ program with Clang libraries.

## Problem: Synchronous Execution in the Extension

The extension was originally executing the Python script synchronously. This could potentially block the main thread, leading to an unresponsive VSCode editor.

## Solution: Asynchronous Execution

To improve this, I updated the extension to execute the `ccc` binary asynchronously. This ensures that the VSCode editor remains responsive even when the cyclomatic complexity is being calculated.

## Problem: Lack of Error Handling

The initial implementation had inadequate error handling for the complexity calculation process. This could lead to silent failures, which are problematic to debug and provide a poor user experience.

## Solution: Improved Error Handling

I enhanced the error handling mechanism to capture and log error events from the `ccc` command execution. If the command exits with a non-zero code, the promise is rejected with an informative error message, thereby improving the robustness of the extension.

## Final Thoughts

With these enhancements, the VSCode extension now calculates cyclomatic complexity for C code accurately and efficiently, without requiring Python. The changes have improved the build process, the execution of the complexity calculation, and the overall user experience. This journey, although filled with challenges, was a great learning experience and led to substantial improvements in the extension's functionality and usability.
