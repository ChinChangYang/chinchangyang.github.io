---
layout: post
title:  "Enhancing Real-time Analysis in a Cyclomatic Complexity VSCode Extension"
date:   2023-05-15 21:38:00 +0800
categories: vscode-extension cyclomatic-complexity c-language refactoring clang
---

I am the author of the Cyclomatic Complexity CodeLens extension for Visual Studio Code, a tool designed to assess the complexity of your C code in real-time. During the development process, I encountered a unique challenge - calculating the cyclomatic complexity for unsaved or in-memory C source code. In this blog post, I will describe the problem, the solution I arrived at, and the steps I took to achieve it. You can follow along with the source code on the project's GitHub page: [https://github.com/ChinChangYang/cyclomatic-complexity-c](https://github.com/ChinChangYang/cyclomatic-complexity-c).

**Problem**

The initial implementation of the extension required the source file to be saved before it could calculate the cyclomatic complexity. This approach was not ideal, as it did not provide real-time feedback to the users while they were modifying their code. The challenge, then, was to modify the extension and its underlying Clang-based tool ("ccc") to process unsaved or in-memory code.

**Solution**

The solution to this problem involved modifications both in the VS Code extension and in the underlying "ccc" program. The extension was changed to pass the in-memory source code directly to the "ccc" program, and the "ccc" program was modified to accept input from a buffer (in this case, stdin) instead of reading from a saved file.

**Step-by-step Guide**

**Step 1: Modification in the Extension**

Firstly, I modified the extension to spawn the "ccc" program and write the C code to its stdin. Here is how the "getScriptOutput" method in "extension.ts" looks after modification:

```ts
private async getScriptOutput(document: vscode.TextDocument): Promise<string> {
    const script = spawn(this.cccPath);

    // Write the content of the current file to the ccc program
    script.stdin.write(document.getText());
    script.stdin.end();

    // Rest of the method...
}
```

**Step 2: Modification in the "ccc" Program**

Next, I modified the "ccc.cpp" file to read the source code from stdin and treat it as an unsaved file. The 'main' function was refactored to improve its cyclomatic complexity and maintainability.

```cpp
int main() {
    std::string code = readSourceCode();
    CXUnsavedFile unsaved_file = createUnsavedFile(code);
    
    CXIndex index = clang_createIndex(0, 0);
    CXTranslationUnit TU = parseTranslationUnit(index, &unsaved_file);
    
    CXCursor root_cursor = clang_getTranslationUnitCursor(TU);
    visitChildren(root_cursor);
    
    clang_disposeTranslationUnit(TU);
    clang_disposeIndex(index);
    return 0;
}
```

You can find the detailed code changes in the GitHub commit.

**Conclusion**

The end result of these changes was a more responsive and user-friendly extension, capable of calculating cyclomatic complexity in real-time as the user types their code. This was a complex problem to solve but ultimately a rewarding one, providing valuable insight into the power and flexibility of both Visual Studio Code extensions and the Clang C API.

If you are a C developer using VS Code, give the extension a try and experience real-time cyclomatic complexity calculation: [https://github.com/ChinChangYang/cyclomatic-complexity-c](https://github.com/ChinChangYang/cyclomatic-complexity-c). Feedback and contributions are always welcome!

Happy coding!