# sandyhq/zengx: what this fork adds to zeronsh/zgx

[zeronsh/zgx](https://github.com/zeronsh/zgx) with the edits
[Zend](https://github.com/sandyhq/zend) needs, as commits on `main`. Zend's
`apps/zgx` builds against this repository at one pinned commit (every
`rev = "…"` in its `Cargo.toml`).

`main` is upstream's `main` with these edits on top. It takes upstream by
merging (below), so every commit Zend ever pinned stays on `main`.

## The edits

| Edit | Files | Why (in Zend) |
|---|---|---|
| `ScriptHost::on_print`: each line a script prints, with its owner | `crates/zgx-script/src/lib.rs`, `node.rs`, `tests/calls.rs` | plugin logs: a plugin's prints reach the engine |
| `Ui::on_draw` / `Ui::draw`; `draw_alone` in `application.rs` | `crates/zgx/src/widgets.rs`, `crates/zgx-desktop/src/application.rs` | plugin_status snapshots and page pictures, also of a window that isn't shown (Linux hands over no frame) |
| `ZGX_OFFSCREEN` on Linux: nothing presented, frames on a timer, no application ID; `application::offscreen()` | `crates/zgx-desktop/src/application.rs` | plugin previews and stand-in instances: a Wayland surface given a buffer is mapped, hidden or not |
| A window drawn on the software raster pictures nodes on a headless GPU context of its own (`picture_context`) | `crates/zgx-desktop/src/application.rs` | snapshots where the only adapter is a software one (`ZGX_RENDERER=auto` picks the raster there) |

Upstream has `ZGX_OFFSCREEN` too (`window_chrome::OFFSCREEN`, `may_show`:
never ordered in, invisible on macOS). The third edit adds only what
Linux needs beyond it, and reads upstream's flag. Where both did the
same, upstream's code is kept.

None of these is upstream as of the commit they sit on (see `git log`).

### Not here

- **Zend's rebrand** (Zeron → Zend in zgx's examples, tests and comments,
  `ZERON_SEND`/`ZERON_MODEL` hooks of the `zeron` example). It stays out on
  purpose. Zend builds no zgx example or test, and the only library string
  it touched is a D-Bus object path any name serves for. In here it would be
  about 250 changed lines in 55 files, which every upstream merge would have
  to resolve again, for nothing Zend uses.
- **Vendoring edits** (`workspace = "../.."` in each crate, a trimmed
  members list): only a copy inside another repository needed them.

## Taking upstream

Merge, never rebase and never force-push: Zend pins commits of `main`.

```sh
git remote add upstream git@github.com:zeronsh/zgx.git   # once
git fetch upstream
git checkout main
git merge upstream/main          # resolve the edits above against upstream
cargo nextest run --workspace --no-fail-fast --locked
```

Compare the failing test names with upstream's own commit, in a worktree:
`git worktree add --detach ../zgx-plain upstream/main`, the same command
there with another `CARGO_TARGET_DIR`. No new names, then
`git push origin main`, a fast-forward. Then move Zend's pin (Zend's
`apps/zgx/README.md`, "zgx").

A new edit for Zend is a commit on `main` with its row in the table above.
Better still, send it upstream: once upstream has it, the next merge
resolves it to upstream's code, and its row goes.
