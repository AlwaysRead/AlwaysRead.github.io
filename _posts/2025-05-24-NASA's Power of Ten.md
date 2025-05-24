---
title: "Lessons from NASA's Power of Ten: How to Write Safer, Cleaner Code in Your Everyday Projects"
date: 2025-05-24 00:00:00 +0530
categories: [Coding]
tags: [C, C++]
---

<img src="https://www.nasa.gov/wp-content/uploads/2018/07/s75-31690.jpeg" alt="NASA's Power of Ten Coding Rules" height="10">

When you're writing C code for a spacecraft, aircraft, or nuclear power plant systems where a single bug could turn "all systems go" into "oh no, we're toast"—there's no room for sloppiness. **NASA's Jet Propulsion Laboratory (JPL)** knew this and unleashed the "Power of Ten Rules," a set of ten ironclad coding commandments carved by **Gerard J. Holzmann**. These rules emphasize simplicity, verifiability, and clarity over flexibility, ensuring rock-solid reliability. Let's explore why these rules are vital, not just for NASA but for any coders/developer, including you and me, and why adopting them can elevate our coding practices to build safer, more dependable software.

## Why These Rules Exist

Most coding guidelines are lengthy, unclear, and often ignored by developers. NASA's JPL takes a different approach with the Power of Ten Rules: ten concise, precise guidelines for C code, designed to be clear, enforceable, and verifiable with automated tools. These rules ensure code is predictable and analyzable, essential for safety-critical systems where failure is not an option. Though strict, they prioritize reliability over flexibility, acting like a safeguard for your code, ensuring it performs flawlessly in high-stakes environments.

## The Ten Rules

Here's a breakdown of the Power of Ten Rules, with simple understanding:

### Rule 1: Simple Control Flow

> **"Restrict all code to very simple control flow constructs – do not use goto statements, setjmp or longjmp constructs, and direct or indirect recursion."**

Keep control flow simple: Avoid `goto`, `setjmp`, `longjmp`, and recursion, ensures safety-critical code remains predictable and verifiable, crucial for systems. By banning complex constructs like `goto` and recursion, which can create messy, hard-to-trace execution paths or stack overflows, this rule enforces straightforward control flow using loops and conditionals, making it easier for automated tools to test and verify. This simplicity guarantees an acyclic call graph, preventing runaway code and ensuring reliability in high-stakes environments, while also making debugging easier for any project.

### Rule 2: Fixed Loop Bounds

> **"All loops must have a fixed upper-bound. It must be trivially possible for a checking tool to prove statically that a preset upper-bound on the number of iterations of a loop cannot be exceeded. If the loop-bound cannot be proven statically, the rule is considered violated."**

"All loops must have a fixed upper-bound," ensures safety-critical code doesn't spiral into infinite loops, a disaster waiting to happen in systems like spacecraft or nuclear plants—imagine a loop running wild at 12:51 PM IST on May 24, 2025, during a critical mission! This rule requires loops to have a provable limit, verifiable by tools, to prevent runaway code, except in cases like process schedulers where non-termination is intentional and must be proven. For variable loops, like traversing a linked list, you set an explicit upper-bound, triggering an assertion failure and error return if breached (see Rule 5), keeping your code predictable, safe, and ready for high-stakes environments.

### Rule 3: No Dynamic Memory Allocation

> **"Do not use dynamic memory allocation after initialization."**

The reason is simple: memory allocators, such as `malloc`, and garbage collectors often have unpredictable behavior that can significantly impact performance. Dynamic memory allocation can be unpredictable, impacting performance, and often leads to errors like memory leaks, using freed memory, or exceeding available memory, all of which are disastrous in systems like spacecraft. By restricting applications to a fixed, pre-allocated memory space and using stack memory whose usage can be statically verified thanks to Rule 1's recursion ban—this rule eliminates these risks, ensuring verifiable, reliable memory use in high-stakes environments.

### Rule 4: Short Functions

> **"No function should be longer than what can be printed on a single sheet of paper in a standard reference format with one line per statement and one line per declaration. Typically, this means no more than about 60 lines of code per function."**

Who likes to write 100 lines of code a function. "No function should be longer than 60 lines," ensures code for safety-critical systems remains manageable and reliable. By keeping functions short—fitting on a single printed page or screen—each becomes a clear, verifiable logical unit, easy to understand and test. Long functions often signal poor structure, hiding bugs and complicating analysis, which is unacceptable in systems like spacecraft where clarity is non-negotiable, and even in everyday coding, this rule helps us write cleaner, more maintainable code.

### Rule 5: Assertion Density

> **"The assertion density of the code should average to a minimum of two assertions per function. Assertions are used to check for anomalous conditions that should never happen in real-life executions. Assertions must always be side-effect free and should be defined as Boolean tests. When an assertion fails, an explicit recovery action must be taken, e.g., by returning an error condition to the caller of the function that executes the failing assertion. Any assertion for which a static checking tool can prove that it can never fail or never hold violates this rule. (I.e., it is not possible to satisfy the rule by adding unhelpful assert(true) statements.)."**

