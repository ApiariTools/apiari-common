# apiari-common

Shared library crate for the Apiari toolchain.

## Quick Reference

```bash
cargo test -p apiari-common    # Run tests (25 unit + 2 doctests)
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
  shell.rs     # shell_quote(), sanitize()
  signal.rs    # Signal struct, Severity enum
  state.rs     # load_state<T>(), save_state<T>() with atomic writes
```

## Design Rules

- **No heavy dependencies.** This crate uses `std::io::Result` (not color-eyre). Keep the dependency footprint minimal.
- **Only shared types belong here.** If a type is only used by one crate, it stays in that crate. A type moves here when 2+ crates need it.
- **Generic over `T`.** `JsonlReader<T>` and `JsonlWriter<T>` are generic over any `Serialize + DeserializeOwned` type. `load_state` and `save_state` are similarly generic.
- **Atomic writes.** `save_state` writes to a `.tmp` file then renames. This prevents partial/corrupt reads.
- **Cursor-based polling.** `JsonlReader` tracks a byte offset. `poll()` reads only new lines since the last call. `skip_to_end()` jumps to EOF without reading.

## Integration Map

Used by all other Apiari crates:

| Crate | Uses |
|-------|------|
| swarm | `shell::shell_quote`, `shell::sanitize` |
| buzz | `signal::Signal`, `signal::Severity`, `ipc::JsonlWriter`, `state::save_state` |
| hive | `state::save_state`, `state::load_state` |
| keeper | `signal::Signal` types (has mirror structs for deserialization) |
| claude-sdk | Does not use common |

## Key Types

- `Signal` fields: id (UUID), source, severity, title, body, url (Option), timestamp, tags (Vec), dedup_key (Option)
- `Severity`: Critical, Warning, Info (implements Display, Ord)
- `JsonlReader<T>`: new(), with_offset(), offset(), poll(), skip_to_end()
- `JsonlWriter<T>`: new(), path(), append()
