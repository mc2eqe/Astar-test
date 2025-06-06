![astar-cover](https://user-images.githubusercontent.com/40356749/135799652-175e0d24-1255-4c26-87e8-447b192fd4b2.gif)

<div align="center">

[![Integration Action](https://github.com/AstarNetwork/Astar/workflows/Integration/badge.svg)](https://github.com/AstarNetwork/Astar/actions)
[![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/AstarNetwork/Astar)](https://github.com/AstarNetwork/Astar/tags)
[![Substrate version](https://img.shields.io/badge/Substrate-3.0.0-brightgreen?logo=Parity%20Substrate)](https://substrate.dev/)
[![License](https://img.shields.io/github/license/AstarNetwork/Astar?color=green)](https://github.com/AstarNetwork/Astar/blob/production/shiden/LICENSE)
 <br />
[![Twitter URL](https://img.shields.io/twitter/follow/AstarNetwork?style=social)](https://twitter.com/AstarNetwork)
[![Twitter URL](https://img.shields.io/twitter/follow/ShidenNetwork?style=social)](https://twitter.com/ShidenNetwork)
[![YouTube](https://img.shields.io/youtube/channel/subscribers/UC36JgEF6gqatVSK9xlzzrvQ?style=social)](https://www.youtube.com/channel/UC36JgEF6gqatVSK9xlzzrvQ)
[![Docker](https://img.shields.io/docker/pulls/staketechnologies/astar-collator?logo=docker)](https://hub.docker.com/r/staketechnologies/astar-collator)
[![Discord](https://img.shields.io/badge/Discord-gray?logo=discord)](https://discord.gg/astarnetwork)
[![Telegram](https://img.shields.io/badge/Telegram-gray?logo=telegram)](https://t.me/PlasmOfficial)
[![Medium](https://img.shields.io/badge/Medium-gray?logo=medium)](https://medium.com/astar-network)

</div>

Astar Network is an interoperable blockchain based the Substrate framework and the hub for dApps within the Polkadot Ecosystem.
With Astar Network and Shiden Network, people can stake their tokens to a Smart Contract for rewarding projects that provide value to the network.

For contributing to this project, please read our [Contribution Guideline](./CONTRIBUTING.md).

## Building From Source

> This section assumes that the developer is running on either macOS or Debian-variant operating system. For Windows, although there are ways to run it, we recommend using [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10) or from a virtual machine for stability.

Execute the following command from your terminal to set up the development environment and build the node runtime.

```bash
# install Substrate development environment via the automatic script
$ curl https://getsubstrate.io -sSf | bash -s -- --fast

# clone the Git repository
$ git clone --recurse-submodules https://github.com/AstarNetwork/Astar.git

# change current working directory
$ cd Astar

# compile the node
# note: you may encounter some errors if `wasm32-unknown-unknown` is not installed, or if the toolchain channel is outdated
$ cargo build --profile production

# show list of available commands
$ ./target/production/astar-collator --help
```

### Building with Nix

```bash
# install Nix package manager:
$ curl https://nixos.org/nix/install | sh

# run from root of the project folder (`Astar/` folder)
$ nix-shell -I nixpkgs=channel:nixos-21.05 third-party/nix/shell.nix --run "cargo build --release"
```

## Running a Collator Node

To set up a collator node, you must have a fully synced node with the proper arguments, which can be done with the following command.

```bash
# start the Shiden collator node with
$ ./target/release/astar-collator \
  --base-path <path to save blocks> \
  --name <node display name> \
  --port 30333 \
  --rpc-port 9944 \
  --telemetry-url 'wss://telemetry.polkadot.io/submit/ 0' \
  --rpc-cors all \
  --collator
```

Now, you can obtain the node's session key by sending the following RPC payload.

```bash
# send `rotate_keys` request
$ curl -H 'Content-Type: application/json' --data '{ "jsonrpc":"2.0", "method":"author_rotateKeys", "id":1 }' localhost:9933

# should return a long string of hex, which is your session key
{"jsonrpc":"2.0","result":"<session key in hex>","id":1}
```

After this step, you should have a validator node online with a session key for your node.
For key management and validator rewards, consult our [validator guide online](https://docs.astar.network/build/validator-guide/configure-node).

## Versioning

### Historical Versioning (Up to `v5.44.0`)

Up to the release `v5.44.0`, **Astar** releases contained both the client & the runtime blobs.
In general, each release contained both, with some specific releases (related to fixes) which only released e.g. client or runtime.
Standard semantic versioning approach was used.

### New Versioning Approach (From `v5.45.0`)

Starting with v5.45.0, the release process has been split into separate client and runtime releases, each following distinct versioning schemes:

The **client release** will continue to follow semantic versioning, continuing where the former approach left off.
E.g. the next expected minor release will be `v5.45.0`.

The **runtime release** will follow a new versioning approach - `runtime-XXYY`.

* The `XX` part will be a number of 2 or more digits, starting with **10**, and will be incremented by **1** each time a new runtime release is made. E.g. `runtime-1000` will be followed by `runtime-1100`, which will be followed by `runtime-1200`, and so on. This is like a combination of _major_ and _minor_ semver versions.
* The `YY` part will always be a 2 digit number, and serves as a _patch_ semver version. E.g. if we have `runtime-1000` and need to release a fix, the new release version will be `runtime-1001`.

The runtime crate version will align its major and minor versions with the Rust crate version, while the patch version will always remain `00`. For example, a runtime release for `runtime-1100` corresponds to the Rust runtime crate version `11.0.0`. A minor runtime release such as `runtime-1101` corresponds to the Rust runtime crate version `11.1.0`.

## Workspace Dependency Handling

All dependencies should be listed inside the workspace's root `Cargo.toml` file.
This allows us to easily change version of a crate used by the entire repo by modifying the version in a single place.

Right now, if **non_std** is required, `default-features = false` must be set in the root `Cargo.toml` file (related to this [issue](https://github.com/rust-lang/cargo/pull/11409)). Otherwise, it will have no effect, causing your compilation to fail.
Also `package` imports aren't properly propagated from root to sub-crates, so defining those should be avoided.

Defining _features_ in the root `Cargo.toml` is additive with the features defined in concrete crate's `Cargo.toml`.

**Adding Dependency**

1. Check if the dependency is already defined in the root `Cargo.toml`
    1. if **yes**, nothing to do, just take note of the enabled features
    2. if **no**, add it (make sure to use `default-features = false` if dependency is used in _no_std_ context)
2. Add `new_dependecy = { workspace = true }` to the required crate
3. In case dependency is defined with `default-features = false` but you need it in _std_ context, add `features = ["std"]` to the required crate.

## Further Reading

* [Official Documentation](https://docs.astar.network/)
* [Whitepaper](https://github.com/AstarNetwork/plasmdocs/blob/master/wp/en.pdf)
* [Whitepaper(JP)](https://github.com/AstarNetwork/plasmdocs/blob/master/wp/jp.pdf)
* [Substrate Developer Hub](https://substrate.dev/docs/en/)
* [Substrate Glossary](https://substrate.dev/docs/en/knowledgebase/getting-started/glossary)
* [Substrate Client Library Documentation](https://polkadot.js.org/docs/)
* [Astar Network Audit Reports](https://github.com/AstarNetwork/Audits)
* [Astar Code Documentation](https://astarnetwork.github.io/Astar/)
