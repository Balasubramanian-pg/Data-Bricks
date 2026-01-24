# 068 Python Magic Command

Canonical documentation for 068 Python Magic Command. This document defines concepts, terminology, and standard usage.

## Purpose
The 068 Python Magic Command system exists to provide a meta-syntactic interface for controlling the execution environment, performing system-level tasks, and enhancing the interactive development workflow without altering the core Python language syntax. It addresses the requirement for a "command-line within the language" that allows developers to interact with the interpreter's state, the underlying operating system, and the memory space in a declarative manner.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative. While frequently associated with the IPython kernel, these specifications define the abstract behavior of Magic Commands as a protocol for interactive computing.

## Scope
Clarify what is in scope and out of scope for this topic.

**In scope:**
* Syntax and prefix conventions (Line vs. Cell magics).
* Execution context and namespace interaction.
* Lifecycle of a magic command invocation.
* Standardized categories of magic operations (Environment, Profiling, Debugging).

**Out of scope:**
* Specific vendor-specific magics (e.g., proprietary cloud-provider magics).
* Implementation details of the underlying parser (e.g., specific Regex patterns used by a kernel).
* Non-Python interactive shells (e.g., Bash, PowerShell) unless invoked via a magic interface.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **Line Magic** | A command prefixed by a single percent sign (`%`) that operates on a single line of input. |
| **Cell Magic** | A command prefixed by a double percent sign (`%%`) that operates on the entire contents of a code block (cell). |
| **Magic Function** | The underlying Python function or object that implements the logic triggered by a magic command. |
| **Automagic** | A configuration state where line magics can be called without the `%` prefix, provided the command does not conflict with a defined variable. |
| **Escaping** | The process of passing a command directly to the underlying system shell, often related to but distinct from the magic system. |

## Core Concepts
### The Prefix Mechanism
The 068 standard utilizes the `%` and `%%` prefixes to signal to the pre-processor that the subsequent string is not standard Python code. This allows the interpreter to intercept the command before it reaches the Python Abstract Syntax Tree (AST) parser.

### Execution Context
Magic commands execute within the same process as the Python kernel but often operate on a "meta" level. They have access to the global namespace, allowing them to inspect, modify, or inject variables into the user's current session.

### Interactivity and State
Unlike standard library functions, magic commands are designed for ephemeral, interactive state management. They are optimized for human readability and rapid feedback rather than programmatic recursion or library integration.

## Standard Model
The standard model for 068 Python Magic Commands follows a two-tier hierarchy:

1.  **Line-Level Operations (`%`):**
    *   Syntax: `%command [arguments]`
    *   Behavior: The command consumes the remainder of the line. It may return a value that can be assigned to a Python variable.
2.  **Cell-Level Operations (`%%`):**
    *   Syntax: `%%command [arguments]` followed by a newline and a block of text.
    *   Behavior: The command consumes the entire block. The first line contains the configuration/arguments, and the subsequent body is passed as a string or stream to the magic handler.

## Common Patterns
### Environment Management
Used to modify the working directory, environment variables, or shell state (e.g., `%cd`, `%env`, `%set_env`).

### Performance Profiling
Used to measure execution time or memory consumption of specific code segments (e.g., `%timeit`, `%prun`, `%memit`).

### External Integration
Used to run external scripts or load extensions into the current session (e.g., `%run`, `%load_ext`, `%pip`).

### Introspection
Used to explore the current namespace or search through command history (e.g., `%who`, `%whos`, `%history`).

## Anti-Patterns
*   **Production Scripting:** Including magic commands in `.py` files intended for production deployment. Magic commands are non-standard Python and will cause `SyntaxError` in a standard Python interpreter.
*   **Variable Shadowing:** Naming a Python variable the same as a common magic command when "Automagic" is enabled, leading to ambiguous execution.
*   **Over-reliance on Cell Magics for Logic:** Using `%%` magics to bypass proper modularization (e.g., writing an entire SQL script inside a Python cell rather than using a database driver).

## Edge Cases
*   **Variable Expansion:** Some implementations allow Python variables to be passed into magic commands using the `{var}` or `$` syntax. The 068 standard recognizes this as an optional but common extension.
*   **Nested Magics:** Generally, cell magics cannot be nested within other cell magics. Line magics may appear within a cell, but their behavior depends on whether the cell magic handler is designed to pre-process them.
*   **Output Redirection:** Capturing the output of a magic command into a Python variable (e.g., `output = %magics_command`) is supported for line magics but behavior is undefined for cell magics unless specifically implemented by the handler.

## Related Topics
*   **IPython Kernel:** The primary reference implementation of the magic system.
*   **Jupyter Messaging Protocol:** The communication layer that often carries magic command requests.
*   **REPL (Read-Eval-Print Loop):** The interactive environment type that necessitates magic commands.
*   **Shell Scripting:** The domain from which many magic commands derive their syntax.

## Change Log
| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-01-12 | Initial AI-generated canonical documentation |