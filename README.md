# apiari-common

Shared library for the [Apiari](https://github.com/ApiariTools) toolchain. Provides types and utilities used by swarm, buzz, hive, and keeper.

## Modules

### `shell`

Shell quoting and sanitization.

```rust
use apiari_common::shell::{shell_quote, sanitize};

assert_eq!(shell_quote("it's fine"), "'it'\\''s fine'");
assert_eq!(sanitize("Fix the BUG!"), "fix-the-bug");
```

### `ipc`

Generic JSONL reader/writer with cursor-based polling. Tracks byte offsets for incremental reads.

```rust
use apiari_common::ipc::{JsonlWriter, JsonlReader};

let writer = JsonlWriter::<MyMessage>::new(".swarm/inbox.jsonl");
writer.append(&msg)?;

let mut reader = JsonlReader::<MyMessage>::new(".swarm/inbox.jsonl");
let new_messages = reader.poll()?;
```

### `signal`

The shared `Signal` type that buzz produces and hive/keeper consume.

```rust
use apiari_common::signal::{Signal, Severity};

let signal = Signal::new("sentry", Severity::Critical, "OOM in prod", "Worker killed")
    .with_url("https://sentry.io/issues/123")
    .with_tags(vec!["production".into()]);
```

### `state`

Atomic JSON state persistence (write to temp file, then rename).

```rust
use apiari_common::state::{load_state, save_state};

let state: MyState = load_state("state.json")?; // returns Default if missing
save_state("state.json", &state)?;              // atomic write
```

## Usage

Add to your `Cargo.toml`:

```toml
[dependencies]
apiari-common.workspace = true
```
