---
title: Chapter 15 Writing Tests
weight: 15
type: docs
---

> This note does not cover all sections of Chapter 15 (code coverage, fuzzing, benchmarks, and race detector).

## Understanding the Basics of Testing

- Go places its tests in the same directory and package as the production code. They are able to access and test unexported functions and variables.
  - Tests are written in files with the `_test.go` suffix. The names are often the same as the files they are testing.
- Test functions start with `Test`, take a single parameter of type `*testing.T` (named `t` by convention), and return nothing.
  - Testing exported functions: `TestFuncName`
  - Testing unexported functions: `Test_funcName`
- `go test` runs all test functions in the current directory.
  - **`go test ./...` runs tests in the current directory and all subdirectories.**
  - `go test -v` provides verbose output.

### Reporting Test Failures

- `t.Error()` and `t.Errorf()` report a test failure but **continue running the current test function**.
- `t.Fatal()` and `t.Fatalf()` report a test failure and **immediately exit the current test function**.
  - Use this if the failure of a check means further checks will always fail/panic.

### Setting Up and Tearing Down

- **Declare a function `TestMain(m *testing.M)` once per the package**. It is called instead of running the tests directly.
  - `m.Run()` runs the test functions in the package and returns the exit code - 0 for all tests passing.
  - `os.Exit()` must be called with the exit code from `m.Run()`.
  - Use this for setting up external data (database) or package-level variables (consider refactoring!).

```go
var testTime time.Time

func TestMain(m *testing.M) {
    fmt.Println("Setting up")
    testTime = time.Now()
    exitVal := m.Run()
    fmt.Println("Tearing down")
    os.Exit(exitVal)
}
```

- **`t.Cleanup()` registers a function to be called when a test function completes.**
  - It can be called multiple times; functions are executed in LIFO order.
  - Use this if tests rely on helper functions to set up temporary data (e.g., creating temporary files).

```go
func createFile(t *testing.T) (_ string, err error) {
    f, err := os.Create("tempFile")
    if err != nil {
        return "", err
    }
    defer func() {
        err = errors.Join(err, f.Close())
    }()
    // Write some data to f
    t.Cleanup(func() {
        os.Remove(f.Name())
    })
    return f.Name(), nil
}
```

- **`t.TempDir()` creates a temporary directory, registers a cleanup function to remove it (along with its contents), and returns its path.**
  - It is usually used along with `os.CreateTemp()` to create temporary files within the temporary directory.

### Testing with Environment Variables

- **`t.Setenv(key, value)` registers an environment variable** that will be reverted to its previous state when the test exits (via a cleanup function).
- Make sure environment variables are abstracted away from main logic. Copy the values into configuration structs at program startup.
  - Consider using Viper, envconfig, or GoDotEnv for managing environment variables.

### Storing Sample Test Data

- Put sample data to test functions in `testdata` subdirectory.
  - Use relative paths (from the current package directory) to access files in `testdata`.

### Caching Test Results

- The tests are recompiled and rerun only if:
  - Files in the package or in the `testdata` directory have changed.
  - Flag `-count=1` is used.

### Testing Your Public API

- **Use `packagename_test` as the package name in test files for exported functions (public API).**
- **Although in the same directory, it needs to import the package being tested.** It can only access exported functions and variables (treat it as a black box).

### Using `go-cmp` to Compare Test Results

