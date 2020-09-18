# Software Development Plan

## Introduction

This document describes software development process for Yagna project and technical requirements related to its source code (technology stack, supported platforms, coding standards).

This document should be updated when project development process or source code requirements change.

## Goals and Scope

### Software Description

Yagna will be used to provide computing power to clients.
It will run WASM code and measure CPU time spent on computations.

### Project Artifacts

| Platform | Artifacts |
|--|--|
| Linux | Debian/Ubuntu Package (.deb) |
| macOS | Apple Disk Image (.dmg) |
| Windows | Windows Installer (.msi) |

## Development Process

### Feature Specification

GitHub Issues should contain specifications of all new features. Every issue should be assigned to a developer.
Milestones should be used to pin issues to specific releases.

Labels should be used to filter issues by other criteria:

| Label | Description |
|--|--|
| enhancement | A new feature that can be implemented. |
| bug | A bug report. |
| in progress | Work on this issue is in progress. |
| duplicate | A very similar issue already exists. |
| deferred | Will be done later, after other issues are closed. |

### Working on Git Branches

Software development should be done on Git branches.

| Branch Name or Prefix | Meaning |
|--|--|
| master | Main branch. |
| feature/ | New feature, e.g. feature/connection-manager. |
| bugfix/ | Bug fix, e.g. bugfix/division-by-zero. |
| release/ | Branch for a special release, e.g. release/3.0. |

### Pull Requests

After work on a feature is finished, a pull request based on the branch where the work happened must be created on GitHub. At least one code reviewer must be added to the pull request.

### Automatic Testing

Every branch is automatically compiled and tested in Jenkins.

| Test Name | Requirement |
|--|--|
| Compilation | All code must compile without errors. |
| Unit Tests | All tests (prefixed with `#[cfg(test)]`) should pass. |
| Code Formatting | All code must be formatted with rustfmt (`cargo fmt`). |

### Code Review and Merging

At least one reviewer must review the code related to a new pull request.
After the review is complete and all the automated tests pass, the code could be merged.

Interesting guidelines for code review:
https://phauer.com/2018/code-review-guidelines/

### Automatic Builds

The project artifacts (i.e. installation packages and standalone/portable binaries) 
should automatically build in Jenkins for every supported operating system (Linux, macOS, Windows).

### Bug Reporting

Bugs should be reported in GitHub Issues.

## Source Code Requirements

### Technology Stack

The programming language used in this project will be Rust (https://www.rust-lang.org/).
The newest stable version of Rust compiler (`rustc`) should compile all source code without errors.

For HTTP client/server code, Actix Web 1.0 (https://actix.rs) will be used.

### Supported Platforms

All code should compile and run on Linux, macOS and Windows.

The main development platform is Ubuntu Linux, but all code should be portable. E.g. instead of using "/tmp", use `std::env::temp_dir()` function; instead of using platform-native functions, use `std::env::current_exe()` to find the path of the current executable.

If this is impossible, use `#[cfg(unix)]` to target Unix platforms and `#[cfg(windows)]` to target Windows. To target only macOS, use `cfg!(target_os = "macos")`.

### Coding Standards

Rust coding style guidelines:

https://doc.rust-lang.org/1.0.0/style/README.html

To enforce formatting, code should be formatted using rustfmt tool (https://github.com/rust-lang/rustfmt).
To install it, run `rustup component add rustfmt` command. To format files in the working dictory, please run `cargo fmt` command.

### Code Repositories

Most Rust crates used in the project should be located in one repository.
For every crate, this repository should contain a crate subdirectory with a `Cargo.toml` file and a `src` directory.

### Versioning

Semantic Versioning (MAJOR.MINOR.PATCH, https://semver.org/) should be used for version numbers.

### Project Dependencies

Project dependencies of each crate are specified in `Cargo.toml` file.
The dependencies should not be spontaneously updated by developers.

### Usage Examples

If a crate needs usage examples, they should be placed in the `examples` subdirectory of the crate. To run such example, 
please run `cargo run -p <package name> --example <example name>` command.

### Tests

Tests can be run using `cargo test --workspace` command:

https://doc.rust-lang.org/book/ch11-01-writing-tests.html

To create a test, open a `tests` module prefixed with `#[cfg(test)]`, add test functions and prefix them with `#[test]`.

### Documentation

Documentation will be automatically generated using rustdoc:

https://doc.rust-lang.org/rustdoc/index.html

To generate documentation, please enter `cargo doc --workspace --no-deps` command in your shell.

The comments that are copied to the documentation are prefixed with `///`, Markdown is supported.
