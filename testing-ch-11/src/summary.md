# Summary

Rust's testing features provide a way to specify how code should function to ensure it continues to work as we expect, even as we make changes. 

Unit tests exercise different parts of a library separately and can test private implementation details. 

Integration tests check that many parts of the library work together correctly, and they use the library's public API to test the code in the same way external code uses it. 

Even though Rust’s type system and ownership rules help prevent some kinds of bugs, tests are still important to reduce logic bugs having to do with how your code is expected to behave.
