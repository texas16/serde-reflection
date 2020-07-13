# serde-reflection: Format Description and Code Generation for Serde

[![Build Status](https://circleci.com/gh/facebookincubator/serde-reflection/tree/master.svg?style=shield&circle-token=4380502426d703f8f000b5467195728e5e8e4ff5)](https://circleci.com/gh/facebookincubator/serde-reflection/tree/master)
[![License](https://img.shields.io/badge/license-Apache-green.svg)](LICENSE-APACHE)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE-MIT)


This project aims to bring the features of a traditional IDL to Rust and Serde.

* [`serde-reflection`](serde-reflection) is a library to extract Serde data formats [![serde-reflection on crates.io](https://img.shields.io/crates/v/serde-reflection)](https://crates.io/crates/serde-reflection) [![Documentation (latest release)](https://docs.rs/serde-reflection/badge.svg)](https://docs.rs/serde-reflection/) [![Documentation (master)](https://img.shields.io/badge/docs-master-59f)](https://facebookincubator.github.io/serde-reflection/serde_reflection/)

* [`serde-generate`](serde-generate) is a library and a tool to generate type definitions and provide (de)serialization in other programming languages [![serde-generate on crates.io](https://img.shields.io/crates/v/serde-generate)](https://crates.io/crates/serde-generate) [![Documentation (latest release)](https://docs.rs/serde-generate/badge.svg)](https://docs.rs/serde-generate/) [![Documentation (master)](https://img.shields.io/badge/docs-master-59f)](https://facebookincubator.github.io/serde-reflection/serde_generate/)

* [`serde-name`](serde-name) is a minimal library to compute Serde names at runtime [![serde-name on crates.io](https://img.shields.io/crates/v/serde-name)](https://crates.io/crates/serde-name) [![Documentation (latest release)](https://docs.rs/serde-name/badge.svg)](https://docs.rs/serde-name/) [![Documentation (master)](https://img.shields.io/badge/docs-master-59f)](https://facebookincubator.github.io/serde-reflection/serde_name/)

The code in this repository is still under active development.


## Quick Start

See [this example](serde-generate/README.md#quick-start-with-python-and-bincode) to transfer data from Rust to Python using the Bincode format.


## Use Cases

### Data Format Specifications

The [Serde](https://serde.rs/) library is an essential component of the Rust ecosystem that provides (de)serialization of Rust data structures in [many encodings](https://serde.rs/#data-formats). In practice, Serde implements the (de)serialization of user data structures using derive macros `#[derive(Serialize, Deserialize)]`.

[`serde-reflection`](serde-reflection) analyzes the result of Serde macros to turn Rust type definitions into a representation of their Serde data layout. For instance, the following definition
```rust
#[derive(Serialize, Deserialize)]
enum Foo { A(u64), B, C }
```
entails a registry containing one [data format](https://facebookincubator.github.io/serde-reflection/serde_reflection/enum.ContainerFormat.html) and represented as follows in YAML syntax:
```
---
Foo:
  ENUM:
    0:
      A:
        NEWTYPE:
          U64
    1:
      B: UNIT
    2:
      C: UNIT
```

This format summarizes how a value of type `Foo` would be encoded by Serde in any encoding. For instance, in Bincode, we deduce that `Foo::B` is encoded as a 32-bit integer `1`.

One difficulty often associated with Serde is that small modifications in Rust may silently change the specifications of the protocol. For instance, changing `enum Foo { A(u64), B, C }` into `enum Foo { A(u64), C, B }` does not break Rust compilation but it changes the serialization of `Foo::B`. Thanks to `serde-reflection`, one can now solve this issue simply by committing Serde formats as a file in the version control system (VCS) and adding a non-regression test ([real-life example](https://github.com/libra/libra/tree/master/testsuite/generate-format/tests)).


### Language Interoperability

The data formats extracted by `serde-reflection` also serve as basis for code generation with the tool [`serde-generate`](serde-generate). Specifically, `serde-generate` takes a registry of Serde data formats as input and translates them into type definitions for other programming languages such as Python and C++.

For instance, the definition of `Foo` above translates into C++ as follows: (omitting methods)
```
struct Foo {
    struct A {
        uint64_t value;
    };
    struct B {};
    struct C {};
    std::variant<A, B, C> value;
};
```

To provide (de)serialization, the code generated by `serde-generate` is completed by runtime libraries in each target language and for each supported binary encoding.


## Benefits

In addition to ensuring an optimal developer experience in Rust, the modular approach based on Serde and `serde-reflection` makes it easy to experiment with new binary encodings. We believe that this approach can greatly facilitate the implementation of distributed protocols and storage protocols in Rust.

This project was initially motivated by the need for canonical serialization and cryptographic hashing in the [Libra](https://github.com/libra/libra) project. In this context, [`serde-name`](serde-name) has been used to provide predictable cryptographic seeds for Rust containers.


## Contributing

See the [CONTRIBUTING](CONTRIBUTING.md) file for how to help out.


## License

This project is available under the terms of either the [Apache 2.0 license](LICENSE-APACHE) or the [MIT license](LICENSE-MIT).