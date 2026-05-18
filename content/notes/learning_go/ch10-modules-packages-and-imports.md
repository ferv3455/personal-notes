---
title: Chapter 10 Modules, Packages, and Imports
weight: 10
type: docs
---

> This note only covers the first three sections of Chapter 10 - how to build packages.

## Repositories, Modules, and Packages

- A **module** is a bundle of Go source code that's distributed and versioned as a single unit. Modules are stored in a repository (usually one-to-one) for version control.
  - **Every (non-local) module has a globally unique identifier - module path.** It is usually based on the repository location.
- Modules consist of one or more **packages**, which are directories of source code.


## Using `go.mod`

- **A valid `go.mod` makes a directory tree a module.**
- `go mod init <module-path>` creates a `go.mod` file that makes the current directory the root of a module.


### Use the `go` Directive to Manage Go Build Versions

- If the `go` directive specifies a version newer than installed, Go 1.20 or earlier will ignore the newer version, and Go 1.21 or later will download the newer version of Go to build the code by default. You may control this behavior with `toolchain` directive or `GOTOOLCHAIN` environment variable. Check [Go Toolchains](https://go.dev/doc/toolchain) for details.


### The `require` Directive

- The require directives list the modules that your module depends on and the **minimum** version required.
  - The first `require` section lists the direct dependencies, and the second `require` section lists the indirect dependencies with a comment `// indirect`.




## Building Packages

### Importing and Exporting

- A package-level identifier whose name starts with an uppercase letter is exported. Otherwise, it is visible only within the package.
  - Everything you export is part of the package's API.

### Creating and Accessing a Package

- Minimal example of a package: [package_example](https://github.com/learning-go-book-2e/package_example)
- Every Go file in a directory must have an identical package clause.
- An **import/package path** consists of the module path followed by **the path to the package** within the module (not package name).
  - It is a compile-time error to import a package but not use it.
  - **Package names in `import` statements are in the file block.**
- Imported functions use the **package name** as the prefix (not the package path).
- **Style convention: make the package name match the name of the directory.** Exceptions:
  - The `main` package.
  - The directory name contains a character not valid in a Go identifier (e.g., `-`) - avoid this directory name in the first place.
  - Support versioning using directories. Refer to "Versioning Your Module" section (WIP).

### Naming Packages

- Think about parts of speech - a function or method would be a verb/action word, while a package would be a noun.
- **Avoid repeating the name of the package in the names of functions and types.** Exception: the identifier is the same as the package name (e.g., `sort`).

### Overriding a Package Name

- You may override the package name in `import`: `crand "crypto/rand"`.
  - `.`: imports all exported identifiers into the current package's namespace. Discouraged.
  - `_`: imports solely for its side effects (e.g., `init`)
- Package names can be shadowed. Override the package name to resolve the conflict, but avoid shadowing in general.

### Documenting Your Code with Go Doc Comments

- *Go Doc* format:
  - Place the comment directly before the item being documented.
  - Start with `// ` instead of `/* ... */`.
  - The first word should be the name of the symbol. You can also use "A" or "An" to make the comment grammatically correct.
  - Use a blank comment line to break comments into paragraphs.
- Make it look prettier:
  - Preformatted content: put an additional space after `//`.
  - Header: use `#`. **Multiple `#` for different levels is NOT supported.**
  - Link to another package: `[package-path]`.
  - Link to an exported symbol: `[SymbolName]` or `[pkgName.SymbolName]`.
  - Raw URL: automatically converted into a link.
  - **Hyperlink: `[text]`. At the end of the comment block, declare tha mappings between text and URLs with `// [text]: URL`.**
- Comments before the package declaration create package-level comments. **If it is too long, put it in `doc.go`.**
- `go doc <package-name>` or `go doc <package-name>.<identifier-name>` displays the documentation for a package/identifier.
- Running `pkgsite` (`go install golang.org/x/pkgsite/cmd/pkgsite@latest`) at the root of the module renders the documentation as a website.
- More on Go documentation: [Go Doc Comments](https://go.dev/doc/comment).

### Using the `internal` Package

- **The special `internal` package can only be imported by its direct parent package and its sibling packages.**

### Avoiding Circular Dependencies

- Circular dependencies: you may have split packages too finely.

### Organizing Your Module

- [**Simple Go project layout with modules**](https://eli.thegreenplace.net/2019/simple-go-project-layout-with-modules/)
- When your module is small, keep all your code in a single package.
- Single application: make the root the `main` package, and **place all your logic in `internal`**.
- Library: give the root package a name matching the repository, and **put all non-API code in `internal`**.
  - **Utility applications included in a library: place them in `/cmd/<app-name>`.**
- Break up packages by slices of **functionality** (e.g., customer management, inventory management) to limit dependencies among packages, in contrast to **implementation** (e.g., business logic, database logic, data models). This also makes it easier to refactor it into microservices.

> *Hyrum's Law: with a sufficient number of users of an API, all observable behaviors of that API will be depended on by someone.* Once something is part of your API, you need to continue supporting it until you decide to break compatibility (major version change).

### Gracefully Renaming and Reorganizing Your API

- To avoid a backward-breaking change, if you want to rename/move some exported identifiers, keep the original identifiers and provide an alternate name instead.
  - For types, define an alias: `type AliasName = TypeName`. Unlike type definition, it does not create a new type, and the alias can be assigned to the original type directly.
    - **You cannot use alias to refer to the unexported methods or fields of the original type.**

### Avoiding the `init` Function if Possible

- The `init` function is run the first time the package is referenced.
  - A package or even a single file may have multiple `init` functions - avoid this (there is a documented order though).
- The primary use of `init` is to initialize package-level variables/states. They are mutable states that are effectively immutable (no way to enforce immutability after `init`).
  - Obsolete: it's unclear what specific operation needs to be performed.
  - **Alternative: put these states into a struct that's initialized and returned by a function.** They can be used as a *handle* to the package.
- **Document the non-explicit invocation of `init` functions in a package-level comment.**




