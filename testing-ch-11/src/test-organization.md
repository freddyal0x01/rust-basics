# Test Organization

Testing is a complex discipline, and different people use different terminology and organization.

The Rust community thinks about tests in terms of 2 main categories: 

- **Unit Tests**

- **Integration Tests**

Unit tests are small and more focused, testing one module in isolation at a time, and can test private interfaces. 

Integration tests are entirely external to our library and use our code in the same way any other external code would, using only the public interface and potentially exercising multiple modules per test. 

Writing both kinds of tests is important to ensure that the pieces of our library are doing what we expect them to, separately and together. 


## Unit Tests

The purpose of unit tests are to test each unit of code in isolation from the rest of the code to quickly pinpoint where code is and isn't working as expected. 

We'll put unit tests in the `src` directory in each file with the code that they're testing. 
The convention is to create a module named `tests` in each file to contain the test functions and to annotate the module with `cfg(test)`.


### The Tests Module and `#[cfg(test)]`

The `#[cfg(test)]` annotation on the tests module tells Rust to compile and run the test code only when run `cargo test`, not when we run `cargo build`. 
This saves compile time when we only want to build the library and saves space in the resulting compiled artifact because the tests aren't included. 

We'll see that integration tests go in a different directory, they don't need the `#[cfg(test)]` annotation. 
However, because unit tests go in the same files as the code, we'll use `#[cfg(test)]` to specify that they shouldn't be included in the compiled result. 

When we generated the new `addr` project, Cargo generated the code for us: 

File - `src/lib.rs`

```
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

This code is the automatically generated tested module. 
The attribute `cfg` stands for **configuration** and tells Rust that the following item should be included given a certain configuration option. 

In this case, the configuration option is `test`, which is provided by test for compiling and running tests.
By using the `cfg` attribute, Cargo compiles our test code only if we actively run the tests with `cargo test`. 
This also includes any helper function that might be within this module, in addition to the functino annotated with `#[test]`.


### Testing Private Functions

The testing community debates whether or not private functions should be tested directly, and other languages make it difficult or impossible to test private functions.

Regardless of which testing ideology we adhere to, Rust's privacy rules allow us to test private functions. 

Example: Testing a private function `internal_addr`

```
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

The `internal_addr` function isn't marked as `pub`. 

Tests are just Rust code, and the `tests` module is just another module.

In the test above, we bring all of the `test` module's parent's items into scope with `use super::*`, and then the test can call `internal_addr`. 

We don't think private functions should be tested, there's nothing in Rust that will compel us to do so. 


## Integration Tests

In Rust, integration tests are entirely external to our library. 
They use our library in the same way any other code would, which means they can only call functions that are part of our library's public API. 
Their purpose is to test whether many parts of our library work together correctly. 

Units of code that work correctly on their own could have problems when integrated, so test coverage of the integrated code is important as well.

To create integration tests, we need a `tests` directory. 


### The `tests` Directory

We create a `tests` directory at the top level of our project directory, next to `src`. 

Cargo knows to look for integration test files in this directory. 
We can make as many test files as we want to in this directory, and Cargo will compile each of the files as an individual crate. 

Example: An integratino test of a function in the `addr` crate

File - `tests/integration_test.rs`

```
use adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

We added `use addr` at the top of the code, which we don't need in the unit tests. 
THe reason we do this is that each file in teh `tests` directory is a separate crate, so we need to bring our library into each test crate's scope. 

We don't need to annotate any code in `tests/integration_test.rs` with `#[cfg(test)]`. 
Cargo treats the `tests` directory specially and compiles files in this directory only when we run `cargo test`. 

Output of `cargo test`

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 1.31s
     Running unittests (target/debug/deps/adder-1082c4b063a8fbe6)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-1082c4b063a8fbe6)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

The 3 sections of output include the unit tests, the integration test, and the docs test. 

The first section for the unit tests is the same as we've been seeing: one line for each unit test (one named `internal`) and then a summary line for the unit tests. 

The integration tests section starts with the line `Running target/debug/deps/integration_test-b2667f444a972106` (The hash at the end of our output will be different).
Next, there's a line for each test function in that integration test and a summary line for the results of the integration test just before the `Docs-test adder` section starts. 

Similarly to how adding more unit test functions adds more result lines to the unit tests section, adding more test functions to the integration test file adds more result lines to this integration test file’s section.
Each integration test file has its own section, so if we add more files in the tests directory, there will be more integration test sections.

We can still run a particular integration test function by specifying the test function's name as an argument to `cargo test`. 

To run all of the tests in a particular integration test file, use the `--test` argument of `cargo test` following by the name of the file:

```
$ cargo test --test integration_test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.64s
     Running tests/integration_test.rs (target/debug/deps/integration_test-82e7799c1bc62298)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

This command runs only the tests in the `tests/integration_test.rs` file.


### Submodules in Integration Tests

As we add more integration tests, we might want to make more than one file in the `tests` directory to help organize them. 
Example: We can group the test functions by the functionality they're testing. 

Treating each integration test file as its own crate is useful to create separate scopes that are more like the way end users will be using our crate. 
However, this means files in the `test` directory don't share the same behavior as files in `src` do. 

The different behavior of files in the `tests` directory is most noticeable when we have a set of helper function that would be useful in multiple integration test files and we try to extract them into a common module.

Example: We create a `test/common.rs` file and place a function named `setup` in it, we can add some code to `setup` that we want to call from multiple test functions in multiple test files: 

File - `test/common.rs`

```
pub fn setup() {
    // setup code specific to your library's tests would go here
}
```

When we run the test again, we'll see a new section in the test output for the `common.rs` file, even though this file doesn't contain any test functions nor did we call the setup function from anywhere; 

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.89s
     Running unittests (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/common.rs (target/debug/deps/common-92948b65e88960b4)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-92948b65e88960b4)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Having the `common` test appear results with `running 0 tests` displayed for it's not what we wanted. 

To avoid having `common` appear in the test output, instead of creating `tests/common.rs`, we'll create `test/common/mod.rs`. 
This is an alternate naming convention that Rust also understands. 
Naming the file this way tells Rust not to treat the `common` module as an integration test file. 

When we move the `setup` function code into `test/common/mod.rs` and delete the `tests/common.rs` file, the section in the test output will no longer appear. 
Files in subdirectories of the `tests` directory don't get compiled as separate crates or have sections in the test output. 

After we've created `tests/common/mod.rs`, we can use it from any integration test files as a module. 

Example: Calling the `setup` function from the `it_adds_two` test in `test/integration_test.rs`

File - `tests/integration_test.rs`

```
use adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```

The `mod common;` declaration is the same as the module declartion we've demonstrated previously. 
In the test function, we can call the `common::setup()` function. 



### Integration Test for Binary Crates

If our project is a binary crate that only contains a `src/main.rs` file and doesn't have a `src/lib.rs` file, when we can't create integration tests in the `tests` directory and bring functions defined in the `src/main.rs` file into scope with a `use` statement. 
Only library crates expose functions that other crates can use; binary crates are meant to be run on their own. 

This is one of the reasons Rust projects that provide a binary have a straightforward `src/main.rs` file that calls logic that lives in the `src/lib.rs` file. 
Using that structure, integration tests **can** test the library crate with `use` to make the important functionality available. 
If the important functionality works, the small amount of code in the `src/main.rs` file will work as well, and that small amount of code doesn't need to be tested. 
