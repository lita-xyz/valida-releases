# Welcome to Valida!

This repository provides the binary release of the Valida toolchain, designed to enable developers to build, run, and prove programs using the Valida zkVM. Whether you're working in Rust or C, the Valida toolchain makes it easy to compile your code, generate proofs of execution, and verify them. This release supports multiple platforms, including x86 and ARM64, with Docker-based installation as the primary installation method for non-x86 platforms.

Dive into the sections below to learn how to install, use, and report any issues with the Valida toolchain.

---

See 'Releases' section for downloads.

# Docker-Based Installation

We provide a Docker container with the Valida LLVM and Rust toolchains already installed.
Docker is the only supported method of running on platforms other than x86 Linux.

### x86_64-based platforms

To install and use the toolchain via Docker on a 64-bit computer with an Intel-compatible chipset (x86_64), such as Intel- or AMD-based computers:

```bash
# Download the container
docker pull ghcr.io/lita-xyz/llvm-valida-releases/valida-build-container:v0.8.0-alpha-amd64

# cd your-valida-project

# Enter the container:
docker run --platform linux/amd64 -it --rm -v $(realpath .):/src ghcr.io/lita-xyz/llvm-valida-releases/valida-build-container:v0.8.0-alpha-amd64

# You are now in a shell with the valida rust toolchain installed!
```

### ARM64-based platforms

To install and use the toolchain via Docker on a 64-bit computer with an ARM64-compatible chipset (ARM64), such as Apple silicon-based computers:

```bash
# Download the container
docker pull ghcr.io/lita-xyz/llvm-valida-releases/valida-build-container:v0.8.0-alpha-arm64

# cd your-valida-project

# Enter the container:
docker run --platform linux/arm64 -it --rm -v $(realpath .):/src ghcr.io/lita-xyz/llvm-valida-releases/valida-build-container:v0.8.0-alpha-arm64

# You are now in a shell with the valida rust toolchain installed!
```

# Non-Docker-Based Installation (x86_64 Linux only)