Statistics for industrial coding efforts indicate that unit tests often find at least one defect per 10 to 100 lines of code written. The odds of intercepting defects increase with assertion density. Use of assertions is often also recommended as part of a strong defensive coding strategy. Assertions can be used to verify pre- and postconditions of functions, parameter values, return values of functions, and loop invariants. Because assertions are side-effect free, they can be selectively disabled after testing in performance-critical code.

A typical use of an assertion would be as follows:

   ```c
    if (!c_assert(p >= 0) == true) {
        return ERROR;
    }
   ```
   {: .nolineno }
   with the assertion defined as follows:
   
   ```c
    #define c_assert(e)     ((e) ? (true) : \
        (tst_debugging("%s,%d: assertion '%s' failed\n", \
        __FILE__, __LINE__, #e), false))
   ```
   {: .nolineno }
   
In this definition, `__FILE__` and `__LINE__` are predefined by the macro preprocessor
to produce the filename and line-number of the failing assertion. The syntax #e turns
the assertion condition e into a string that is printed as part of the error message. In
code destined for an embedded processor there is of course no place to print the error
message itself – in that case, the call to `tst_debugging` is turned into a no-op, and
the assertion turns into a pure Boolean test that enables error recovery from
anomolous behavior.

### Rule 6: Minimize Variable Scope

> **"Data objects must be declared at the smallest possible level of scope."**

This promotes data-hiding to ensure safety-critical code stays reliable, crucial for systems like aircraft where a corrupted value could lead to failure during a flight. By keeping a variable's scope minimal, it can't be unintentionally accessed or modified, lowering the risk of errors. Additionally, with fewer spots where the variable might be changed, tracking down incorrect values becomes simpler. This rule also discourages reusing variables for unrelated tasks, which can hinder fault diagnosis, leading to clearer, more dependable code for critical applications and our own work.

### Rule 7: Check Return Values

> **"The return value of non-void functions must be checked by each calling function, and the validity of parameters must be checked inside each function."**

This ensures safety-critical code catches errors. While often violated, this rule requires checking even `printf` or `close` return values, unless success and failure responses are identical, in which case you can cast to `(void)` with a comment justifying it. Ignoring return values, like in `strlen(0)` or `strcat(s1, s2, -1)`, can lead to crashes, so this rule enforces accountability, using tools to flag violations, making code safer and more reliable for critical and everyday use.

### Rule 8: Limited Preprocessor Use

> **"The use of the preprocessor must be limited to the inclusion of header files and simple macro definitions. Token pasting, variable argument lists (ellipses), and recursive macro calls are not allowed. All macros must expand into complete syntactic units. The use of conditional compilation directives is often also dubious, but cannot always be avoided. This means that there should rarely be justification for more than one or two conditional compilation directives even in large software development efforts, beyond the standard boilerplate that avoids multiple inclusion of the same header file. Each such use should be flagged by a tool-based checker and justified in the code."**

Limiting preprocessor use to header includes and simple macros," tackles the chaos the C preprocessor can unleash. The preprocessor, with its ability to obscure code through complex macros, token pasting, or recursive calls, confuses both developers and text-based checkers, making it hard to decipher program logic even with C's formal standards. Additionally, excessive conditional compilation, like using ten directives, can create up to 1,024 (2^10) code versions, each needing testing, which skyrockets complexity. By restricting preprocessor use, this rule ensures clarity and verifiability, benefiting both high-stakes systems and our own coding projects.

### Rule 9: Restrict Pointer Use

> **"The use of pointers should be restricted. Specifically, no more than one level of dereferencing is allowed. Pointer dereference operations may not be hidden in macro definitions or inside typedef declarations. Function pointers are not permitted."**

Pointers, especially when misused by even seasoned programmers, obscure data flow, making it tough for static analyzers to verify code. Function pointers further complicate things by hindering checks for recursion or control flow, requiring extra guarantees if used. By limiting pointer use no more than one dereference, no hiding in macros or typedefs, and no function pointers, this rule ensures clarity and verifiability, keeping code safe and manageable for both critical missions.

### Rule 10: Strict Compilation and Analysis

> **"All code must be compiled, from the first day of development, with all compiler warnings enabled at the compiler's most pedantic setting. All code must compile with these setting without any warnings. All code must be checked daily with at least one, but preferably more than one, state-of-the-art static source code analyzer and should pass the analyses with zero warnings."**

With powerful static analyzers—both commercial and free—readily available, there's no excuse to skip them, even for non-critical projects. This rule demands compiling at the strictest settings and using tools to catch issues early, rewriting code if warnings arise, even if they seem false, as they often reveal subtle bugs. Modern analyzers, unlike older tools like lint, are fast and accurate, making their use essential for reliable, high-quality code in any serious project.

## Conclusion

While we may not always follow NASA's Power of Ten Rules in everyday coding. Since we're not building sci-fi spaceships; understanding their focus on critical code reliability can make our programs clearer and less error-prone. C, being a highly versatile language, offers countless ways to write code, which is both powerful and astonishing. However, adopting some of these rules in our daily work, even for non-critical projects, simplifies our code, making it easier to understand and debug, ultimately leading to more robust and maintainable codebases.

---

> **Further Reading**: You can find the original [NASA JPL Power of Ten Rules document (PDF)](https://spinroot.com/gerard/pdf/P10.pdf) by Gerard J. Holzmann and additional resources on their official website. These guidelines continue to influence modern software development practices across various industries beyond aerospace.
