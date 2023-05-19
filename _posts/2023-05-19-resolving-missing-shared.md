---
layout: post
title:  "Resolving a Missing Shared Library Issue in a Compiled Binary on Linux"
date:   2023-05-19 20:17:00 +0800
categories: cross-platform-build clang libclang dependency-management llvm
---

Recently, while working on a project, I came across an interesting issue that I thought would be worthwhile to share with you all. 

The project involved a C++ program named `ccc`, which I was building in various environments using GitHub Actions. I had set up a workflow to build this program automatically on three different operating systems: Windows, Linux, and macOS. After overcoming a few hurdles, such as shell scripting syntax errors and compatibility issues with `llvm-config`, I was able to successfully build my program in these environments and automatically release the compiled binary using GitHub Actions.

However, things took a turn when I tried to run the compiled binary on a different Linux machine. As soon as I ran it, I was greeted with this error message: 

```
.cyclomatic-complexity-c/ccc: error while loading shared libraries: libclang-14.so.13: cannot open shared object file: No such file or directory
```

### Analyzing the Problem

The problem was clear - the compiled binary was trying to load a shared library, `libclang-14.so.13`, which was not present on the Linux machine. 

Typically, a compiled binary would dynamically link to shared libraries on the system it's being run on. This process allows for a more efficient use of system resources as multiple programs can share the same library without having it loaded into memory multiple times. But in this case, the shared library our program depended on (`libclang`) was not installed on the system.

### Looking for Solutions

Initially, I thought of using static linking to bundle all necessary libraries with the binary. However, after further consideration and an attempt, this solution proved to be unworkable.

This left me with two possible solutions:

1. Build `libclang` from source
2. Find a pre-compiled version of `libclang` compatible with the system.

Building from source would ensure compatibility, but it could take a lot of time and resources. On the other hand, finding a pre-compiled version would be faster and easier, but there might be compatibility issues. 

After evaluating these options, I decided to go with building `libclang` from source.

### Building libclang from Source

Despite the fact that building from source could be time-consuming, it provided the best chance of compatibility. The process involved the following steps:

1. First, I downloaded the source code of `libclang` from the LLVM website.
2. Then, I configured the build system using `cmake`, specifying the `clang` and `lld` subprojects with `-DLLVM_ENABLE_PROJECTS`, and setting the build type to "Release".
3. Finally, I used `ninja` to build the source code and installed it to a directory in the home directory, where I had the necessary permissions.

Upon completing these steps, I was able to successfully build `libclang` from source. I then ran the `ccc` program again and this time it executed without any issues.

### Conclusion

This endeavor provided a valuable lesson in understanding the intricacies of software distribution and dependency management. It emphasized that while tools can streamline many processes, a sound comprehension of underlying principles is indispensable. Through overcoming the challenge of a missing shared library, we saw first-hand the importance of dynamic and static linking, and the compatibility issues that can arise when deploying binaries across different systems.
