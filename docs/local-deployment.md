# Lunatic Lorry local desktop deployment

This repository uses `ORESoftware/ores-compose` for the declared laptop/desktop lifecycle.

## Local orchestration

```sh
ores-compose check .ores-compose.yaml
ores-compose plan .ores-compose.yaml
ores-compose up .ores-compose.yaml
```

Stop it from another terminal with:

```sh
ores-compose down .ores-compose.yaml
```

The manifest pins exact daemon revision `df380a882a457706fcf2fe1dffc0e2e532aa5fad` under `tmp/dev`, builds it with `cargo build --release --locked`, executes the built release binary directly, and binds only to the loopback address recorded in `appliance.json`.

That merge commit has tree `902fa0cec861a5aa2f3257d1e02d44baeedbff31`, identical to exact tested PR #21 head `4369c7556c07fb0580f4b001b06b34369c538289`. The reviewed head's `rust-ci` run `36497867179` completed successfully. The merged revision contains a Cargo v4 lockfile, so the lockfile promotion gate is now truthful and the compose build fails closed if dependency resolution drifts.

## Cloudflare boundary

The daemon remains loopback-only. Public ingress is gated because `/v1/invoke` currently uses the local desktop-control bearer; a distinct remote-auth bridge must land before a runnable public compose profile is added. Cloudflare credentials must never be committed or copied into plaintext config.

## Common desktop implementation layer

The appliance consumes `ORESoftware/ores-common-desktop-infra` at exact merge commit `20ec084cc550c824c009d5413de84ab519081bd7`. `common_layer_ci_verified` remains false until this exact consumer pin completes stepful shared-validator proof.

The desktop-contract workflow therefore runs the historical product-specific gate plus the shared Rust `validate_appliance_contract` binary from the exact common checkout. The Ruby validator is temporary parity evidence; it must not be removed by weakening coverage.

## One-shot Lunatic/WASM boundary

Lunatic-Lorry uses one resident native daemon/runtime process but creates fresh isolated Lunatic actor/process state for every invocation. Compiled module caching is allowed; actor state reuse is not. Completion still requires end-to-end proof for launch/invoke/cancel/status/logs/teardown, timeout cleanup, crash recovery, stale-state cleanup, port conflicts, tunnel reconnect, and no cross-invocation actor-state leakage.

Mutable branches or tags are not acceptable release dependencies.
