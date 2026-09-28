# Local lifecycle contract

Lunatic Lorry desktop mirrors production lifecycle semantics while keeping the WASM runtime implementation independent.

States:

- `active`: a fresh Lunatic/WASM process is executing an admitted invocation.
- `idle`: trusted host/runtime may remain resident, but tenant process memory is not retained for invocation reuse.
- `frozen`: an admitted process may be suspended for pressure management; no new messages/requests are admitted.
- `reclaiming`: host memory pressure policy may reclaim/swap pages separately from suspension.
- `draining`: old generation receives no new work while admitted work completes.
- `stopped`: tenant process is gone; immutable WASM module/artifact remains available for restart.

Rules:

1. Tenant actor/process reuse across invocations is forbidden even when compiled WASM/module metadata is cached.
2. Every invocation gets a fresh Lunatic process.
3. Freeze and memory reclamation are separate observability states.
4. Deployment activation is prepare -> validate -> stage -> health-check -> activate -> drain -> retire/rollback.
5. The Rust desktop daemon is the sole machine-lifecycle writer; CLI and desktop app are clients only.
6. Erlang may supervise the long-lived local host while Lunatic tenant processes remain disposable.
7. Payloads use stdin/message IPC/authenticated loopback transport, never argv.
