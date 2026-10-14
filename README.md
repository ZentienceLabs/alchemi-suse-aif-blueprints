# AlchemiStudio — SUSE AI Factory Blueprints

Partner-contributed **[SUSE AI Factory](https://documentation.suse.com/suse-ai-factory/)**
Blueprints for the **AlchemiStudio** platform.

A *Blueprint* bundles one or more Helm charts with baked defaults so an operator can
stand up a complete stack from the AI Factory UI — no per-value form. The Helm chart the
blueprint deploys (`alchemistudio-platform`) is published separately as a Rancher App /
Helm repo; this repo holds only the **Blueprint definitions**.

> **This is our staging / testing copy.** The **official** AlchemiStudio blueprint lives in
> [SUSE/partner-blueprints](https://github.com/SUSE/partner-blueprints) (`partners/alchemistudio/`)
> and pulls the chart from the `rancher-partner-charts` ClusterRepo. This copy pulls from our own
> Helm repo instead, so new chart versions can be tested through our Fleet GitRepo before they
> reach Rancher partner charts. Use the SUSE copy for real installs.

## Blueprints

| Blueprint | What it deploys |
|---|---|
| [`alchemistudio-with-vllm`](blueprints/alchemistudio-with-vllm) | AlchemiStudio + vLLM serving Qwen2.5-3B (Apache-2.0) as the Copilot model + a bundled CPU embedding server — fully self-hosted, no NGC / HuggingFace / cloud keys. |

## Install (per cluster)

1. **Register the chart repo** the blueprint pulls from (once per cluster):
   ```bash
   kubectl apply -f clusterrepo/alchemi-helm-charts.clusterrepo.yaml
   ```
2. **Choose how the app is reached** in the blueprint — set **one** of `global.domain`
   (real domain) or `gateway.loadBalancerIP` (static IP, served at `<ip>.nip.io`) — then
   register it:
   ```bash
   # edit blueprints/alchemistudio-with-vllm/0.1.0/blueprint.yaml
   kubectl apply -f blueprints/alchemistudio-with-vllm/0.1.0/blueprint.yaml
   ```
3. **Install from the AI Factory UI**: Blueprints → *AlchemiStudio + vLLM* → Install →
   Deployment Type **Fleet Bundle** → target cluster → Review → Install.

Full prerequisites + post-install steps are in each blueprint's
[version README](blueprints/alchemistudio-with-vllm/0.1.0).

## Notes
- **Gateway (APISIX):** the chart deploys **Apache APISIX** into the **`apisix` namespace** by
  default (`gateway.mode: bundled`) as the required ingress / TLS / OIDC-routing layer for the
  app hostnames — it is core architecture, not an optional or leftover component. Set
  `gateway.mode: external` to route through your own gateway instead. See the per-version
  README for details.
- **Images:** the blueprint pulls AlchemiStudio images from a registry via
  `global.imageCredentials`. A scoped **pull-only** ACR token is baked in (required for image
  pull; distributed the same way as the partner charts), or mirror the images into your own
  registry and set your own creds. (SUSE / NVIDIA images use the operator's own pull secret.)
- **The chart is the single source of truth** — the same `alchemistudio-platform` chart
  also ships as a Rancher App; this repo never forks or copies the chart, only references it.
