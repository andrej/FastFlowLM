# IRON model extension

This C++ plugin starts the IRON server and exchanges tokens/logits or text/embeddings
through its versioned pipe protocol. Llama uses FLM's built-in frontend, tokenizer
and sampler. EmbeddingGemma-2 uses IRON's text processor.

With IRON installed in the active environment, build FLM and this extension from
the repository root:

```bash
source /opt/xilinx/xrt/setup.sh
FLM_PREFIX="$PWD/install" src/home_install.sh
source install/flm_env.sh
cmake -S extensions/iron -B build-iron -DCMAKE_PREFIX_PATH="$PWD/install"
cmake --build build-iron --parallel 2
ctest --test-dir build-iron --output-on-failure
flm run llama3.2-iron:1b --ctx-len 2048 --model-extension "$PWD/build-iron/extension.json"
```

Use `flm serve` for the API server. To serve embeddings, select
`embeddinggemma-2:740m` instead. `/v1/embeddings` accepts text or an array of texts,
and optional `dimensions` (128, 256, 512, or the default 768).

Set `FLM_MODEL_PATH` to the directory containing `models/`. Missing weights are
pulled from the pinned sources in the plugin's catalog. Set `HF_TOKEN` to a read
token with access to the selected checkpoint. IRON uses `NPU_CACHE_HOME` for its
compiled artifacts. `FLM_IRON_PYTHON` optionally selects the server interpreter.

The bundle consists of `extension.json`, `model-list.json`, and the shared library.
It can be built independently with FLM's installed `ModelExtension` SDK, Boost
headers, and nlohmann/json headers (`JSON_INCLUDE_DIR` when outside this source tree).
Match the SDK's XRT/HRX runtime to the host FLM build.

To add a model, register its existing FLM frontend family (or an embedding family)
in `model_extension.cpp`, add a catalog entry with `iron_model` naming the IRON
registry entry, and supply its pinned file records. No host FLM registration is needed.
