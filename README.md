# ollama-rocm

AMD ROCm inference backend for the Ollama LLM server, as a charly *add-on*
layer.

The `ollama-rocm` candy adds the ROCm/HIP `ggml` backend so a composing box runs
inference on an AMD card instead of falling back to CPU. It is an **add-on to
the `ollama` candy, never a replacement**: it contributes only the backend
libraries that drop into Ollama's own runner directory
(`/usr/lib/ollama/rocm_v*/libggml-hip.so` plus the HIP/HSA runtime it links).
At ~2.9 GiB installed it is by far the largest backend, so keeping it in its own
candy stops every Ollama image from paying for it — an image opts in only when it
targets AMD hardware.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `ollama-rocm` |
| Requires | `pod-ollama` (the base `ollama` candy) |
| Backend | `/usr/lib/ollama/rocm_v*/libggml-hip.so` |
| Package | `ollama-rocm` (Arch) |
| Security | `group_add: keep-groups` (preserves host `video`/`render` groups) |
| Service / port | none (contributes a backend only) |

## Install paths

On Arch (CachyOS included) the split `ollama-rocm` package supplies the backend
and depends on the `ollama` package, so the two compose rather than conflict.

A distro without an Ollama package is **not covered here**: unlike CUDA, the
upstream tarball ships no ROCm backend (upstream publishes ROCm as a separate
`ollama-linux-amd64-rocm` archive), so composing this candy there would install
nothing and the check below would correctly fail rather than pretend. Extending
it to the tarball distros is a change to the `ollama` candy, not a second copy of
this one.

`keep-groups` preserves the host `video`/`render` groups so a passed-through AMD
GPU (`/dev/kfd` + `/dev/dri/renderD*`) is usable from the container. The backend
library is verifiable at build scope; live GPU enumeration needs a host with a
real AMD GPU passed through.

## How to use it

Compose the add-on alongside the base Ollama candy in an AMD GPU box:

```yaml
my-ollama-amd:
  candy:
    base: cachyos
    candy:
      - '@github.com/opencharly/pod-ollama:v2026.243.0411'
      - '@github.com/opencharly/layer-ollama-rocm:v2026.243.1755'
```

## Layout

- `charly.yml` — the `ollama-rocm:` candy entity: the `require:` on
  `pod-ollama`, the `security:` block, the `distro:` package arm, and the
  `plan:` `check:` assertion.
- `CHANGELOG/` — per-CalVer release history.
- `README.md` — this user overview.

## Related

This repo carries no `skill:` entity of its own; `/charly-ollama:ollama` is the closest
family owning procedure.

- Base server: `/charly-ollama:ollama` — the CPU-first Ollama server this adds to.
- NVIDIA alternative: `opencharly/layer-ollama-cuda`.
- ROCm runtime: `/charly-distros:rocm`.
- [`opencharly/opencharly](https://github.com/opencharly/opencharly) — the umbrella.
