# apiari-common

Minimal shared library crate for the Apiari toolchain.

## Quick Reference

```bash
cargo test -p apiari-common    # Run tests (11 unit tests)
cargo doc -p apiari-common     # Generate docs
```

## Git Workflow

- You are working in a swarm worktree on a `swarm/*` branch. Stay on this branch.
- NEVER push to or merge into `main` directly.
- When done, create a PR from your branch. Swarm will handle merging.
- NEVER run `git push origin main` or `git checkout main`.

## Architecture

```
src/
  lib.rs       # Module declarations
  ipc.rs       # JsonlReader<T> / JsonlWriter<T> with byte-offset cursor
  state.rs     # load_state<T>(), save_state<T>() with atomic writes
```

## Design Rules

- **No heavy dependencies.** This crate uses `std::io::Result` (not color-eyre). Only deps are serde + serde_json.
- **Only shared types belong here.** If a type is only used by one crate, it stays in that crate. A type moves here when 2+ crates need it.
- **Generic over `T`.** `JsonlReader<T>` and `JsonlWriter<T>` are generic over any `Serialize + DeserializeOwned` type. `load_state` and `save_state` are similarly generic.
- **Atomic writes.** `save_state` writes to a `.tmp` file then renames. This prevents partial/corrupt reads.
- **Cursor-based polling.** `JsonlReader` tracks a byte offset. `poll()` reads only new lines since the last call. `skip_to_end()` jumps to EOF without reading.

## What moved out

- `signal.rs` (Signal, Severity) -> `hive::signal` (only hive uses it now)
- `shell.rs` (shell_quote, sanitize) -> `swarm::core::shell` (only swarm uses it)

## Integration Map

| Crate | Uses |
|-------|------|
| swarm | `ipc::JsonlReader`, `ipc::JsonlWriter`, `state::load_state`, `state::save_state` |
| hive | `ipc::JsonlReader`, `ipc::JsonlWriter`, `state::load_state`, `state::save_state` |
| claude-sdk | Does not use common |

## Key Types

- `JsonlReader<T>`: new(), with_offset(), offset(), set_offset(), poll(), skip_to_end()
- `JsonlWriter<T>`: new(), path(), append()
- `load_state<T>(path)`: Load JSON, returns T::default() if missing
- `save_state<T>(path, &T)`: Atomic write via tmp + rename
