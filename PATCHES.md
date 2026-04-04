# Local Patches (gru-svdd)

This fork carries a small compatibility patch set on top of upstream
`tokenizers-cpp` `v0.1.1` to make builds deterministic and compatible with the
Fedora 42 toolchain used in this project.

## Patch Summary

1. **Make Rust flags externally configurable**
   - File: `CMakeLists.txt`
   - Change: only set `TOKENIZERS_CPP_RUST_FLAGS` and
     `TOKENIZERS_CPP_CARGO_TARGET` when not already defined.
   - Reason: allows integrating repositories to pass compatibility flags without editing
     Rust source files.

2. **Use locked Cargo resolution**
   - File: `CMakeLists.txt`
   - Change: initialize `TOKENIZERS_CPP_CARGO_FLAGS` with `--locked`.
   - Reason: enforce `Cargo.lock` for reproducible dependency resolution and
     avoid accidental API drift from newer crates.

3. **Honor sentencepiece ON/OFF switch in all build paths**
   - File: `CMakeLists.txt`
   - Changes:
     - include `src/sentencepiece_tokenizer.cc` only when
       `MLC_ENABLE_SENTENCEPIECE_TOKENIZER=ON`
     - add `sentencepiece` subdirectory only when enabled
     - link `sentencepiece-static` only when enabled
   - Reason: the integrating repository uses only Hugging Face `tokenizer.json` and disables
     sentencepiece to avoid protobuf runtime conflicts with ONNX Runtime.

## Integrating repository configuration (in gru-svdd/app)

The integrating repository configures:

- `MLC_ENABLE_SENTENCEPIECE_TOKENIZER=OFF`
- `TOKENIZERS_CPP_RUST_FLAGS=-Adangerous_implicit_autorefs`

These settings are applied in `app/CMakeLists.txt` and rely on the fork patches
above.
