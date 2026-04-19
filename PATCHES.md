# Local patches (gru-svdd fork)

This fork tracks [mlc-ai/tokenizers-cpp](https://github.com/mlc-ai/tokenizers-cpp)
`main` at commit **`acbdc5a27ae01ba74cda756f94da698d40f11dfe`**, with the following
changes in **`CMakeLists.txt`** only.

## 1. Respect parent-provided Rust / cargo target variables

**Upstream:** `TOKENIZERS_CPP_RUST_FLAGS` and `TOKENIZERS_CPP_CARGO_TARGET` are set
unconditionally to empty strings at the start of the file, which overwrites values
an integrating project sets (including via `CACHE`) before `add_subdirectory()`.

**Change:** Wrap those assignments in `if(NOT DEFINED …)` so a parent CMake
project can set them first (for example `RUSTFLAGS` for toolchain lints).

## 2. Honor `MLC_ENABLE_SENTENCEPIECE_TOKENIZER` for the whole SentencePiece dependency

**Upstream:** `MLC_ENABLE_SENTENCEPIECE_TOKENIZER` only toggles a compile definition.
`sentencepiece_tokenizer.cc` is always compiled, `sentencepiece/` is always added as
a subdirectory, and `sentencepiece-static` is always linked.

**Change:** When the option is `OFF`:

- Build only `src/huggingface_tokenizer.cc` and `src/rwkv_world_tokenizer.cc` (omit
  `src/sentencepiece_tokenizer.cc`).
- Do not add `sentencepiece/src` to include paths.
- Do not call `add_subdirectory(sentencepiece …)`.
- Link `tokenizers_cpp` against `tokenizers_c` and `TOKENIZERS_CPP_LINK_LIBS` only
  (omit `sentencepiece-static`).

**Reason (integrating project):** gru-svdd uses Hugging Face `tokenizer.json` only.
Skipping SentencePiece avoids pulling in its protobuf stack alongside ONNX Runtime.

## Integrating project (gru-svdd)

`app/CMakeLists.txt` sets `MLC_ENABLE_SENTENCEPIECE_TOKENIZER=OFF` before
`add_subdirectory(third_party/tokenizers-cpp)`.
