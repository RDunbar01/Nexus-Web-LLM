# NEXUS Web LLM

## Created with AI-assisted coding

**Developed by Rich Dunbar through hundreds of build–measure–learn iterations.**

This was one of the first web-based inference engines I created using **AI-assisted coding**. I directed the project, made the design decisions and used AI to help implement, debug, review and refine the application.

Development followed a repeated cycle: **build a capability, measure its behavior, learn from the results, and improve the next iteration**. Hundreds of revisions helped shape the engine and its browser-based workflow.

The custom engine **does not require llama.cpp for local inference**. Its development formed a backbone for my web-based inference applications, bringing model execution into the browser through WebGPU.

> [!IMPORTANT]
> **Development of this standalone program has stopped.**
> Work on the engine and application experience continues in **Nexus AI Studio**. This README preserves the scope and evidence for the legacy build; it does not imply that newer Studio features have been backported here.

### Radeon hardware testing

I bench-tested the application on my **AMD Radeon 7900**, observing GPU utilization and VRAM use to demonstrate inference without CPU model-compute offloading in the configurations I tested.

This is my report of hands-on hardware testing. The automated evidence for the specific adaptive repair is documented separately below. No per-model hardware logs or numerical Radeon benchmark results accompany this README, so this statement does not qualify every listed profile or establish a throughput figure.

“Without offloading” refers to model computation remaining on the GPU in those tests. The browser still uses CPU and system memory for application control, file handling and staging.

---

**Local GGUF inference in your browser, powered by a custom WebGPU engine.**

NEXUS Adaptive WebGPU combines the NEXUS browser interface with the LittleBit packed-matrix and embedding kernels from the NEXUS v11 project. It reads GGUF metadata, validates model graphs, selects supported chat formats, estimates memory requirements, and performs execution checks before generation.

| Project detail | Value |
| --- | --- |
| Application | `Nexus_Web_LLM_Debugged.html` |
| Build | `adaptive-20261008-1` |
| Documentation date | 8 October 2026 |
| Runtime | Browser-based WebGPU (no inference server required) |
| Model input | Compatible local `.gguf` files (weights not included) |
| Current status | **Legacy experimental build · Development stopped** |
| Continuing development | **Nexus AI Studio** |

> [!IMPORTANT]
> **This is not a complete `llama.cpp` replacement.** The 100 searchable configuration profiles are presets, **not** 100 validated models. Model family, quantization support, successful loading, and language quality are separate questions. Unsupported configurations are rejected.

## Contents

