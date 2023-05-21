---
layout: post
title:  "Decoding Cyclomatic Complexity Calculation for C++ Programs"
date:   2023-05-21 22:22:00 +0800
categories: software-testing cyclomatic-complexity clang libclang llvm
---

Tackling intricacies in software development often leads to profound insights. This was evident when I was working on a project that required calculating the cyclomatic complexity for a C++ program named `ccc`. During testing, I ran into unexpected outputs, which led to an exciting journey of debugging and refining the cyclomatic complexity calculator.

## Test Case Discrepancy: Ternary Operators

Consider the following test function:

```cpp
void ternary() {
    int a = 10;
    int b = (a == 10) ? 20 : 30;
}
```

The cyclomatic complexity calculation for this function came out as 1, while it should be 2 because the ternary operator `(a == 10) ? 20 : 30` represents a decision point. This revealed that the cyclomatic complexity calculator wasn't correctly accounting for ternary operators. 

To fix this, I adjusted the Clang-based AST parser to include ternary operators as decision points in the complexity calculation, thereby aligning the results with the expected outcomes.

## Test Case Discrepancy: Logical Operators

In the next test function:

```cpp
void multi_conditions() {
    int a = 10, b = 20;
    if (a == 10 && b == 20) {}
    if (a == 10 || b == 20) {}
}
```

The calculator attributed a higher complexity than expected due to the presence of logical AND `&&` and OR `||` operators. To address this discrepancy, I further refined the Clang parser to appropriately account for these logical operators.

## Function to Detect Binary Logical Operators

This required identifying whether a binary operator was a logical operator and modifying the calculation to account for these as additional decision points. I accomplished this with the function `getBinaryOperator` and the struct `EdgeAndNodeCounter`:

```cpp
const std::vector<CXCursorKind> decision_kinds = {
    CXCursor_IfStmt, CXCursor_ForStmt, CXCursor_WhileStmt, CXCursor_DefaultStmt,
    CXCursor_CaseStmt, CXCursor_ConditionalOperator, CXCursor_BinaryOperator};

static CXCursor getFirstChild(CXCursor parentCursor)
{
    CXCursor childCursor = clang_getNullCursor();
    clang_visitChildren(
        parentCursor,
        [](CXCursor c, CXCursor parent, CXClientData client_data)
        {
            CXCursor *cursor = static_cast<CXCursor *>(client_data);
            *cursor = c;
            return CXChildVisit_Break;
        },
        &childCursor);

    return childCursor;
}

std::string getBinaryOperator(CXTranslationUnit translationUnit, CXCursor expressionCursor)
{
    CXToken *expressionTokens;
    unsigned numExpressionTokens;
    clang_tokenize(translationUnit, clang_getCursorExtent(expressionCursor),
                   &expressionTokens, &numExpressionTokens);

    CXCursor leftHandSideCursor = getFirstChild(expressionCursor);
    CXToken *leftHandSideTokens;
    unsigned numLeftHandSideTokens;
    clang_tokenize(translationUnit, clang_getCursorExtent(leftHandSideCursor),
                   &leftHandSideTokens, &numLeftHandSideTokens);

    CXString operatorString = clang_getTokenSpelling(translationUnit,
                                                     expressionTokens[numLeftHandSideTokens]);
    std::string operatorSymbol(clang_getCString(operatorString));

    clang_disposeString(operatorString);
    clang_disposeTokens(translationUnit, leftHandSideTokens, numLeftHandSideTokens);
    clang_disposeTokens(translationUnit, expressionTokens, numExpressionTokens);

    return operatorSymbol;
}

struct EdgeAndNodeCounter
{
    CXTranslationUnit translationUnit;
    int edges = 0;
    int nodes = 0;
};

CXChildVisitResult countEdgesAndNodesCallback(CXCursor cursor, CXCursor parent, CXClientData clientData)
{
    EdgeAndNodeCounter *counter = static_cast<EdgeAndNodeCounter *>(clientData);

    const CXCursorKind cursorKind = clang_getCursorKind(cursor);
    if (std::find(decision_kinds.begin(), decision_kinds.end(), cursorKind) != decision_kinds.end())
    {
        if (cursorKind == CXCursor_BinaryOperator)
        {
            std::string operatorSymbol = getBinaryOperator(counter->translationUnit, cursor);
            if (operatorSymbol == "&&" || operatorSymbol == "||")
            {
                counter->edges += 2;
                counter->nodes += 1;
            }
        }
        else
        {
            counter->edges += 2;
            counter->nodes += 1;
        }
    }

    clang_visitChildren(cursor, countEdgesAndNodesCallback, clientData);

    return CXChildVisit_Continue;
}
```

This code detects the `&&` and `||` operators by using Clang's APIs to parse and tokenize the source code of a C++ program. It then traverses the abstract syntax tree (AST) of the program to examine each node, particularly focusing on those that represent binary operators.

Here's a step-by-step breakdown:

1. **Setting up Decision Kinds:** The `decision_kinds` vector contains all the cursor kinds that represent decision points in the code, including `IfStmt`, `ForStmt`, `WhileStmt`, `DefaultStmt`, `CaseStmt`, `ConditionalOperator`, and `BinaryOperator`.

2. **Fetching the First Child:** The `getFirstChild` function retrieves the first child cursor of a given cursor, which can be helpful to get the operands of a binary operator.

3. **Tokenizing the Expression:** The `getBinaryOperator` function tokenizes an expression represented by a `CXCursor` into a list of `CXToken` objects. This tokenization process breaks down the expression into its smallest components.

4. **Finding the Operator Symbol:** After tokenizing the expression and its left-hand side operand, the function retrieves the operator symbol as a string by taking the spelling of the token immediately after the left-hand side tokens in the expression tokens list.

5. **Counting Edges and Nodes:** The `countEdgesAndNodesCallback` function visits each cursor in the AST and checks if the cursor kind is among the decision kinds. If the cursor is a binary operator, it retrieves the operator symbol and checks if it's either `&&` or `||`. If it is, the function increments the edges counter by 2 and the nodes counter by 1.

This approach ensures the detection of logical operators `&&` and `||` by analyzing the tokenized form of expressions and focusing on binary operators within decision-making constructs.

## Debugging Lessons and Conclusion

This debugging journey demonstrated the complexity introduced by programming constructs such as ternary and logical operators. We refined our cyclomatic complexity calculator to accurately handle these scenarios, bolstering the reliability of this critical software metric. Additionally, this experience reinforced the importance of rigorous testing and a sound understanding of both the programming constructs and the tools used in the development process. The hands-on exercise underscored the crucial role of underlying principles for success in software development.
