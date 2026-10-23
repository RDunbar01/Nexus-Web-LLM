# Nexus Web LLM

A legacy browser inference application with a custom WebGPU runtime for supported local model files. Standalone development has stopped; continuing work is in [Nexus AI](https://github.com/RDunbar01/Nexus-AI-Studio).

[Application HTML](<Nexus Web LLM.html>) · [Historical technical notes](docs/README_PREVIOUS.md) · [License](LICENSE)

This documentation describes the committed application without creating a new software release. The current HTML identifies itself as **NEXUS Stable Turbo v19**; the preserved technical notes describe a different adaptive build and should not be treated as matching-source test evidence.

## What the source implements

- Readers for normal GGUF and NHSC1 V3 compressed GGUF files, with model metadata and tensor inspection.
- A custom WebGPU execution path, including transformer and Nexus GRU components, tokenizer handling and generation controls.
- Parallel NHSC1 reconstruction and chunked GPU upload while loading supported models.
- Runtime and model-health displays, memory-budget settings and GPU capability checks.
- Local conversation controls and persistent-memory export/clear controls. Stored memory does not train model weights.

These are implementation paths, not a list of qualified models. A model must satisfy the actual graph, tensor, tokenizer and device requirements; a matching name or `.gguf` extension is insufficient. This is not a complete llama.cpp replacement.

## Engineering contribution

Developed by **Rich Dunbar / Nexus Emerging Technology with AI-assisted coding**. The work combines model-file readers, browser GPU execution, loading controls and a local chat interface without requiring a remote inference server. It provided a foundation for subsequent Nexus browser-inference applications.

The monolithic HTML keeps the interface and JavaScript together, avoiding a build step when inspecting a downloaded copy and making online trials easier to package. No working hosted trial is supplied here. Model weights are separate, and opening the interface does not demonstrate successful inference.

## Inspect locally

1. Download the committed [`Nexus Web LLM.html`](<Nexus Web LLM.html>). GitHub's source viewer does not execute HTML.
2. Open it in a WebGPU-capable desktop browser and choose **Check WebGPU**.
3. Use **Load a GGUF model** with a compatible local file, then inspect runtime and health details before a short test prompt.

The browser runtime needs neither Python nor PyTorch. If WebGPU is unavailable from a local-file origin, serve the directory on localhost using a local server. An optional Python command is `python -m http.server 8000 --bind 127.0.0.1`; then open `http://127.0.0.1:8000/Nexus%20Web%20LLM.html`. Python only serves the file and does not perform inference.

Normal file-picker use does not need a remote inference service. The retained Foundry autoload integration can make network requests when explicitly invoked through its URL parameters.

## Evidence and license

The [historical notes](docs/README_PREVIOUS.md) separate developer-reported Radeon testing from saved software-adapter checks. Those results concern the documented historical build, not a new qualification of this committed HTML. No universal model compatibility, physical-GPU throughput or language-quality result is asserted here.

Existing terms remain in [LICENSE](LICENSE) and [Nexus_Web_LLM_License.txt](Nexus_Web_LLM_License.txt). Preserve model, upstream and separately licensed component notices. This README changes neither the application nor its licensing and does not restart standalone development.
