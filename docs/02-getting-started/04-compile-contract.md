---
sidebar_position: 4
---

# Downloading and Compiling Contract

In this section, we will download a sample contract, and compile to it to a wasm binary executable.

Please first review the [client setup instructions](03-setting-env.md), and configure a client before proceeding. Either
the Node.js REPL or Go CLI will work.

## Compiling and Testing Contract {#compiling-and-testing-contract}

Let's download the repo in which we collect [`cosmwasm-examples`](https://github.com/CosmWasm/cw-examples). We're going
to try out a simple escrow contract. This contract can hold native tokens, and allows an arbiter to release them to a
pre-defined beneficiary. First, clone the repo and try to build the wasm bundle:

```shell
# get the code
git clone https://github.com/CosmWasm/cw-examples
cd cw-examples
git fetch --tags
git checkout escrow-0.10.0
cd escrow

# compile the wasm contract with stable toolchain
rustup default stable
cargo wasm
```

After this compiles, it should produce a file in `target/wasm32-unknown-unknown/release/cw_escrow.wasm`. A quick `ls -l`
should show a file of around 2MB. This is a release build, but not stripped of all unneeded code. To produce a much
smaller version, you can run this, which tells the compiler to strip all unused code out:

```shell
RUSTFLAGS='-C link-arg=-s' cargo wasm
```

This produces a file about 174kB. We use this and another optimizer in the [next section](#optimized-compilation) to
produce an optimised binary to upload to the blockchain. You don't need to worry about running this yourself (unless you
are curious), but you should have an idea of the final size of your contract this way.

## Unit Tests {#unit-tests}

For completeness, let's try running the unit tests:

```shell
RUST_BACKTRACE=1 cargo unit-test
```

After some compilation steps, you should see:

```text
running 5 tests
test contract::tests::cannot_initialize_expired ... ok
test contract::tests::proper_initialization ... ok
test contract::tests::init_and_query ... ok
test contract::tests::handle_refund ... ok
test contract::tests::handle_approve ... ok

test result: ok. 5 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

`RUST_BACKTRACE=1` will provide you with full stack traces on any error, which is super useful. This only works for unit
tests (which test native rust code, not the compiled wasm). Also, if you want to know where `cargo wasm` and `cargo
unit-test` come from, they are just aliases defined in `.cargo/config`. Take a look there to understand the cargo flags
better.

## Optimized Compilation {#optimized-compilation}

To reduce gas costs, the binary size should be as small as possible. This will result in a less costly deployment, and
lower fees on every interaction. Luckily, there is tooling to help with this. You can **optimize production code** using
[cosmwasm/rust-optimizer](https://github.com/CosmWasm/rust-optimizer). **rust-optimizer** produces reproducible builds
of cosmwasm smart contracts. This means third parties can verify the contract is actually the claimed code.

```shell
docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/rust-optimizer:0.11.3
```
