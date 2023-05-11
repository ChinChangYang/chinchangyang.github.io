---
layout: post
title:  "Developing a Cyclomatic Complexity Analyzer for C in VS Code"
date:   2023-05-11 23:16:00 +0800
categories: github vs-code-extension cyclomatic-complexity c-parser clang
---
# Cyclomatic Complexity Analyzer for C in VS Code

## Introduction

During my recent project, I developed a Visual Studio Code extension that computes the cyclomatic complexity of C functions. Cyclomatic complexity is a software metric that provides a quantitative measure of the complexity of a program. It directly measures the number of linearly independent paths through a program's source code.

## The Journey

### 1. Initial Development

Initially, I started with the base structure of a VS Code extension, with the idea to use CodeLens to display the complexity of each function. I wrote an extension activation function and registered a CodeLens provider for C files.

### 2. Integrating C Parser

Then came the challenge of parsing the C code. At first, I considered writing my own parser, but soon realized that it would be a major undertaking. I then decided to use an existing parser, and Clang was chosen for its robustness and popularity.

### 3. Cyclomatic Complexity Computation

With the parser in hand, I then implemented the cyclomatic complexity computation. According to McCabe's definition, the cyclomatic complexity M is defined as:

M = E - N + 2 * P

where:
- E = the number of edges in the graph
- N = the number of nodes in the graph
- P = the number of connected components

However, when I ran my first tests, the computed complexities were off. 

### 4. Debugging and Fixes

Through debugging, I realized I was miscounting the number of edges and nodes. After reviewing the McCabe's definition, I found the issue was that I was not properly handling the control flow constructs. I was not incrementing the edge count by 2 for decision nodes like 'if', 'for', 'while', 'default' and 'case'. After fixing the counts and handling nested 'if' statements correctly, the cyclomatic complexities computed matched the expected values.

### 5. Finishing Touches

Finally, I refactored the code for better readability and adjusted the CodeLens positioning to display the complexity above the function name. I also handled edge cases, like ignoring invalid lines and ensuring the correct initialization of nodes and edges.

## Conclusion

Developing this extension was both challenging and rewarding. Through this project, I not only learned about cyclomatic complexity and how to compute it, but also gained experience in writing VS Code extensions and integrating external libraries. The extension is now capable of providing a quick and easy way for developers to monitor the complexity of their C functions right in their code editor, which can be a great aid in maintaining code quality.

## Next Steps

In the future, I plan to enhance the extension to support more languages and add more features such as providing suggestions for reducing complexity. I look forward to the continuous improvement of this tool.

## Acknowledgements

I want to thank OpenAI's ChatGPT for its invaluable assistance throughout the project. It served as a reliable partner in brainstorming solutions, providing code snippets, and even writing tests. This project would not have been possible without it.
