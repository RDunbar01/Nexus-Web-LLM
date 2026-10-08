NEXUS ADAPTIVE WEBGPU — README
Companion to: Nexus_Web_LLM_Debugged.html
Build: adaptive-20261008-1
Prepared: 8 October 2026

1. WHAT THIS APP DOES

NEXUS runs compatible local GGUF models inside a browser using a custom
WebGPU inference engine. It combines the attached NEXUS interface with
LittleBit packed-matrix and embedding kernels from your working v11 package.

It reads model metadata, validates the tensor graph, selects supported chat
formatting, estimates memory requirements, and runs model execution checks.
It also includes 100 searchable configuration profiles and a measured speed
tuning feature.

This build is NOT a complete replacement for llama.cpp. A model appearing in
the catalog does not mean that its architecture, every quantization, or its
language quality has been validated. Unsupported configurations are rejected.

2. WHAT YOU NEED

- Nexus_Web_LLM_Debugged.html.
- A compatible local .gguf model. Model weights are not included.
- A browser that exposes WebGPU, with graphics acceleration available.
  Browser testing for this build used Chromium with software graphics.
- Enough available GPU memory for weights, the attention cache, and working
  buffers. This build does not provide general CPU fallback or automatic
  CPU/GPU offloading for oversized transformer models.

PyTorch, a Python inference backend, an API key, and a model server are not
required for the normal local-file workflow.

3. QUICK START

1) Download the HTML file and open it in your browser.
2) Check the GPU indicator. Open Runtime details to inspect the adapter.
3) Open Settings. In “VRAM GiB,” enter your GPU's actual total memory if known.
   For example, entering 20 makes the planner use a 15 GiB budget, leaving
   25% outside that budget. The planner reserves more space within its budget
   for working buffers. This is an estimate, not an available-memory reading.
4) Choose Load model and select your .gguf file.
5) Wait for model loading and execution checks to finish. Open Model health
   to inspect the checks if loading fails.
6) Send a short test prompt first.
7) Open Settings and press “Detect hardware / Auto tune” or
   “Benchmark / tune speed” after loading. With a model loaded, either button
   starts the measured tuning procedure.

Do not open several copies of a large model while testing. Each browser tab
can allocate its own GPU buffers.

If opening the file directly does not expose WebGPU, you can serve the folder
locally. With Python already installed, open a terminal in the folder and run:

    python -m http.server 8000 --bind 127.0.0.1

Then open:

    http://127.0.0.1:8000/Nexus_Web_LLM_Debugged.html

On Windows, “py” may be used instead of “python.” This optional command only
serves the HTML page; inference still runs in the browser. Stop it with Ctrl+C.

4. AUTOMATIC CONFIGURATION AND TUNING

At model load, the app reads GGUF architecture, dimensions, attention-head
counts, tokenizer information, tensor types, and context metadata. These
values determine execution; catalog names do not override tensor shapes.

Memory planning:

- Uses browser-exposed GPU buffer limits and available hardware hints.
- Cannot reliably read total VRAM, free VRAM, or exact installed system RAM.
- Uses a conservative 1 GiB GPU budget when total VRAM is left blank.
- Uses 75% of entered total VRAM when you supply a value.
- Reserves space for working buffers and estimates the attention cache.
- Selects context in 128-token increments, capped at 4096 tokens and further
  limited by model metadata, memory estimates, and GPU binding limits.
- Adjusts RAM staging and NHSC1 decode concurrency using browser hints.

The GPU context and VRAM budget displays are calculated outputs, not manual
controls in this build. Entering a different total VRAM value requires
reloading the model to recalculate its allocations. Do not enter a larger
value than your actual hardware has just to bypass a memory error.

Measured speed tuning:

- Compares prompt-processing queue depths of 4, 8, and 16.
- Selects the fastest measured depth for the current session.
- Measures a short prompt and eight decode steps.
- Displays measured prefill and decode tokens per second.

These are short, workload-dependent measurements. They do not establish
sustained performance or the globally best settings. Repeat tuning after
changing models. Tuning resets the execution cache but does not erase the
visible conversation; the next request rebuilds context from chat history.

5. COMPATIBILITY

Container formats:

- Ordinary GGUF versions 2 and 3.
- The app's existing NHSC1 V3 compressed GGUF reader.
- v11 LittleBit factor GGUF files with the expected metadata and factor layout.

Implemented dense weight types:

    F32, F16, BF16
    Q4_0, Q4_1
    Q5_0, Q5_1
    Q8_0
    Q4_K, Q5_K, Q6_K

A mixed-quantization model must use implemented types for every tensor needed
by its execution graph. The .gguf filename alone is insufficient to establish
compatibility.

Architecture names admitted for further validation:

    llama, mistral, mistral3, ministral, qwen2, qwen3

SmolLM2 normally uses a Llama-style architecture with its own tokenizer and
ChatML formatting. Qwen support includes projection biases and Qwen3 per-head
query/key normalization.

Chat formatting includes supported ChatML, Llama 3 header, Mistral instruction,
and plain-completion paths. The app does not execute arbitrary Jinja
chat templates. Architecture admission is not a guarantee that every model
in that family will pass all remaining checks.

Not implemented in this build:

- Mixture-of-experts models, including GPT-oss and Mixtral.
- Gemma and Phi architecture graphs.
- Hybrid/state-space graphs.
- IQ, TQ, MXFP4, Q2_K, Q3_K, and other unlisted execution quantizations.
- Loading multiple GGUF shards as one model.
- Vision, image, audio, or other multimodal inputs.
- Sliding-window attention, partial rotary embeddings, per-frequency RoPE
  tensors, logit softcapping, and unsupported RoPE scaling variants.