For instructions for installation on certain x86_64 Linux platforms, such as Ubuntu 24.04 LTS or later, see [the release notes for the latest release](https://github.com/lita-xyz/valida-releases/releases/tag/v0.8.0-alpha).

# Usage instructions

## Entering the Valida shell (non-Docker only)

**This step applies only if you are using a non-Docker installation of the toolchain.**

Upon having installed the toolchain, the Valida shell should be on your `PATH`, and if you run `which valida-shell`, you should see:

```bash
$ which valida-shell
/usr/local/bin/valida-shell
```

If the result is something else, then either the installation did not complete successfully, or you had another `valida-shell` executable somewhere on your `PATH`.

If you run `valida-shell`, then you should see a shell prompt that reads `valida> `. You should then have on your `PATH` all of the executables from the Valida toolchain needed to follow the instructions below.

## Compiling and running Rust programs

For examples of how to build a Rust program which compiles and runs on Valida, see [lita-xyz/rust-examples on GitHub](https://github.com/lita-xyz/rust-examples). You can use any of these examples as a starting point for developing your own programs using the Valida toolchain. Here are steps for doing so:

1. Clone the project template:

```bash
git clone https://github.com/lita-xyz/fibonacci.git
```

2. `cd` into the project template:

```bash
cd fibonacci
```

3. Enter the Valida shell (skip this step if you are using the Docker toolchain):

```bash
valida-shell
```

4. Build the project:

```
cargo +valida build
```

5. Run the code (taking input from `stdin`):

```
valida run --fast target/valida-unknown-baremetal-gnu/debug/fibonacci log
```

6. Prove the execution (taking input from `stdin`):

```
valida prove target/valida-unknown-baremetal-gnu/debug/fibonacci proof
```

7. Verify the proof:

```
valida verify target/valida-unknown-baremetal-gnu/debug/fibonacci proof log
```

## Writing Rust programs to run on Valida

For a starting point to build a project using the Rust Valida toolchain, please take a look at
[the template project](https://github.com/lita-xyz/fibonacci). You can clone this repo and use
it as a starting point for your project. You should be able to write Rust programs more or less
as normal. There are a few limitations to keep in mind:

 * All of the usual operating system facilities are unavailable, except for standard in (`stdin`)
   and standard out (`stdout`). So for example, there is no access to command line arguments,
   environment variables, networking, or the filesystem.
 * Multi-threading is not supported.
 * Interactive programs may not work as expected.

### An example

Here is an example program using Valida, which computes Fibonacci numbers:

```c
use std::io::stdin;

pub fn main() {
    println!("Please enter a number from 0 to 46:");
    let n = loop {
        let mut input = String::new();
        // Read a line from stdin and parse it as an u8.
        match stdin().read_line(&mut input) {
            Ok(_) => {
                match input.trim().parse::<u8>() {
                    Ok(num) => {
                        if num == 0 {
                            println!("The 0th fibonacci number is: 0");
                            return;
                        } else if num > 46 {
                            println!("Error: n is too large. Please enter a number no larger than 46.");
                        } else {
                            break num;
                        }
                    },
                    Err(e) => {
                        println!("Error reading input: {}. Please try again:", e);
                    }
                }
            }
            Err(e) => {
                println!("Error reading input: {}. Please try again:", e);
            }
        }
    };
    let mut a: u32 = 0;
    let mut b: u32 = 1;
    let mut sum: u32;
    for _ in 1..n {
        sum = a + b;
        a = b;
        b = sum;
    }
    println!("The {}-th fibonacci number is: {}", n, b);
}
```

### More examples in Rust

The following examples are available under `/valida-toolchain/examples/rust`:
 * `conway`: Conway's game of life
 * `ed25519`: ECDSA Ed25519 signature verification
 * `factorial`: The factorial function
 * `fibonacci`: The Fibonacci sequence
 * `fizzbuzz`: The classic fizz-buzz interview problem
 * `grep`: Search text for a substring
 * `guessing_game`: An interactive number guessing example
 * `hello_world`: The classic "hello world" example
 * `json_contains`: JSON parsing and property fetching
 * `palindrome`: Test if a string is a palindrome
 * `prime_factorization`: Check prime factorization
 * `secp256k1`: ECDSA Secp256k1 signature verification
 * `sha256`: SHA-256 hashing
 * `simple_calculator`: A simple calculator app
 * `sudoku`: Checking solutions to Sudoku problems
 * `unit_tests`: A suite of tests of basic language functionality

The [reva](https://github.com/lita-xyz/reva) example executes Ethereum blocks in Valida. This is a work in progress and may produce results that are incorrect. This is plausibly the most complex program that has been run in Valida so far.

If you have installed the toolchain, see `/valida-toolchain/examples/rust` for some more examples of Rust Valida projects.

## Compiling and running C programs

See `/valida-toolchain/examples/c/` for some examples of C programs which can be compiled and run on Valida. Here is an example C program from this release bundle, called `/valida-toolchain/examples/c/cat.c`:

```c
#include <stdio.h>

const unsigned EOF = 0xFFFFFFFF;

int main() {
    unsigned c = 0;
    while (1) {
        c = getchar();
        if (c == EOF) {
            break;
        } else {
            putchar(c);
        }
    }
}
```

To compile, for example, the `cat.c` example, after installing the toolchain, and with the toolchain on your `PATH` (such as, in the `valida-shell` or in the Docker container shell):

```bash
clang -target valida /valida-toolchain/examples/c/cat.c -o cat
valida run cat log
```

Once running, the cat example will wait for input. After you are done providing input, press `ctrl+D`. The program should echo back what you wrote, writing its output to log.

Compiling and running the other examples follows the same procedure, substituting `$NAME` for the name of the example:

```bash
clang -target valida /valida-toolchain/examples/${NAME}.c -o ${NAME}
valida run ${NAME} log
```

Some other C examples that are provided in this release bundle:

 * `reverse.c` will output its reversed input.
 * `checksum.c` will output a checksum, i.e., a sum of the characters, of its input.
 * `merkle-path.c` will verify an opening proof for a SHA256 binary Merkle tree
    * For an example proof you can use as input, see `examples/example-merkle-proof`
 * `sha256.c` will output a SHA-256 hash of the first 256 bytes of its input.
 * `sha256_32byte_in.c` will output the SHA-256 hash of a constant array of 32 bytes. This is used as a benchmark.

### Using `libc`

There is a partial `libc` for Valida, bundled with this release. This `libc` is a version of [LLVM `libc`](https://libc.llvm.org/).

There is an example, `/valida-toolchain/examples/cat-alpha.c`, which makes use of this `libc`. This example echoes all of the alphabetic characters in its input. It makes use of the `libc` function `isalpha`. The following commands, run from this directory, should compile and run this example:

```bash
clang -target valida /valida-toolchain/examples/cat-alpha.c -o cat-alpha
valida run cat-alpha log
```

[See the docs for more details](https://lita.gitbook.io/lita-documentation/usage/using-llvm-libc) on using the bundled version of `libc` for Valida.

## Compiling and running WASM on Valida

The examples at `/valida-toolchain/examples/wasm` demonstrate WebAssembly
to Valida compilation.  They are present at the stated path on a system where
the toolchain is installed, including in an instance of the toolchain Docker image.

The `io-program-via-rust` example is a Rust program that parses an integer
from standard input, squares it and prints the result to standard output.
The program is compiled to `.wasm` file with `wasm32-wasip1` Rust target.
Rust is only used to conveniently generate `.wasm` assembly file.

The `exit-1-program-via-rust` example demonstrates a program which exits
with an exit code 1.

The `fibonacci-via-rust` example computes n-th fibonacci number.

To compile and run the examples execute the script: `./compile_run.sh`.

It's assumed that Valida toolchain is installed and is located at default
installation location: `/valida-toolchain`.

It's assumed that `wasm2c` and `wat2wasm` in version 1.0.34 are in PATH.
On Ubuntu 24.04 these can be installed with a command: `apt install wabt`.

All the examples use `/valida-toolchain/bin/compile-wasm.sh` to compile a
`.wasm` file into a valida executable. See [Valida Gitbook](https://lita.gitbook.io/lita-documentation/quick-start/valida-compiler-toolchain/wasm-usage) for more details
about WASM support and the usage of the script.

# Reporting issues

If you have any issues to report, please report them at [the llvm-valida-releases issue tracker](https://github.com/lita-xyz/llvm-valida-releases/issues). Please include the following elements in your bug report: what release version you encountered the bug on, steps to reproduce, expected behavior, and actual behavior.

# Known issues

 * The prover is unsound, which means that verifying a proof does not provide completely convincing evidence that the statement being proven is true. This will be resolved once some missing constraints are added.
 * There are some issues with standard I/O functions in Rust when it comes to the behavior of interactive programs. Sometimes, code behaves differently in Valida vs native code, such as sometimes needing ctrl+D to be pressed twice instead of once to signal end of input.
 * Some Rust programs are compiled incorrectly when compiled to Valida in release mode.
