# Summary

Rust’s error handling features are designed to help you write more robust code.
The `panic!` macro signals that our program is in a state it can’t handle and lets you tell the process to stop instead of trying to proceed with invalid or incorrect values. The `Result` enum uses Rust’s type system to indicate that operations might fail in a way that your code could recover from. 

We can use `Result` to tell code that calls our code that it needs to handle potential success or failure as well. Using `panic!` and Result in the appropriate situations will make your code more reliable in the face of inevitable problems.
