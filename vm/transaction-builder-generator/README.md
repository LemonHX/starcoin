---
id: transaction-builder-generator
title: Transaction Builder Generator
custom_edit_url: https://github.com/diem/diem/edit/master/language/transaction-builder-generator/README.md
---

# Transaction Builder Generator

A *transaction builder* is a helper function that converts its arguments into the payload of a Diem transaction calling a particular Move script.

In Rust, the signature of such a function typically looks like this:
```rust
pub fn encode_peer_to_peer_with_metadata_script(
    token: TypeTag,
    payee: AccountAddress,
    amount: u64,
    metadata: Vec<u8>,
    metadata_signature: Vec<u8>,
) -> Script;
```

This crate provides a binary tool `generate-transaction-builders` to generate and install transaction builders in several programming languages.

The tool will also generate and install type definitions for Diem types such as `TypeTag`, `AccountAddress`, and `Script`.

In practice, hashing and signing Diem transactions additionally requires a runtime library for Diem Canonical Serialization ("LCS").
Such a library will be installed together with the Diem types.


## Supported Languages

The following languages are currently supported:

* Python 3

* C++ 17

* Java 8

* Rust (NOTE: Code generation of dependency-free Rust is experimental. Consider using the libraries of the Diem repository instead.)


## Quick Start

From the root of the Diem repository, run `cargo build -p transaction-builder-generator`.

You may browse command line options with `target/debug/generate-transaction-builders --help`.

### Python

To install Python3 modules `serde`, `lcs`, `libra_types`, and `libra_stdlib` into a target directory `$DEST`, run:
```bash
target/debug/generate-transaction-builders \
    --language python3 \
    --module-name starcoin_stdlib \
    --with-diem-types "etc/starcoin_types.yml" \
    --diem-package-name "starcoin" \
    --serde-package-name "starcoin" \
    --target-source-dir "target/python" \
    "vm/stdlib/compiled/latest/transaction_scripts/abi"
```
Next, you may copy and execute the [Python demo file](examples/python3/stdlib_demo.py) with:
```
cp language/transaction-builder-generator/examples/python3/stdlib_demo.py "$DEST"
PYTHONPATH="$PYTHONPATH:$DEST" python3 "$DEST/stdlib_demo.py"
```

### C++

To install C++ files `serde.hpp`, `lcs.hpp`, `libra_types.hpp`, `libra_stdlib.hpp`, `libra_stdlib.cpp` into a target directory `$DEST`, run:
```bash
target/debug/generate-transaction-builders \
    --language cpp \
    --module-name libra_stdlib \
    --with-diem-types "etc/starcoin_types.yml" \
    --target-source-dir "target/cpp" \
    "vm/stdlib/compiled/latest/transaction_scripts/abi"
```
Next, you may copy and execute the [C++ demo file](examples/cpp/stdlib_demo.cpp) with:
```
cp language/transaction-builder-generator/examples/cpp/stdlib_demo.cpp "$DEST"
clang++ --std=c++17 -I "$DEST" "$DEST/libra_stdlib.cpp" "$DEST/stdlib_demo.cpp" -o "$DEST/stdlib_demo"
"$DEST/stdlib_demo"
```

### Java

To install Java source packages `com.facebook.serde`, `com.facebook.lcs`, `org.diem.types`, and a class `org.diem.stdlib.Stdlib` into a target directory `$DEST`, run:
```bash
target/debug/generate-transaction-builders \
    --language java \
    --module-name org.starcoin.stdlib.Stdlib \
    --with-diem-types "etc/starcoin_types.yml" \
    --target-source-dir "target/java" \
    "vm/stdlib/compiled/latest/transaction_scripts/abi"
```
Next, you may copy and execute the [Java demo file](examples/java/StdlibDemo.java) with:
```
cp language/transaction-builder-generator/examples/java/StdlibDemo.java "$DEST"
(find "$DEST" -name "*.java" | xargs javac -cp "$DEST")
java -cp "$DEST" StdlibDemo
```

### Rust (experimental)

To install dependency-free Rust crates `diem-types` and `diem-stdlib` into a target directory `$DEST`, run:
```bash
target/debug/generate-transaction-builders \
    --language rust \
    --module-name diem-stdlib \
    --with-diem-types "etc/starcoin_types.yml" \
    --target-source-dir "target/rust" \
    "vm/stdlib/compiled/latest/transaction_scripts/abi"
```
Next, you may copy and execute the [Rust demo file](examples/rust/stdlib_demo.rs). (See [unit test](tests/generation.rs) for details.)


## Adding Support for a New Language

Supporting transaction builders in an additional programming language boils down to providing the following items:

1. Code generation for Diem types (Rust library and tool),

2. LCS runtime (library in target language),

3. Code generation for transaction builders (Rust tool).


Items (1) and (2) are provided by the Rust library `serde-generate` which is developed in a separate [github repository](https://github.com/facebookincubator/serde-reflection).

Item (3) --- this tool --- is currently developed in the Diem repository.

Items (2) and (3) are mostly independent. Both crucially depend on (1) to be sufficiently stable, therefore our suggestion for adding a new language is first to open a new github issue in `serde-generate` and contact the Diem maintainers.


The new issue created on `serde-generate` should include:

* requirements for the production environment (e.g. Java >= 8);

* suggestions for a testing environment in CircleCI (if not easily available in Debian 10.4 "buster");

* optionally, suggested definitions for a sample of types in the target language (structs, enums, arrays, tuples, vectors, maps, (un)signed integers of various sizes, etc).
