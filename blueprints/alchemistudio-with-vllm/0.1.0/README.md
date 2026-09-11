# alchemistudio-with-vllm 0.1.0

Deploys **alchemistudio-platform 1.0.23** + **vllm** and wires them together.

## Prerequisites
- **SUSE AI Factory** (aif-operator) on the target RKE2/Rancher cluster.
- A **GPU node** with runtimeclass `nvidia` and a **free 16 GB GPU** (e.g. T4). vLLM takes
  the whole GPU — scale any NIM to 0 first (`kubectl -n nim patch nimservice <NAME>
  --type=merge -p '{"spec":{"replicas":0}}'`).
- A **kata-enabled** node (the AlchemiStudio code sandbox runs kata-isolated).
- StorageClass **`local-path`** for the vLLM model cache (the GPU node is typically not a
  Longhorn node).
- **cert-manager** for `tls.mode=acme` (or bring your own cert for `byo`).
- **Egress** to pull the vLLM + TEI images and the Qwen + bge-small weights
  (`dp.apps.rancher.io`, `ghcr.io`, `huggingface.co`) — mirror all of these for air-gap.
- The chart repo registered: `kubectl apply -f ../../clusterrepo/alchemi-helm-charts.clusterrepo.yaml`.

## Configure before installing (edit `blueprint.yaml`)
Blueprints render their **baked** values (the AI Factory UI has no per-value form, and
AIWorkload `componentValues` are not applied to blueprint components), so edit the YAML
before you register it:

- **Hostname — set EITHER `global.domain` OR `gateway.loadBalancerIP`** (one is required;
  the install **fails fast if BOTH are blank**). Use `global.domain` for a real domain, or
  `gateway.loadBalancerIP` (a reserved static IP) for a DNS-less test — the app is then
  served at `<ip>.nip.io` automatically. See "Getting a working hostname" below.
- **`global.imageCredentials`** — a scoped, **pull-only** ACR token is **baked in** (required
  to pull the app images; distributed the same way as the partner charts). For a real customer
  deployment you can instead mirror the images into your own registry and set your own creds.
- **admin** — `admin.email` / `admin.initialPassword`. The first user gets the Copilot seat.
- **TLS** — `tls.mode=acme` (Let's Encrypt) or `byo`.
- **Off by default:** guardrails + ClamAV (`guardrails.enabled=false`). Enable for a
  hardened deployment.

## Getting a working hostname (domain or static IP)
The install needs **either** a real domain **or** a pinned static IP. If **both** are blank
it **stops** with a clear message (no usable hostname). Pick one:

- **A — Real domain (recommended / production).** Set `global.domain` (e.g.
  `alchemi.acme.com`), install, read the gateway IP (`kubectl -n apisix get svc
  apisix-gateway`), and create DNS A-records for each subdomain (`alchemi-app.<domain>`,
  `alchemi-auth.<domain>`, …) → that IP. The TLS cert is **pending until those A-records
  resolve** — once they do, cert-manager issues a trusted Let's Encrypt cert automatically.
- **B — Static IP, no domain (quick trusted test — this is what we use).** Leave
  `global.domain` blank and set **`gateway.loadBalancerIP`** to a reserved static IP. The
  chart pins the gateway to that IP and serves the app at `<ip>.nip.io` — no DNS setup, and
  Let's Encrypt issues a trusted cert in **one phase** (`nip.io` resolves the host to your
  IP immediately). `gateway.loadBalancerIP` is **one portable knob** — it works on
  **bare-metal (MetalLB)** and **Azure** alike (on Azure the IP must be a pre-reserved
  Public IP in the cluster's own resource group).
  - *Escape hatch:* if your reserved IP lives in a **different** resource group (Azure) or
    your LB needs a provider-specific hint, use `gateway.serviceAnnotations` instead (e.g.
    Azure `service.beta.kubernetes.io/azure-pip-name` + `…-resource-group`, or AWS
    `aws-load-balancer-eip-allocations`).
- **C — Two-phase, no reservation (DNS-less).** Install with a placeholder `global.domain`,
  read the assigned gateway IP, set `global.domain: "<ip>.nip.io"`, and **re-apply** the
  blueprint (a Fleet *upgrade* keeps the same apisix IP). The cert issues only after this
  **second** apply, and the IP isn't stable across a full teardown — so prefer **B** whenever
  you can reserve an IP.

> nip.io is a free wildcard DNS that maps `anything.<IP>.nip.io → <IP>` — handy for tests, no
> DNS setup. For anything lasting, use a real domain (path A).

## Install
```bash
kubectl apply -f blueprint.yaml           # register (with your domain set)
# then AI Factory UI: Blueprints -> AlchemiStudio + vLLM -> Install -> Fleet Bundle -> Install
```

## After install
```bash
# gateway IP (for DNS A-records if you set a real domain):
kubectl -n apisix get svc apisix-gateway
# app URLs (computed from your domain + subdomains):
kubectl -n alchemi get configmap alchemi-endpoints \
  -o jsonpath='App:  {.data.APP_URL}{"\n"}Auth: https://{.data.AUTH_HOST}{"\n"}Ops:  https://{.data.OPS_HOST}{"\n"}'
# login credentials:
kubectl -n alchemi get configmap alchemi-endpoints -o jsonpath='{.data.ADMIN_EMAIL}' ; echo
kubectl -n alchemi get secret alchemi-secrets -o jsonpath='{.data.FIRST_USER_PASSWORD}' | base64 -d ; echo
# verify the model wiring (account-scoped provider + model):
kubectl -n alchemi-data exec alchemi-pgvector-0 -- \
  psql -U alchemiadmin -d alchemiapp_db \
  -c "select scope, provider_id, deployment_name from config_models where provider_id in ('suse_open','suse_embed');"
```

Log in, pick **Qwen2.5 3B (SUSE vLLM)** in the Copilot model dropdown, and chat — the seat,
credits, pricing, and provider credential are already seeded. (vLLM takes ~2–3 min after the
pods come up to download the model weights on first install.)
