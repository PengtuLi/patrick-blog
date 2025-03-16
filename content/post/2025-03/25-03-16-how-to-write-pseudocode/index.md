---
title: "How to write a Pseudo Code"
author : "tutu"
date: '2025-03-15'
lastmod: '2025-03-15'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['program language']
---

怎样去用伪代码描述一个算法是很重要的，下面是一些推荐的标准。

伪代码是一种用于描述算法的结构化语言。它允许设计者**专注于算法的逻辑**，而不会被语言语法的细节所分散。
同时，伪代码需要**完整**，它描述算法的整个逻辑，使得实现变成了一种机械的逐行翻译成源代码的任务。

- concise: In general the vocabulary used in the pseudocode **should be the vocabulary of the problem domain**, not of the implementation domain. The pseudocode is a narrative for someone who knows the requirements (problem domain) and is trying to learn how the solution is organized. 可以假设你在向一个不会编程的人描述你实现某个算法的过程。

```raw
Extract the next word from the line (good)
set word to get next token (poor)

Append the file extension to the name (good)
name = name + extension (poor)

FOR all the characters in the name (good)
FOR character = first to last (ok)
```

- fine-grained: Note that the logic must be decomposed to the level of a single loop or decision. Thus "Search the list and find the customer with highest balance" is too **vague** because it takes a loop AND a nested decision to implement it.

- Use mathematical notation when applicable.

- structured: The "structured" part of pseudocode is a notation for representing six specific structured programming constructs: SEQUENCE, WHILE, IF-THEN-ELSE, REPEAT-UNTIL, FOR, and CASE. 

```raw
**SEQUENCE** is a linear progression where one task is performed sequentially after another.
**WHILE** is a loop (repetition) with a simple conditional test at its **beginning**.
**IF-THEN-ELSE** is a decision (selection) in which a choice is made between two alternative courses of action.
**REPEAT-UNTIL** is a loop with a simple conditional test at the **bottom**.
**CASE** is a multiway branch (decision) based on the value of an expression. CASE is a generalization of IF-THEN-ELSE.
**FOR** is a "counting" loop.

# SEQUENCE
all actions aligned with the same indent

# IF-THEN-ELSE
IF HoursWorked > NormalMax THEN
    Display overtime message
ELSE
    Display regular time message
ENDIF

# WHILE
WHILE Population < Limit
    Compute Population as Population + Births - Deaths
ENDWHILE

# CASE
CASE grade OF
    A : points = 4
    B : points = 3
    C : points = 2
    D : points = 1
    F : points = 0
    others : points = -1
ENDCASE

# REPEAT-UNTIL
REPEAT
    sequence
UNTIL condition

# FOR
FOR each month of the year (good)
FOR month = 1 to 12 (ok)
FOR each employee in the list (good)
FOR empno = 1 to listsize (ok)

FOR each month of the year
    sequence
ENDFOR

-----------------------------------

#  INVOKING SUBPROCEDURES

Use the CALL keyword. For example:

CALL AvgAge with StudentAges
CALL Swap with CurrentItem and TargetItem
CALL Account.debit with CheckAmount
CALL getBalance RETURNING aBalance
CALL SquareRoot with orbitHeight RETURNING nominalOrbit
```

>"Pretty Good"  This example shows how pseudocode is written as comments in the source file.

```raw
public boolean moveRobot (Robot aRobot)
{
    //IF robot has no obstacle in front THEN
        // Call Move robot
        // Add the move command to the command history
        // RETURN true
    //ELSE
        // RETURN false without moving the robot
    //END IF
}
```