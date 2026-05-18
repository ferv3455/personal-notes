---
title: Chapter 1 Setting Up Your Go Environment
weight: 1
type: docs
---

## Installing the Go Tools

- Be sure to include the Go binary (e.g. `/usr/local/go/bin/go`, `/opt/homebrew/bin/go`) in the executable path.
- [gvm](https://github.com/moovweb/gvm) (Go Version Manager) can be used to manage multiple Go versions on your machine.
- The Go team is committed to backward compatibility: Go releases tend to be incremental.

## The Go Workspace

- Third-party Go tools installed via `go install` are placed in a single workspace. By default, this workspace is located in `$HOME/go`. This can be changed by setting the `$GOPATH` environment variable.
  - Source code is in `$GOPATH/src` and binaries are in `$GOPATH/bin`.
- It is recommended to explicitly define `$GOPATH` and add `$GOPATH/bin` to the executable path.
- `go env` lists all Go environment variables. Some of these will be covered in Chapter 9 (modules).

## The `go` Command

### `go run` and `go build`

- **Each of these commands takes either a single Go file, a list of Go files, or the name of a package.**
- `go run <file>` builds the binary in a temporary directory, executes it from there, and then deletes the binary after the program finishes.
  - This treats the Go program like a script.
- `go build <file>` creates an executable in the current directory. The name matches the file or package name by default. The `-o` flag can be used to specify a different name.
  - This creates a binary distributed for other people to use.
  - **Go runtime is compiled into the binary**, so there are no external dependencies.

### Getting Third-Party Go Tools

- There are no centrally hosted services for Go tools. **Go projects are shared via source code repositories.**
- `go install <path>@<version>` downloads, compiles, and installs the tool into `$GOPATH/bin`:
  - `<path>` is the location of the repository.
  - `<version>` defines the version to install. `latest` gets the latest version.
  - **Rerunning the same command updates the tool to the specified version.**
- `go install` may download from a proxy server depending on the repository and the `$GOPROXY` environment variable. If it downloads from a repository, it relies on the command-line tools (e.g. Git).

### Formatting Your Code

- **Go enforces a standard code format to make it easier to write tools that manipulate source code.**
  - Go programs use tabs to indent.
  - It is a syntax error if the opening brace is not on the same line as the declaration or command that begins the block.
- `go fmt` reformats the code to match the standard format.
- `goimports` (`golang.org/x/tools/cmd/goimports`) is an enhanced version that also cleans up the import statements - putting them in alphabetical order, removing unused imports, and guessing missing imports.
  - `goimports -l -w .` scans all files **recursively** in the current directory: `-l` lists files that need formatting, and `-w` modifies the files in place.
- **Go requires a semicolon at the end of every statement, but it is added by the Go compiler based on the semicolon insertion rule:** insert a semicolon if the last token before a newline is any of the following:
  - An identifier
  - A basic literal
  - One of the tokens: `break`, `continue`, `fallthrough`, `return`, `++`, `--`, `)`, or `}`


## Linting and Vetting

- Read [Effective Go](https://go.dev/doc/effective_go) and [Go Wiki: Go Code Review Comments](https://go.dev/wiki/CodeReviewComments) for idiomatic Go code.
- `golint` (`golang.org/x/lint/golint`) ensures the code **follows style guidelines** (e.g. naming conventions, formatting error message, comments).
  - `golint ./...` runs it over the entire project.
  - `golint` might have false positives that do not need to be fixed.
- `go vet` **detects errors that the compiler does not catch** (e.g. wrong number of parameters, unused variables, unreachable code).
  - `go vet ./...` runs it over the entire project.
- `golangci-lint` combines multiple code quality tools. You can configure which linters are enabled and which files to analyze in a configuration file `.golangci.yml`.
  - Since it runs multiple tools, make sure your team agrees on the rules to enforce.


## Makefiles

- Each possible operation in a Makefile is called a *target*. `make [<target>]` runs the commands associated with that target.
  - `.DEFAULT_GOAL` defines which target to run when no target is specified. It is usually set to `build` or `all`.
  - **The word before the colon is the target name. Any words after the colon are the other targets (dependencies) that must be run beforehand.**
  - The tasks to be performed are on indented lines after the target.
  - `.PHONY` indicates that the target is not a file name (if there is a directory/file with the same name as the target).

```makefile
.DEFAULT_GOAL := build

fmt:
  go fmt ./...
.PHONY: fmt

build: fmt
  go build hello.go
.PHONY: build
```
