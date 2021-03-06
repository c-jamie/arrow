<!---
  Licensed to the Apache Software Foundation (ASF) under one
  or more contributor license agreements.  See the NOTICE file
  distributed with this work for additional information
  regarding copyright ownership.  The ASF licenses this file
  to you under the Apache License, Version 2.0 (the
  "License"); you may not use this file except in compliance
  with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing,
  software distributed under the License is distributed on an
  "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  KIND, either express or implied.  See the License for the
  specific language governing permissions and limitations
  under the License.
-->

# Native Rust implementation of Apache Arrow

[![Coverage Status](https://coveralls.io/repos/github/apache/arrow/badge.svg)](https://coveralls.io/github/apache/arrow)

This crate contains a native Rust implementation of the [Arrow columnar format](https://arrow.apache.org/docs/format/Columnar.html). It uses nightly Rust.

## Developer's guide

Here you can find general information about this crate's content and its organization.

### DataType, Field, Schema and RecordBatch

Every array in Arrow has a data type, that specifies how the data should be layed in memory and casted, and an optional null bitmap, that specifies whether each value is null or not.
Thus, a central enum of this crate is `arrow::datatypes::DataType`, that contains the set of valid
DataTypes in the specification. For example, `arrow::datatypes::DataType::Utf8`.

Many (but not all) data types have an associated Rust native type. The trait that represents 
this relationship is `arrow::datatypes::ArrowNativeType`, that most native types implement.

`arrow::datatypes::Field` is a struct that contains an arrays' metadata (datatype and whether its values
can be null), and a name. `arrow::datatypes::Schema` is a vector of fields with optional metadata.

Finally, `arrow::record_batch::RecordBatch` is a struct with a `Schema` and a vector of `Array`s all with the same `len`. A record batch is the highest order struct that this crate currently offers.

### Array

The central trait of this package is `arrow::array::Array`, a dynamically-typed trait that
can be downcasted to specific implementations, such as `arrow::array::UInt32Array`.

`Array` has `Array::len()`, `Array::data_type()`, and nullability of each of its entries, that can be obtained via `Array::is_null(index)`. To downcast an `Array` to a specific implementation, you can use

```rust
let specific_array = array.as_any().downcast_ref<UInt32Array>().unwrap();
```

Once downcasted, it offers two calls to retrieve specific values (and nullability):

```rust
let is_null_0: bool = specifcic_array.is_null(0)
let value_at_0: u32 = specifcic_array.value(0)
```

### Memory and Buffers

You can access the whole buffer of an `Array` via `Array::data()`, which returns an `arrow::data::ArrayData`. This struct holds the array's `DataType`, `arrow::buffer::Buffer`s, and `childs` (which are themselves `ArrayData`).

The central structs that array implementations use to allocate and refer to memory
aligned according to the specification are the `arrow::buffer::Buffer` and `arrow::buffer::MutableBuffer`.
These are the lowest abstractions of this crate, and are used throughout the crate to 
efficiently allocate, write, read and deallocate memory.

This implementation uses a architecture-dependent alignment of sizes that are multiples of 64 bytes.

### Compute

This crate offers many operations (called kernels) to operate on `Array`s, that you can find at `arrow::compute::kernels`.

## Status

The current status is:

- [x] Primitive Arrays
- [x] List Arrays
- [x] Struct Arrays
- [x] CSV Reader
- [X] CSV Writer
- [X] JSON Reader
- [ ] Parquet Reader
- [ ] Parquet Writer
- [X] Arrow IPC
- [ ] Interop tests with other implementations

## Examples

The examples folder shows how to construct some different types of Arrow
arrays, including dynamic arrays created at runtime.

Examples can be run using the `cargo run --example` command. For example:

```bash
cargo run --example builders
cargo run --example dynamic_types
cargo run --example read_csv
```

## IPC

The IPC flatbuffer code was generated by running this command from the root of the project, using flatc version 1.10.0:

```bash
./regen.sh
```

The above script will run the `flatc` compiler and perform some adjustments to the source code:

- Replace `type__` with `type_`
- Remove `org::apache::arrow::flatbuffers` namespace
- Add includes to each generated file

## Features

Arrow uses the following features:

* `simd` - Arrow uses the [packed_simd](https://crates.io/crates/packed_simd) crate to optimize many of the
 implementations in the [compute](https://github.com/apache/arrow/tree/master/rust/arrow/src/compute) module using SIMD
 intrinsics. These optimizations are turned *off* by default.
* `flight` which contains useful functions to convert between the Flight wire format and Arrow data
* `prettyprint` which is a utility for printing record batches

Other than `simd` all the other features are enabled by default. Disabling `prettyprint` might be necessary in order to
compile Arrow to the `wasm32-unknown-unknown` WASM target.

# Publishing to crates.io

An Arrow committer can publish this crate after an official project release has
been made to crates.io using the following instructions.

Follow [these
instructions](https://doc.rust-lang.org/cargo/reference/publishing.html) to
create an account and login to crates.io before asking to be added as an owner
of the [arrow crate](https://crates.io/crates/arrow).

Checkout the tag for the version to be released. For example:

```bash
git checkout apache-arrow-0.11.0
```

If the Cargo.toml in this tag already contains `version = "0.11.0"` (as it
should) then the crate can be published with the following command:

```bash
cargo publish
```

If the Cargo.toml does not have the correct version then it will be necessary
to modify it manually. Since there is now a modified file locally that is not
committed to GitHub it will be necessary to use the following command.

```bash
cargo publish --allow-dirty
```
