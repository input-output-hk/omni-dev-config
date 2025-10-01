# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`cardano-base` is a collection of foundational Haskell packages for Cardano, covering cryptography, serialization, and slotting. This is a multi-package Cabal project with 12 packages:

**Core packages:**
- `cardano-crypto-class` - Type classes and implementations for cryptographic primitives (digital signatures, hashing, key-evolving signatures, VRF)
- `cardano-binary` - Binary serialization/deserialization built on cborg
- `cardano-slotting` - Time and slot management
- `cardano-strict-containers` - Strict variants of standard containers

**Supporting packages:**
- `base-deriving-via`, `orphans-deriving-via` - DerivingVia utilities
- `cardano-crypto-praos`, `cardano-crypto-tests` - Praos crypto and tests
- `cardano-git-rev` - Git revision tracking
- `heapwords`, `measures` - Memory and performance utilities

## Building

### Nix (Recommended)

```bash
nix develop                    # Enter dev shell
cabal build all                # Build all packages
```

The default GHC is 9.6.7 (see `flake.nix`). Alternative compilers and shells:
```bash
nix develop .#ghc912          # Use GHC 9.12
nix develop .#profiling       # Shell with profiling enabled
nix develop .#pre-commit      # Shell with pre-commit hooks
```

### Without Nix

**See [INSTALL.md](INSTALL.md) for comprehensive installation instructions.**

Quick reference - requires custom cryptographic libraries:
- **libsodium-vrf** (custom fork with VRF support, NOT standard libsodium)
- **libsecp256k1** (with Schnorr signatures)
- **libblst**

Pre-built binaries available at: https://github.com/input-output-hk/iohk-nix/releases/latest

After installing libraries, set:
```bash
export PKG_CONFIG_PATH="/usr/local/opt/cardano/lib/pkgconfig:$PKG_CONFIG_PATH"
cabal build all
```

**Common Issue**: If hsc2hs crashes (exit code -9), you're using standard libsodium instead of the VRF fork. See INSTALL.md troubleshooting section.

## Testing

```bash
cabal test all                                                    # Run all tests
cabal test cardano-crypto-tests                                   # Run specific package tests
cabal test cardano-crypto-tests --test-options '-p blake2b_256'   # Run specific test pattern
TASTY_PATTERN="blake2b_256" cabal test cardano-crypto-tests       # Alternative pattern syntax
```

Tests use Tasty framework with pattern matching via `-p` flag or `TASTY_PATTERN` environment variable.

## Code Quality

### Formatting

Use `fourmolu` for code formatting:
```bash
./scripts/fourmolize.sh           # Format all .hs files
./scripts/fourmolize.sh --changes # Format only changed files vs master
```

Format configuration is enforced, and the script will exit with non-zero if changes are needed.

### Warnings

The project builds with `-Werror` in CI. To temporarily disable during development:
```bash
cabal configure <package-name> --ghc-options="-Wwarn"
cabal build <package-name>
```

Disable warnings at module level (not package level) if truly necessary.

## Dependencies

### Package Repositories

Packages come from:
- Hackage (pinned via `index-state` in `cabal.project`)
- CHaP (Cardano Haskell Packages, also pinned via `index-state`)

### Updating Index State

When bumping `index-state` in `cabal.project`:
1. Run `cabal update` to sync local Cabal cache
2. Update Nix inputs:
   - `nix flake lock --update-input haskellNix/hackage` for Hackage
   - `nix flake lock --update-input CHaP` for CHaP

### Source Repository Packages

Avoid using `source-repository-package` in `cabal.project`. Packages cannot be released to CHaP while depending on `source-repository-package` entries. For long-running forks, release to CHaP instead.

If unavoidable, add `--sha256` comment for Nix.

## Development Workflow

### Git Configuration

Recommended setup:
```bash
git config blame.ignoreRevsFile .git-blame-ignore-revs
```

### Branching

Trunk-based development: branch from `master`, merge back to `master`. Linear history required (rebase before merge).

### Commit Signing

All commits must be signed.

## Architecture Notes

### Cryptographic FFI

`cardano-crypto-class` uses Foreign Function Interface (FFI) to C libraries:
- Constants extracted via `hsc2hs` (e.g., `Cardano/Crypto/Libsodium/Constants.hsc`)
- Direct bindings to libsodium, libsecp256k1 (with flag), libblst

The `secp256k1-support` flag (default: True) enables/disables secp256k1 functionality.

### Package Structure

Each subdirectory is an independent Cabal package with its own `.cabal` file and source tree. The root `cabal.project` ties them together as a multi-package project.

### Testing Infrastructure

`cardano-crypto-tests` provides test suite and benchmarks for crypto primitives. Tests are organized using Tasty's hierarchical test groups.

## Releasing

Packages are released to CHaP (Cardano Haskell Packages). See `RELEASING.md` and CHaP documentation for versioning and release process.

## CI and Hydra

Nix-based CI builds on Hydra. The flake defines `hydraJobs` which aggregate all build outputs. Windows cross-compilation is included (x86_64 only).
