---
layout: post
title:  "Refactoring and Extending a VSCode Extension for Cyclomatic Complexity"
date:   2023-05-14 20:54:00 +0800
categories: vscode-extension cyclomatic-complexity c-language refactoring clang
---

I continuously strive to make my code more maintainable and user-friendly. This commitment led me to refine one of my recent projects, a Visual Studio Code extension that calculates Cyclomatic Complexity of C functions, found at [this GitHub repository](https://github.com/ChinChangYang/cyclomatic-complexity-c). 

This extension has been designed to enhance the readability and maintainability of C code by providing real-time Cyclomatic Complexity calculation. However, during my continuous pursuit of excellence, I identified a few areas that required improvements to increase the usability and robustness of this extension. Here's a walkthrough of the problems I encountered and how I resolved them to create a better user experience.

## The Challenges

The initial challenge was the lack of clarity in the installation instructions for the `ccc` program, an integral part of this extension that performs the Cyclomatic Complexity calculations. The building and installation process of the `ccc` program was not adequately explained, which could lead to potential confusion for users.

In addition to this, the code within the `extension.ts` file, particularly the `provideCodeLenses` function, was a little lengthy and complex. This function was responsible for running the `ccc` program and creating CodeLens instances for each function in the user's C code. To adhere to best coding practices and enhance maintainability, it was crucial to break down this function into smaller, more manageable pieces.

## The Solutions

To make the installation process of the `ccc` program more straightforward, I decided to update the `README.md` file with step-by-step instructions. These detailed guidelines would ensure that users could seamlessly build and install the `ccc` program, thereby improving the user experience of the extension.

As for the `extension.ts` file, the `provideCodeLenses` function was refactored into two separate functions: `getScriptOutput` and `createCodeLensesFromOutput`. The `getScriptOutput` function handled the execution of the `ccc` program and returned its output, while `createCodeLensesFromOutput` parsed this output and created CodeLens instances for each function. This separation of concerns made the code more readable and maintainable.

I also added error handling to the `provideCodeLenses` function. If the `ccc` program wasn't found, the function would now return an empty array instead of throwing an error, preventing the extension from breaking down. A message was also displayed to the user, guiding them to the installation instructions in the `README.md` file.

## The Result

After these changes, the extension is now more robust and user-friendly. The `README.md` file provides clear instructions for the installation of the `ccc` program, and the refactored code within the `extension.ts` file is more readable and maintainable.

This journey through refining the extension was a valuable experience that reinforced the importance of clear documentation and clean, maintainable code. I'm confident that these improvements will provide a better user experience and make the extension even more useful for C developers.

Feel free to explore the improved extension at the [GitHub repository](https://github.com/ChinChangYang/cyclomatic-complexity-c) and share your feedback or contributions.

As we continue to grow as a community, let's strive to make our code more accessible and our tools more user-friendly, one line at a time.