- The `cmp` package (`github.com/google/go-cmp/cmp`) provides a way to compare structs in tests.
- Custom comparers must be defined with a symmetric, deterministic, and pure (not modifying its parameters) function.
- [More on go-cmp](https://pkg.go.dev/github.com/google/go-cmp/cmp).

```go
// Strict comparison
if diff := cmp.Diff(expected, result); diff != "" {
    t.Error(diff)
}

// Custom comparers
comparer := cmp.Comparer(func(x, y MyType) bool {
    return x.ID == y.ID
})
if diff := cmp.Diff(expected, result, comparer); diff != "" {
    t.Error(diff)
}
```


## Running Table Tests

- **Use `t.Run()` to create a test case for each set of input data.**

```go
data := []struct {
  name     string
  num1     int
  num2     int
  op       string
  expected int
  errMsg   string
}{
    {"Addition", 2, 3, "+", 5, ""},
    {"Subtraction", 5, 3, "-", 2, ""},
    {"Multiplication", 2, 3, "*", 6, ""},
    {"Division", 6, 3, "/", 2, ""},
    {"Division by zero", 6, 0, "/", 0, "division by zero"},
}

for _, d := range data {
    t.Run(d.name, func(t *testing.T) {
        result, err := DoMath(d.num1, d.num2, d.op)
        if result != d.expected {
            t.Errorf("expected %d, got %d", d.expected, result)
        }
        var errMsg string
        if err != nil {
            errMsg = err.Error()
        }
        if errMsg != d.errMsg {
            t.Errorf("expected error %s, got %s", d.errMsg, errMsg)
        }
    })
}
```


## Running Tests Concurrently

- **Use `t.Parallel()` as the first line in a test function to run it concurrently with other tests.**
  - The test will panic if you mark it as parallel and use `t.Setenv()`.
  - **Be careful when using shared resources.**

```go
for _, d := range data {
    d := d // use this line to shadow d - otherwise it will be captured by the function literal prior to Go 1.22
    t.Run(d.name, func(t *testing.T) {
        t.Parallel()
        fmt.Println("Running test:", d.name)
        // test logic here
    })
}
```


## Using Stubs in Go

> [**Mocks and stubs**](https://martinfowler.com/articles/mocksArentStubs.html): a stub returns a canned value for a given input, while a mock validates that a set of calls happen in the expected order with the expected inputs.

- With code depending on abstractions, you can create stubs to simulate the behavior of real implementations when testing other code.
- To test code that depends on a large interface, you may use the following patterns to create stubs for the interface:
  - Pattern 1: **embed the interface in a struct - this automatically defines (not implements) all methods on the struct**. You may only implement the methods you need.
    - Drawback: this stub does not apply to other tests where different inputs and outputs are needed.
  - Pattern 2: **define a stub struct that proxies method calls to function fields, and define its methods with function fields to implement the interface**. Instances of the stub can be directly used as the interface.


```go
// Suppose we have an interface with many methods
type Entities interface {
    A()
    B()
    C()
    D()
}

// And the code we want to test only calls method A
func ProcessA(e Entities) {
    e.A()
}

// Pattern 1: We can write a stub struct that implements only method A
type AStub struct {
    Entities
}

func (a AStub) A() {
    fmt.Println("AStub.A called")
}

// Pattern 2: We can create a stub struct that proxies method calls to function fields
type EntitiesStub struct {
    a func()
    b func()
    c func()
    d func()
}

func (e EntitiesStub) A() {
    if e.a != nil {
        e.a()
    }
}

// Now we can write tests with different stubs for A()
data := []struct {
  name string
  a    func()
}{
    {"Case 1", func() { fmt.Println("A called in Case 1") }},
    {"Case 2", func() { fmt.Println("A called in Case 2") }},
}
for _, d := range data {
    t.Run(d.name, func(t *testing.T) {
        var entities Entities = EntitiesStub{a: d.a}
        ProcessA(entities)
        // other test logic
    })
}
```


## Using `httptest`

- `net/http/httptest` package allows you to **test HTTP clients with a local server**.

```go
// Define an input type for a test case
// Note: you controls both the server and the client, so input here includes both request and response data
type info struct {
    expression string // client input
    code       int    // response code
    body       string // response body
}
var io info // global variable to hold the current test case

// Define a server
server := httptest.NewServer(
    http.HandlerFunc(func(rw http.ResponseWriter, req *http.Request) {
        // Check the request
        expression := req.URL.Query().Get("expression")
        if expression != io.expression {
            rw.WriteHeader(http.StatusBadRequest)
            fmt.Fprintf(rw, "unexpected expression: got %s, want %s", expression, io.expression)
            return
        }

        // Send the response
        rw.WriteHeader(io.code)
        rw.Write([]byte(io.body))
    }),
)
defer server.Close()

// Define test cases
data := []struct{
    name   string
    io     info
    result float64
    errMsg string
}{
    {"Case 1", info{"2 + 2 * 10", http.StatusOK, "22"}, 22, ""},
}

// Run tests with a client instance
rs := RemoteSolver{
    ServerURL: server.URL,
    Client:    server.Client(),
}
for _, d := range data {
    t.Run(d.name, func(t *testing.T) {
        io = d.io
        result, err := rs.Solve(context.Background(), d.io.expression)
        if result != d.result {
            t.Errorf("expected %f, got %f", d.result, result)
        }
        var errMsg string
        if err != nil {
            errMsg = err.Error()
        }
        if errMsg != d.errMsg {
            t.Errorf("expected error %s, got %s", d.errMsg, errMsg)
        }
    })
}
```


## Using Integration Tests and Build Tags

- Build tags (e.g., `//go:build integration`) may be used to specify code for integration tests.

```bash
go test -tags integration -v ./...
```

- Some developers are against using build tags for integration tests (hard to find out build tags to use) in favor of environment variables: **check an environment variable in each integration test and use `t.Skip()` to skip the test with a detailed message** if the variable is not set.
- `go test -short` may be used to skip long-running tests if slow tests are marked in this way:

```go
if testing.Short() {
    t.Skip("skipping test in short mode.")
}
```

