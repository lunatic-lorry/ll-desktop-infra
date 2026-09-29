# Lunatic Lorry Desktop Infra

Single-host desired-state and local deployment authority for developer- or end-user-owned desktops/laptops.

## ORES Compose local deployment

The audited local lifecycle is declared in `.ores-compose.yaml`:

```sh
ores-compose check .ores-compose.yaml
ores-compose plan .ores-compose.yaml
ores-compose up .ores-compose.yaml
```

Public ingress is intentionally gated until `/v1/invoke` has a remote-auth credential separate from the local desktop-control bearer.

The daemon source is exact-commit pinned to merged revision `df380a882a457706fcf2fe1dffc0e2e532aa5fad`, loopback-only, built with its committed Cargo v4 lockfile via `cargo build --release --locked`, and executed from the built release binary. That merge has the same tree as exact tested PR #21 head `4369c7556c07fb0580f4b001b06b34369c538289`.

See [docs/local-deployment.md](docs/local-deployment.md) and [appliance.json](appliance.json) for the audited boundary and remaining promotion gates.

## Shared desktop infra dependency

Generic desktop lifecycle/security behavior lives in `ORESoftware/ores-common-desktop-infra`. The appliance pins merged common revision `20ec084cc550c824c009d5413de84ab519081bd7`; mutable branch dependencies are forbidden. `common_layer_ci_verified` remains false until this exact consumer pin receives stepful shared-validator evidence.

During migration CI dual-runs the historical product admission gate and the shared Rust `validate_appliance_contract` binary from that exact common revision. The Ruby gate is temporary parity evidence, not the long-term authority.

## Runtime semantics

Lunatic-Lorry remains a one-shot/non-reusable actor runtime. The resident daemon may cache compiled WebAssembly modules, but every invocation must receive a fresh isolated Lunatic actor/process state. No actor state may survive into another invocation. Timeout/cancel, crash recovery, bounded output, immutable deployment evidence, and deterministic actor cleanup remain runtime-specific completion gates.

## Hot-reload routing and middleware

This product consumes the shared ORES generation model with **native atomic routes + Lunatic/Wasm middleware generations** as its default. Routing/middleware is a separate lifecycle and memory/failure boundary from standalone servers and lambda/actor workers, so route or middleware updates do not restart unrelated compute.

`hot-reload-policy.json` declares the product policy. The edge may optionally use nginx, HAProxy, or Caddy. nginx uses validated worker-generation reloads; HAProxy prefers Runtime API changes and falls back to master-worker reload for structural changes; Caddy uses its transactional Admin API. Proxy-managed application routes are opt-in and limited to declarative routing/middleware. Arbitrary middleware code stays in BEAM, Wasm, or a separately supervised process generation.

Long-lived WebSockets/streams are bounded by a hard generation drain timeout so repeated reloads cannot accumulate old generations indefinitely.
