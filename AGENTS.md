# AGENTS.md — layer-ollama-rocm

Standalone candy repo for the `ollama-rocm` add-on layer — the ROCm/HIP `ggml`
backend that lets the Ollama LLM server run inference on an AMD GPU. The candy
lives in `charly.yml` at the repo root: the `require:` on `pod-ollama`, the
`security:` block (`group_add: keep-groups`), the `distro:` package arm, and the
`plan:` `check:` assertion.

This repo has **no `skill:` entity** in `charly.yml`, so there is no dedicated
owning skill projected into the marketplace corpus. The gap is recorded against
`opencharly/opencharly#291` (the batch that authors missing `skill:` entities).

Canonical files:

- `charly.yml` — the `ollama-rocm:` candy entity.
- `CHANGELOG/` — per-CalVer release history.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-ollama:ollama` — the base server this candy adds to, and the
  composition contract for GPU backends. Load before editing or troubleshooting.
- `/charly-distros:rocm` — the AMD ROCm runtime layer whose backend this candy
  contributes. Load before touching the backend path or group-add posture.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, per-distro `distro:` arms, package/repo
  sections, and service declarations). Load before editing any entity field or
  plan step.

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- The candy's `plan:` `check:` step is the functional evidence — it asserts the
  ROCm/HIP `ggml` backend library is installed in Ollama's runner directory.

## Modify this repo

- There is no `skill:` entity to keep in sync; if one is added (per #291), it
  must be edited together with the candy entity in the same change.
- Keep the backend path **version-agnostic**: the runner directory carries the
  ROCm version in its name, so a fixed path would rot on the next ROCm bump.
- Keep `group_add: keep-groups` — it is what makes a passed-through AMD GPU
  usable from the container.
- This is an add-on: it contributes a backend, never the server.
- New behaviour claims belong in the `plan:` as an observable `check:` step.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge
  on PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the
  PR body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