- General CPU fallback and model weights spilling automatically into RAM.

Some Mistral or Llama-family models require features in this unsupported list.
A familiar family name therefore does not guarantee support.

The 100 profiles are curated names with configuration defaults and status
labels. They are not a popularity ranking, model downloads, or a list of 100
successfully tested models. GPT-oss entries are marked unsupported; the app
does not provide access to the ChatGPT service.

6. CHAT, MEMORY, AND STORAGE

Each new request formats the conversation history and rebuilds its execution
context. Prompt tokens, conversation history, template text, and requested
output must fit the configured context together.

“Response tokens” limits total generated tokens for the turn, including any
reasoning the model generates. This is not a word count.

Use Stop to interrupt generation. If you choose New chat while an operation
is active, the app requests a stop; choose New chat again after it finishes.
Stop the current operation before switching models.

Selected settings and the existing local-memory feature use browser local
storage. Memory can retain extracted facts and brief episodes; it does not
train the model or change its weights. In this adaptive generation path,
persistent memory is not automatically injected into the prompt. Visible
conversation history is used, but a complete restorable chat archive is not
promised. New chat does not clear persistent memory; use the Memory controls
or the “clear my memory” command for that.

The normal file-picker inference path reads local model files and does not
require a remote inference service. The retained Foundry integration can make
network requests when explicitly launched with project_id and autoload=1 URL
parameters. Browser storage behavior can differ between direct-file and
localhost use. Keep your original model files separately.

7. TROUBLESHOOTING

“WebGPU unavailable”
  Check browser graphics acceleration and Runtime details. Try reopening the
  browser or the localhost launch method above. A browser displaying the HTML
  successfully does not establish that WebGPU inference is available.

“Model weights need ... conservative GPU budget ...”
  Enter your actual total VRAM in Settings and load the model again. If it
  still does not fit, use a smaller model or a supported smaller quantization.

“Architecture ... has no ... execution graph” / unsupported quantization
  That graph or tensor format is not implemented. Renaming the file or choosing
  a catalog profile will not repair it. Use a compatible model or conversion.

“Unrecognized chat template” / unsupported pre-tokenizer
  The model needs a tokenizer or template implementation absent from this
  build. Do not force a different model's formatter just to make it load.

“Truncated tensor,” invalid shape, duplicate tensor, or GGUF parsing error
  Confirm the model file is complete and that you did not select a shard or
  unrelated artifact. Re-copy or re-download the original model if necessary.

“Prompt plus output exceeds context” / context-limit error
  Start a new chat, shorten the prompt, or reduce Response tokens. This build
  caps automatic context at 4096; it does not enable unlimited context merely
  because a model advertises a larger training window.

GPU device lost, browser instability, or allocation failure
  Close other GPU-heavy tabs/apps, reload the page, and try a smaller model.
  A successful memory estimate is not a guarantee that allocation will succeed.

Model loads but produces nonsense or repeats itself
  Export diagnostics. Loading, finite logits, and a tokenizer round trip do
  not prove real-model language quality. A base model may also need a different
  prompting approach from an instruction-tuned model. Do not treat preset
  selection as evidence of reference parity.

Numbers labelled “est v15,” “faster,” or a legacy reasoning budget
  The inherited chat footer can still display these older estimates. They are
  not measured comparisons with llama.cpp or proof of a speed improvement.
  Use the dedicated Benchmark / tune speed results for measured timings.

To report a problem, use Settings → Export diagnostics and include the exact
model filename, quantization, browser, GPU, and error message. The JSON contains
runtime/model details and measurements; review it before sharing.

8. WHAT WAS TESTED

The repair was checked using:

- 11 JavaScript, parser, catalog, and configuration tests.
- SmolLM2 token-ID comparison against Hugging Face tokenizers on seven cases,
  including multilingual text and whitespace-sensitive input.
- Compilation of 12 WGSL inference pipelines.
- Independent NumPy comparisons for all 11 listed dense weight formats,
  LittleBit matrix/embedding math, Qwen biases, and Qwen3 normalization.
- Browser loading and token generation with synthetic Llama, Qwen2, Qwen3,
  and LittleBit models.
- Full synthetic GPU-graph logits against independent NumPy references.
- Failed-load buffer cleanup, successful reload after failure, and measured
  queue-depth tuning in Chromium.

Maximum absolute logit errors in the synthetic graph comparisons:

    Llama:       approximately 0.000024803
    Qwen2:       approximately 0.000013024
    Qwen3:       approximately 0.000117302
    LittleBit:   approximately 0.000000030

Graphics execution used SwiftShader and llvmpipe software adapters. No Radeon
throughput or broad real-model quality benchmark was performed in this repair.
Synthetic weights test implementation math; they are not trained language
models and their generated text is not a quality demonstration. The NHSC1
reader was retained but was not independently revalidated end to end here.

9. SOURCE AND SCOPE

This build derives from the supplied Nexus Web LLM HTML and your
NEXUS_WebGPU_Complete_v11 package, including smol_integration.js and the v11
SmolLM2 tokenizer and packed-embedding changes.

The deliverable is one self-contained HTML application. This README documents
that specific build. It does not imply that future models, arbitrary LittleBit
layouts, or all features supported by llama.cpp are implemented.
