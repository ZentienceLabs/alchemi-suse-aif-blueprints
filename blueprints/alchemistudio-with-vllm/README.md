# alchemistudio-with-vllm

**AlchemiStudio + vLLM (Qwen, open stack)** — the AlchemiStudio AI platform pre-wired to
an in-cluster **open** model: vLLM serving `Qwen/Qwen2.5-3B-Instruct` (Apache-2.0) as the
Copilot chat model, plus a bundled CPU embedding server (bge-small via HuggingFace TEI) for
RAG. Fully self-hosted — **no** NVIDIA NGC key, **no** HuggingFace token, **no** cloud API
key, and no LiteLLM (AlchemiStudio is its own gateway).

The blueprint auto-configures AlchemiStudio to use the model (provider + model seeded,
priced, credentialed, account-scoped; the Copilot function roles mapped to it; the first
user granted the Copilot seat + initial credits) — no console clicks.

## Components
1. **alchemistudio-platform** — the app + bundled embeddings + the model wiring seed.
2. **vllm** — Qwen2.5-3B on the GPU (tool-calling + fast restarts).

## Versions
| Version | Chart | Notes |
|---|---|---|
| [0.1.0](0.1.0) | alchemistudio-platform 1.0.23 | Initial open-stack blueprint. |

See [0.1.0/README.md](0.1.0/README.md) for prerequisites and the full install/verify steps.
