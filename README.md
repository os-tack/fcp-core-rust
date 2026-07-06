# fcp-core-rust

**Deprecated.** This crate has no consumers: [fcp-rust](https://github.com/os-tack/fcp-rust) and
[fcp-regex](https://github.com/os-tack/fcp-regex) each vendor their own self-contained copy of the
FCP core modules under `src/fcpcore/` and do not depend on this crate. The last published
crates.io version is [`fcp-core` v0.1.1](https://crates.io/crates/fcp-core); v0.1.2 is the final
release before deprecation. It is kept around for reference only — do not build new servers
against it; port the modules you need directly into your server's own tree instead, as fcp-rust
and fcp-regex do.

Shared Rust framework for building [FCP](https://github.com/os-tack/fcp) (File Context Protocol) servers.

## What It Provides

fcp-core-rust extracts the common infrastructure an FCP server needs — tokenizer, operation parser, verb registry, event log, session lifecycle, and response formatter — into a standalone library crate.

This is the Rust equivalent of the [fcp-core](https://github.com/os-tack/fcp-core) TypeScript/Python package.

## Modules

| Module | Purpose |
|--------|---------|
| `tokenizer` | Quote-aware tokenizer — splits FCP operation strings on whitespace, handles quoted strings and escape sequences |
| `parsed_op` | Operation parser — classifies tokens into verb, positionals, key:value params, and @selectors |
| `verb_registry` | Verb specification registry with reference card generation grouped by category |
| `event_log` | Cursor-based event log with undo/redo and named checkpoints |
| `session` | Session lifecycle dispatcher (new/open/save/checkpoint/undo/redo) with domain hooks trait |
| `formatter` | Response prefix formatting (`+` created, `*` modified, `-` removed, `=` result, `!` error) and Levenshtein-based suggestions |

## Usage

Add to your `Cargo.toml`:

```toml
[dependencies]
fcp-core = { git = "https://github.com/os-tack/fcp-core-rust" }
```

### Parsing Operations

```rust
use fcp_core::parsed_op::parse_op;

let op = parse_op("define digits any:digit+").unwrap();
assert_eq!(op.verb, "define");
assert_eq!(op.positionals, vec!["digits"]);
assert_eq!(op.params["any"], "digit+");
```

### Event Log (Undo/Redo)

```rust
use fcp_core::event_log::EventLog;

let mut log = EventLog::new();
log.append("created widget");
log.append("renamed widget");
log.checkpoint("v1");
log.append("deleted widget");

let undone = log.undo(1);           // undoes "deleted widget"
let undone = log.undo_to("v1");     // undoes back to checkpoint
```

### Verb Registry

```rust
use fcp_core::verb_registry::{VerbRegistry, VerbSpec};

let mut reg = VerbRegistry::new();
reg.register(VerbSpec {
    name: "define".into(),
    syntax: "define NAME ELEMENT [ELEMENT...]".into(),
    category: "fragments".into(),
});
let card = reg.generate_reference_card(None);
```

## Development

```bash
cargo test          # Run all tests (95)
cargo clippy        # Run lints
```

No external dependencies — pure Rust standard library only.

## License

MIT
