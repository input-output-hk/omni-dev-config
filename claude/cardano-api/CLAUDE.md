# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository provides a Haskell API for interacting with Cardano as a user of the system. It integrates the [ledger](https://github.com/IntersectMBO/cardano-ledger), [consensus](https://github.com/IntersectMBO/ouroboros-consensus), and [networking](https://github.com/IntersectMBO/ouroboros-network) repositories into a unified interface intended for third-party applications.

## Packages

This is a multi-package repository with four main Cabal packages:

- **cardano-api**: The main library for constructing and submitting transactions in both Byron and Shelley formats. Primary entry point is `Cardano.Api` module.
- **cardano-api-gen**: Test generators and property testing utilities (has a public `gen` library)
- **cardano-rpc**: gRPC client and server modules implementing [UTxO RPC](https://utxorpc.org/introduction) protobuf protocol
- **cardano-wasm**: Experimental WebAssembly compilation support

## Build Commands

### Standard Build and Test

```bash
# Build all packages
cabal build all

# Build a specific package
cabal build cardano-api

# Build with specific GHC version (9.6 or 9.10 supported for regular builds)
cabal build --with-ghc=ghc-9.6.7

# Run all tests
cabal test all

# Run tests for specific package
cabal test cardano-api:cardano-api-test

# Run golden tests
cabal test cardano-api:cardano-api-golden

# Run single test by pattern
cabal test cardano-api:cardano-api-test --test-option='-p /pattern/'
```

### Code Quality

```bash
# Format code with fourmolu (settings in fourmolu.yaml)
fourmolu -i <file>

# Lint with HLint (config in .hlint.yaml)
hlint <file>
```

### WASM Build

**Important**: WebAssembly builds require a special wasm32-wasi-ghc compiler, not the standard GHC 9.6/9.10 used for regular builds. The WASM GHC is a cross-compiler targeting WebAssembly.

```bash
# Enter WASM Nix shell (provides wasm32-wasi-ghc and dependencies)
nix develop .#wasm

# Build with WASM compiler
cabal build cardano-wasm
```

For detailed WASM setup instructions (including non-Nix setup), see [cardano-wasm/README.md](cardano-wasm/README.md).

### Using Nix

```bash
# Enter development shell with all tools available
nix develop
```

## Architecture

### Era-Based Design

The API uses a type-level era system to track Cardano protocol versions. Understanding eras is crucial:

**Eras** represent distinct protocol versions:
- `ByronEra` - Original Cardano
- `ShelleyEra`, `AllegraEra`, `MaryEra` - Shelley family
- `AlonzoEra` - Plutus smart contracts introduced
- `BabbageEra` - Reference inputs, inline datums
- `ConwayEra` - Current era with governance features

**Eons** (type classes) represent era ranges where features exist:

*Forward-looking eons* (commonly used for new features):
- `ShelleyBasedEra` - All eras from Shelley onwards (most common constraint)
- `AllegraEraOnwards` - Eras from Allegra onwards (token support)
- `MaryEraOnwards` - Eras from Mary onwards (multi-asset support)
- `AlonzoEraOnwards` - Eras supporting Plutus scripts
- `BabbageEraOnwards` - Eras with reference scripts, inline datums, reference inputs
- `ConwayEraOnwards` - Eras with governance features (current era)

*Historical range eons* (for backwards compatibility):
- `ByronToAlonzoEra` - Byron through Alonzo (pre-Babbage)
- `ShelleyEraOnly` - Only the Shelley era
- `ShelleyToAllegraEra` - Shelley and Allegra only
- `ShelleyToMaryEra` - Shelley through Mary
- `ShelleyToAlonzoEra` - Shelley through Alonzo
- `ShelleyToBabbageEra` - Shelley through Babbage

Eons enable writing functions that work across multiple eras without boilerplate. When adding features, determine which eon constraint to use based on when the feature was introduced. Use forward-looking eons (e.g., `BabbageEraOnwards`) for new features, and historical range eons for maintaining backwards compatibility.

### Module Structure

The API is organized around domain concepts:

- **Cardano.Api** - Main entry point, re-exports all stable public functionality
- **Cardano.Api.Address** - Payment and stake addresses
- **Cardano.Api.Tx** - Transaction construction, signing, submission
- **Cardano.Api.Query** - Node state queries
- **Cardano.Api.Certificate** - Stake pool and delegation certificates
- **Cardano.Api.Governance** - Voting procedures, proposals (Conway onwards)
- **Cardano.Api.Plutus** - Plutus script integration
- **Cardano.Api.LedgerState** - Ledger state management and queries
- **Cardano.Api.Serialise.\*** - Serialisation modules:
  - **Bech32** - Bech32 encoding/decoding for addresses and keys
  - **Cbor** - CBOR serialisation for blockchain types
  - **Cbor.Canonical** - Canonical CBOR encoding (deterministic)
  - **Cip129** - CIP-129 serialisation format
  - **Json** - JSON serialisation for API types
  - **TextEnvelope** - Text envelope format for keys and certificates
  - **Raw** - Raw binary serialisation utilities
  - **SerialiseUsing** - Custom serialisation combinators
  - **DeserialiseAnyOf** - Polymorphic deserialisation helpers
- **Cardano.Api.Experimental** - Unstable APIs subject to change (**requires explicit import**, not re-exported by `Cardano.Api`)

**Why Experimental requires explicit import**: The `Cardano.Api.Experimental` module is intentionally not re-exported by the main `Cardano.Api` module to maintain a clear separation between stable and unstable APIs. This design ensures:
- **Explicit opt-in**: Users must consciously choose to use experimental features by writing `import Cardano.Api.Experimental`
- **API stability**: The main `Cardano.Api` module only exposes stable, production-ready interfaces
- **Clear boundaries**: Breaking changes in experimental APIs won't affect users who only import `Cardano.Api`

**Internal modules** (under `.Internal` namespaces) are implementation details and should not be exposed through public API. To add functionality, add it to an Internal module and re-export through the appropriate public module.

### Ledger Integration

The API wraps types from `cardano-ledger-*` packages. Key pattern: ledger types are typically wrapped in newtype wrappers to provide stable API. Convert using pattern matching or provided conversion functions.

## Code Formatting

This repository uses **fourmolu** for formatting with specific settings in [fourmolu.yaml](fourmolu.yaml):
- 2-space indentation
- 100 column limit
- Leading function arrows and commas
- Custom import grouping (Cardano.Api first, then other Cardano/Ouroboros, then standard libs, then Test modules)

Format before committing. CI checks formatting.

## Testing Strategy

- `cardano-api-test`: Property tests and unit tests using Hedgehog
- `cardano-api-golden`: Golden tests for serialization formats
- Test generators in `cardano-api-gen` package provide reusable generators

Use golden tests when testing serialization formats to catch unintended changes. Update golden files with `--accept` flag when changes are intentional.

## Release Process

Releases follow [PVP (Package Versioning Policy)](https://pvp.haskell.org/). The process is documented in [RELEASING.md](RELEASING.md):

1. Determine version bump based on breaking changes
2. Update version in all `.cabal` files
3. Generate changelog using `cardano-dev` scripts
4. Release to CHaP (Cardano Haskell Packages)
5. Tag release after CHaP succeeds

Key point: tag and merge release PR only **after** CHaP PR is merged and built successfully.

## Important Dependencies

- **cardano-ledger-\***: Core ledger types and rules
- **ouroboros-consensus**: Consensus layer integration
- **ouroboros-network**: Network protocols
- **plutus-ledger-api**: Plutus script integration

Dependencies are managed via CHaP. See [cabal.project](cabal.project) for index-state pinning. When updating dependencies, update both `cabal.project` index-state and regenerate `flake.lock`.

## GHC Compatibility

Supports GHC 9.6 and 9.10. All code must compile with `-Werror` (configured in cabal.project).

## Special Considerations

- **Never add emojis** to code or documentation unless explicitly requested
- **Avoid temp source-repository-package stanzas** in cabal.project - these should be temporary only
- **Respect export lists** - don't expose Internal module constructors
- When working with WASM, note many dependencies have special WASM-compatible forks
