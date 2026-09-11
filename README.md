# AlchemiStudio — SUSE AI Factory Blueprints

Partner-contributed **[SUSE AI Factory](https://documentation.suse.com/suse-ai-factory/)**
Blueprints for the **AlchemiStudio** platform.

A *Blueprint* bundles one or more Helm charts with baked defaults so an operator can
stand up a complete stack from the AI Factory UI — no per-value form. The Helm chart the
blueprint deploys (`alchemistudio-platform`) is published separately as a Rancher App /
Helm repo; this repo holds only the **Blueprint definitions**.

> SUSE is building an official partner-blueprints repository (similar to
> `rancher/partner-charts`). Until then this repo hosts the blueprints, structured so it
> ports there cleanly.

## Blueprints

| Blueprint | What it deploys |
|---|---|
| [`alchemistudio-with-vllm`](blueprints/alchemistudio-with-vllm) | AlchemiStudio + vLLM serving Qwen2.5-3B (Apache-2.0) as the Copilot model + a bundled CPU embedding server — fully self-hosted, no NGC / HuggingFace / cloud keys. |

## Install (per cluster)

1. **Register the chart repo** the blueprint pulls from (once per cluster):
   ```bash
   kubectl apply -f clusterrepo/alchemi-helm-charts.clusterrepo.yaml
   ```
2. **Set your domain** in the blueprint (`global.domain` is blank on purpose — the app
   hostnames are `<subdomain>.<domain>`), then register it:
   ```bash
   # edit blueprints/alchemistudio-with-vllm/0.1.0/blueprint.yaml -> global.domain
   kubectl apply -f blueprints/alchemistudio-with-vllm/0.1.0/blueprint.yaml
   ```
3. **Install from the AI Factory UI**: Blueprints → *AlchemiStudio + vLLM* → Install →
   Deployment Type **Fleet Bundle** → target cluster → Review → Install.

Full prerequisites + post-install steps are in each blueprint's
[version README](blueprints/alchemistudio-with-vllm/0.1.0).

## Notes
- **Images:** the blueprint pulls AlchemiStudio images from a registry via
  `global.imageCredentials`. A scoped **pull-only** ACR token is baked in (required for image
  pull; distributed the same way as the partner charts), or mirror the images into your own
  registry and set your own creds. (SUSE / NVIDIA images use the operator's own pull secret.)
- **The chart is the single source of truth** — the same `alchemistudio-platform` chart
  also ships as a Rancher App; this repo never forks or copies the chart, only references it.
