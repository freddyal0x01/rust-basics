# Controlling How Tests are Run

Just as `cargo run` compiles your code and then runs the resulting binary, `cargo test` compiles our code in test mode and runs the resulting test binary.

We can specify command line options to change the default behavior of `cargo test`. 

Example: The default behavior of the binary produced by `cargo test` is to run all the tests in parallel and capture output generated during test runs, preventing the output from being displayed and making it easier to read the output related to the test results. 

Some command line options go to `cargo test`, and some go to the resulting test binary.
To separate these two types of arguments, we list the arguments that go to `cargo test` followed by the separator `--` we can use with the `cargo test`, and running the `cargo test -- --help` displays the options we can use after the separtor `--`. 


## Running Tests in Parallel or Consecutively

When we run multiple tests, by default they run in parallel using threads. 
This means the tests will finish running faster we can get feedback quicker on whether or not our code is working. 
Because the tests are running at the same time, we should make sure our tests don't depend on each other or on any shared state, including a shared environment, such as the current working directory or env variables. 

Example: Each of our tests run some code that creates a file on disk named `test-output.txt` and writes some data to that file. 
Then each test reads the data in that file and asserts that the file contains a particular value, which is different in each test. 
Because the tests run at the same time, one test might overwrite the file between when another test writes and reads the file. 
The second test will then fail, not because the code is incorrect but because the tests have interfered with each other while running in parallel
One solution is to make sure each test writes to a different file; another solution is to run each test consecutively. 

If we don't want to run tests in parallel or if we want more fine-grained control over the number of threads used, we can send the `--test-threads` flag and the number of threads we want to use to the test binary. 

Example: Telling the program to not use any parallelism by setting the test threads to `1`. 

```
$ cargo test -- --test-threads=1
```

Running the tests using one thread will take longer than running them in parallel, but the tests won't interfere with each other if they share state. 

## Showing Function Output

By default, if a test passes, Rust's test library captures anything printed to standard output. 

Example: If we call `println!` in a test and the test passes, we won't see the `println!` output in the terminal; we'll see only the line that indicates that the test passed. 
If a test fails, we'll see whatever was printed to the standard output with the rest of the failure message. 

Code Example: Testing a function that prints the value of its parameter and return 10, as well as a test that passes and a test that fails. 

```
fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {}", a);
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(10, value);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(5, value);
    }
}
```

When we run the test with `cargo test`, we see this output: 

```
$ cargo test
   Compiling silly-function v0.1.0 (file:///projects/silly-function)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests (target/debug/deps/silly_function-160869f38cff9166)

running 2 tests
test tests::this_test_will_fail ... FAILED
test tests::this_test_will_pass ... ok

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8
thread 'main' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass '--lib'
```

Note: Nowhere in the output do we see `I got the value 4`, which is what's printed when the test that passes runs. That output has been captured. 

In the failed test, we got the output `I got the value 8`, which appears in the test summary output which also shows the cause of the test failure. 

If we want to see printed values for passing tests as well, we can tell Rust to also show the output of successful test at the end with `--show-output`. 

```
$ cargo test -- --show-output
```

When we run the test with the `--show-output` flag, we see this output: 

```
$ cargo test -- --show-output
   Compiling silly-function v0.1.0 (file:///projects/silly-function)
    Finished test [unoptimized + debuginfo] target(s) in 0.60s
     Running unittests (target/debug/deps/silly_function-160869f38cff9166)

running 2 tests
test tests::this_test_will_fail ... FAILED
test tests::this_test_will_pass ... ok

successes:

---- tests::this_test_will_pass stdout ----
I got the value 4


successes:
    tests::this_test_will_pass

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8
thread 'main' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass '--lib'
```

We get the output of `I got the value 4`. 


## Running a Subset of Tests by Name

Running a full test suite can take a long time. 
If we're working on code in a particular are, we can run only the tests pertaining to that code. 

We can choose which tests to run by passing `cargo test <name or names of the tests we want to run>`

Example: We will run a subset of the 3 tests for the `add_two` function. 

```
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
```

When we run the tests, we will find that all of the test that are ran in parallel will pass. 


### Running Single Tests

We can pass the name of any test function to `cargo test` to run only that test: 

```
$ cargo test one_hundred
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.69s
     Running unittests (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out; finished in 0.00s
```

The only test that ran was the one with the name `one_hundred`; the other two tests didn't match that name. 
The test output lets us know that we had more tests than what this command ran by displaying `2 filtered out` at the end of the summary line. 

We can’t specify the names of multiple tests in this way; only the first value given to `cargo test` will be used. 


## Filtering to Run Multiple Tests

There's a way to run multiple tests. 

We can specify part of a test name, and any test whose name matches that value will be run. 

Example: Since 2 of our tests' names contain `add`, we can run those 2 by running `cargo test add` 

```
$ cargo test add
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s
```

This command ran all tests with `add` in the name and filtered out the test named `one_hundred`. 

<ins>Note: The module in which a test appears becomes part of the test's name, so we can run all the tests in a module by filtering on the module's name.</ins>


## Ignoring Some Tests Unless Specifically Requested 

Sometimes, there are tests that are very time consuming to execute, so we would want to exclude them during most runs of `cargo test`. 
Rather than listing as arguments all the tests that we want to run, we can instead annotate the time-consuming tests using the `ignore` attribute to exclude them. 

File - `src/lib.rs` 

```
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```

After `#[test]` we add the `#[ignore]` line to test that we want to exclude. 
When we run our test, `it_works` runs, but `expensive_test` doesn't: 

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.60s
     Running unittests (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test expensive_test ... ignored
test it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

The `expensive_test` function is listed as `ignored`. 
If we want to run only the ignored tests, we can use `cargo test -- --ignored`: 

```
$ cargo test -- --ignored
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

By controlling which tests run, we can make sure our `cargo test` results will be fast. 
When we're at a point where it makes sense to check the results of the `ignored` tests and we have to time to wait for the results, we can run `cargo test -- --ignored` instead. 
If we want to run all of the tests whether they're ignored or not, we can run `cargo test -- --include-ignored`. 