- [Created with AI-assisted coding](#created-with-ai-assisted-coding)
- [Radeon hardware testing](#radeon-hardware-testing)
- [Features](#features)
- [Requirements](#requirements)
- [Quick start](#quick-start)
- [Automatic configuration and tuning](#automatic-configuration-and-tuning)
- [Compatibility](#compatibility)
- [Chat, memory, and storage](#chat-memory-and-storage)
- [Troubleshooting](#troubleshooting)
- [Validation and test results](#validation-and-test-results)
- [Source and scope](#source-and-scope)

## Features

| Capability | Description |
| --- | --- |
| **Local browser inference** | Loads compatible GGUF files through a file picker and runs the execution graph with WebGPU. |
| **Model inspection** | Reads architecture, dimensions, tensor types, tokenizer data, context metadata, and attention-head configuration. |
| **Compatibility gates** | Validates the tensor graph, tokenizer/chat format, and implemented weight types. |
| **LittleBit support** | Supports the expected v11 packed-factor GGUF layout and embedding kernels. |
| **Memory planning** | Estimates GPU budgets, working buffers, attention cache, and a safe context configuration. |
| **Adaptive speed tuning** | Measures prompt processing and decode speed across candidate queue depths. |
| **Configuration catalog** | Includes 100 searchable profiles; a catalog entry does not certify model compatibility. |
| **Diagnostics** | Displays runtime details and model health checks; supports exporting diagnostics. |

## Requirements

- **Application:** `Nexus_Web_LLM_Debugged.html`.
- **Model:** A compatible local `.gguf` file. No model weights are distributed with the application.
- **Browser:** A browser exposing WebGPU with usable graphics acceleration. The build's recorded browser testing used Chromium with software graphics adapters.
- **GPU memory:** Sufficient capacity for model weights, attention cache, and working buffers.

**Not required for ordinary local inference:** llama.cpp, PyTorch, a Python inference backend, an API key, or a model server.

> [!WARNING]
> This build does **not** provide a general CPU fallback or automatic CPU/GPU offloading when a transformer model exceeds GPU memory.

## Quick start

1. Download `Nexus_Web_LLM_Debugged.html` and open it in a WebGPU-capable browser.
2. Check the **GPU** indicator; open **Runtime details** to inspect the adapter.
3. Open **Settings**. If known, enter the GPU's actual total memory under **VRAM GiB**. For example, entering `20` uses a **15 GiB planning budget** (75% of the entered total), with an additional internal allowance for working buffers.
4. Select **Load model** and choose a compatible local `.gguf` file.
5. Allow model loading and execution checks to finish. If loading fails, inspect **Model health**.
6. Send a short test prompt before using longer conversations.
7. With a model loaded, select **Detect hardware / Auto tune** or **Benchmark / tune speed** in **Settings** to start measured tuning.

> [!TIP]
> Avoid opening several copies of a large model in separate browser tabs. Each tab can allocate its own GPU buffers.

### Alternative: launch from localhost

If direct file opening does not expose WebGPU, serve the directory locally. In a terminal opened in the same folder as the HTML file, run:

```bash
python -m http.server 8000 --bind 127.0.0.1
```

Open the following URL in your browser:

```text
http://127.0.0.1:8000/Nexus_Web_LLM_Debugged.html
```

On Windows, `py` may work in place of `python`. This optional command **only serves the HTML**; inference still executes in the browser. Press `Ctrl+C` in the terminal to stop the server.

## Automatic configuration and tuning

At load time, model metadata—not a catalog name—determines graph selection and execution limits.

### Memory planning

The planner:

- Uses WebGPU-exposed buffer limits and browser hardware hints.
- **Cannot reliably read** total or free VRAM, or exact installed system RAM.
- Defaults to a conservative **1 GiB GPU budget** when total VRAM is unspecified.
- Uses **75% of user-entered total VRAM** as its initial planning budget.
- Accounts for working buffers and estimates the attention cache.
- Selects context in **128-token increments**, capped at **4,096 tokens** and additionally constrained by metadata, memory estimates, and GPU binding limits.
- Adjusts RAM staging and NHSC1 decode concurrency using browser hints.

The GPU context and VRAM budget displays are **calculated outputs**, not editable controls. After changing the total VRAM setting, **reload the model** to recalculate allocations. Do not enter more VRAM than the system actually has to bypass an allocation error.

### Measured speed tuning

| Measurement | Behaviour |
| --- | --- |
| Queue depths | Compares `4`, `8`, and `16` |
| Selection | Uses the fastest measured depth for the current session |
| Test workload | Short prompt plus eight decode steps |
| Output | Measured prefill and decode tokens per second |

These are brief, workload-dependent tests—not sustained throughput benchmarks or proof of globally optimal settings. Repeat tuning after changing models. Tuning clears the execution cache **without erasing visible chat history**; the next turn rebuilds context from that history.

## Compatibility

A matching architecture label or `.gguf` filename alone **does not guarantee compatibility**. Every required tensor, operator, tokenizer, and model-specific graph feature must be supported.

### Containers and compression

| Format | Status |
| --- | --- |
| GGUF v2 / v3 | Supported for implemented model graphs and tensor types |
| NHSC1 V3 compressed GGUF | Existing reader retained; **not revalidated end to end** in this repair |
| NEXUS v11 LittleBit factor GGUF | Supported when expected metadata and factor layout are present |

### Implemented dense weight formats

| Category | Formats |
| --- | --- |
| Floating point | `F32`, `F16`, `BF16` |
| Standard quantization | `Q4_0`, `Q4_1`, `Q5_0`, `Q5_1`, `Q8_0` |
| K-quantization | `Q4_K`, `Q5_K`, `Q6_K` |

Mixed-quantization GGUFs must use implemented formats for **all** required graph tensors.

### Architectures admitted for additional validation

`llama` · `mistral` · `mistral3` · `ministral` · `qwen2` · `qwen3`

- **SmolLM2** generally uses a Llama-style graph but needs its own tokenizer and ChatML handling.
- **Qwen** support covers projection biases; **Qwen3** also includes per-head query/key normalization.
- Implemented formatting paths include supported **ChatML**, **Llama 3 header**, **Mistral instruction**, and **plain completion** modes.
- Arbitrary **Jinja chat templates are not executed**.

Admission to this architecture list means the build can attempt further validation, **not** that every model in the family runs correctly.

### Not implemented

- Mixture-of-experts graphs, including **GPT-oss** and **Mixtral**.
- **Gemma**, **Phi**, and hybrid/state-space architectures.
- Quantizations such as `IQ`, `TQ`, `MXFP4`, `Q2_K`, `Q3_K`, and other formats not listed above.
- Multi-shard GGUF loading as a single model.
- Vision, image, audio, and other multimodal inputs.
- Sliding-window attention, partial rotary embeddings, per-frequency RoPE tensors, logit softcapping, and unsupported RoPE scaling variants.
- General CPU fallback or automatic overflow of model weights into system RAM.

Some Llama- and Mistral-family models require one or more unsupported features. The **100 configuration profiles** are curated configuration suggestions and status labels, not rankings, model downloads, or proof of 100 tested models. GPT-oss entries are marked unsupported; this app does not provide access to the ChatGPT service.

## Chat, memory, and storage

### Context and generation

Each turn reformats the visible conversation and rebuilds its execution context. The **prompt, chat history, chat template, and requested output** must all fit the selected context window. **Response tokens** limits generated tokens for the turn, including reasoning tokens; it is not a word count.

Use **Stop** to interrupt generation. If **New chat** is selected during an active operation, the app requests a stop; select **New chat** again when the operation has stopped. Stop inference before switching models.

### Local memory

Selected settings and the existing memory feature use **browser local storage**. Saved memories can preserve extracted facts and brief episodes, but they **do not train the model or modify weights**. In this adaptive generation path, stored persistent memory is **not automatically inserted into prompts**. Visible conversation history is used; full restore of a chat archive is not guaranteed.

**New chat does not clear saved memory.** Use the memory controls or the `clear my memory` command when needed.

### Local vs. network behaviour

Normal file-picker inference works with local model files **without a remote inference service**. The retained Foundry integration **may make network requests** when explicitly launched with `project_id` and `autoload=1` URL parameters. Browser storage differs between `file://` and `localhost` in some environments; keep original model files separately.

## Troubleshooting

| Symptom | Recommended check or action |
| --- | --- |
| **WebGPU unavailable** | Verify browser graphics acceleration and **Runtime details**. Restart the browser or try the localhost method. HTML rendering alone does not demonstrate WebGPU availability. |
| **Conservative GPU budget / insufficient space** | Enter actual total VRAM in **Settings**, then reload the model. If it still cannot fit, choose a smaller model or a supported lower-bit format. |
| **Unsupported architecture or quantization** | The required graph or tensor format is not implemented. Renaming a file or choosing a different preset will not add support. |
| **Unrecognized chat template / pre-tokenizer** | The tokenizer or formatter is unsupported. Do not force an unrelated model's chat format. |
| **Truncated tensor / invalid shape / duplicate tensor / GGUF parsing error** | Check for an incomplete or corrupted file, a model shard, or the wrong artifact. Re-copy or re-download the original if necessary. |
| **Prompt plus output exceeds context** | Start a new chat, shorten input, or lower **Response tokens**. Automatic context remains capped at 4,096. |
| **GPU device lost / allocation failure / instability** | Close other GPU-intensive tabs and apps, reload, and try a smaller model. A memory estimate does not guarantee successful allocation. |
| **Nonsense or repetitive output** | Export diagnostics and inspect model and tokenizer compatibility. Passing load and finite-logit checks does not prove language quality. Base models may need different prompts than instruct models. |
| **Legacy “est v15”, “faster”, or reasoning figures** | These inherited interface estimates are **not** measured llama.cpp comparisons. Use **Benchmark / tune speed** for session measurements. |

### Reporting an issue

Active development of this standalone program has ended. Diagnostic records can still help document legacy behavior and inform continuing work in Nexus AI Studio; this is not a commitment to further repairs of this build.

Use **Settings → Export diagnostics**. Include the exact **model filename**, **quantization**, **browser**, **GPU**, and **error message**. Review exported diagnostic JSON before publishing it, as it contains runtime/model details and measurements.

## Validation and test results

The developer-reported Radeon bench testing above and the saved automated checks below are separate evidence sources. The supplied documentation for the adaptive repair records these checks:

- **11** JavaScript/parser/catalog/configuration tests.
- SmolLM2 token-ID comparison with Hugging Face tokenizers on **seven cases**, including multilingual and whitespace-sensitive inputs.
- Compilation of **12 WGSL inference pipelines**.
- Independent NumPy comparisons for all **11 implemented dense weight formats**, LittleBit matrix/embedding math, Qwen projection biases, and Qwen3 normalization.
- Synthetic Llama, Qwen2, Qwen3, and LittleBit browser model loading and token generation.
- Synthetic GPU graph logits compared against independent NumPy references.
- Failed-load buffer cleanup, reload after failure, and Chromium queue-depth tuning.

### Synthetic GPU graph logit comparisons

Maximum reported **absolute logit error**:

| Synthetic graph | Maximum absolute error (approx.) |
| --- | ---: |
| Llama | `0.000024803` |
| Qwen2 | `0.000013024` |
| Qwen3 | `0.000117302` |
| LittleBit | `0.000000030` |

> [!CAUTION]
> Graphics testing for **this adaptive repair** used the **SwiftShader** and **llvmpipe** software adapters. **The saved repair receipts do not include a Radeon throughput benchmark or broad real-model quality benchmark.** The developer reports separate Radeon bench testing above; its per-model measurements are not included in these receipts. Synthetic tensor calculations demonstrate implementation math; they are not trained language-model quality tests. The NHSC1 reader was retained but not independently revalidated end to end.

## Source and scope

This adaptive build derives from the supplied NEXUS Web LLM HTML and `NEXUS_WebGPU_Complete_v11`, including `smol_integration.js`, SmolLM2 tokenizer integration, and packed-embedding work.

The documented deliverable is **one self-contained HTML application**: `Nexus_Web_LLM_Debugged.html`. This README describes **build `adaptive-20261008-1` only**; it does not promise all `llama.cpp` features, arbitrary LittleBit layouts, or compatibility with future models.

---

**Rich Dunbar · NEXUS Emerging Technology**  
**AI-assisted development · Build–measure–learn · Continued in Nexus AI Studio**
