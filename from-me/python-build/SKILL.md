---
name: python-build
description: Write and modify Python code. Only used when the user explicitly states such as Use python-build.
---

## Overview

Help users create or modify Python scripts. Prioritize clear logic, minimal changes, verifiable results, and code suitable for stable execution.

## Core Instructions
   
1. **Minimal Changes**
   
   - Change only the code required to satisfy the request.
   - Avoid unrelated refactoring, renaming, formatting, or style cleanup. However, limited refactoring is acceptable when it directly supports the task — for example, extracting a helper to eliminate duplication introduced by the change, or renaming a misleading identifier that would cause confusion in the new logic. When you do deviate, briefly note why in the response.
   
2. **Python Code Standards**
   
   - Extract critical configuration values into top-level constants using `UPPER_SNAKE_CASE` when they are reused, user-configurable, environment-specific, or important for operation.
   - Add a brief Chinese comment for constants whose meaning is not obvious from the name or value.
   - Add Chinese docstrings for public functions, complex functions, or functions with non-obvious inputs and outputs. Simple internal helper functions do not require docstrings when their names and code are self-explanatory.
   - Explain parameters in docstrings when parameter meaning, accepted values, side effects, or constraints are not immediately obvious.
   - Avoid clever syntax. Prefer clear `if-else` blocks, standard loops, and explicit logic.
   
3. **Language Requirements**
   - All code comments, docstrings, parameter explanations, and descriptive text inside generated code must be written in professional, concise Chinese.


> For path handling strategies and LLM access patterns, see [REFERENCE.md](REFERENCE.md).
