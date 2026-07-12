# pytest-deltatest

> **Run only the tests affected by your code changes — in one flag.**

```bash
pip install pytest-deltatest
pytest --delta
```

No config. No CI changes. Just faster feedback.

---

## Why?

Running your full test suite on every change is slow. Most of the time, a 2-line change affects **3 tests out of 10,000**.

`pytest-deltatest` builds a precise map of *which tests cover which lines*, then on every run it diffs your git changes and runs only the tests that touch those lines.

**Real-world benchmark:**

| Project | Full suite | With `--delta` | Saved |
|---|---|---|---|
| 10,350 tests | 8 min 42 sec | 23 sec | **95.6%** |
| 2,800 tests | 2 min 10 sec | 8 sec | **93.8%** |
| 640 tests | 38 sec | 4 sec | **89.5%** |

---

## Quickstart

### 1. Install

```bash
pip install pytest-deltatest
```

### 2. Build the mapping (one-time)

```bash
delta build-mapping
```

This runs your full suite once with coverage and stores a compact SQLite map of every test → every line it touches. Takes as long as your full suite, but only needs to be done once (and self-heals as you add new tests).

### 3. Run affected tests

```bash
pytest --delta
```

That's it. From now on, every `pytest --delta` call will:
1. `git diff` your working tree against `master`
2. Look up which tests cover the changed lines
3. Run *only those tests*
4. Auto-include any unmapped (new) tests so nothing slips through

---

## Options

```
pytest --delta                  Run only affected tests
pytest --delta --delta-local    Use local DB only (skip cloud sync)
pytest --delta --delta-base develop   Diff against a different base branch
```

Pass any normal pytest flags alongside `--delta`:

```bash
pytest --delta -x -v            Stop on first failure, verbose output
pytest --delta --tb=short       Short tracebacks
pytest --delta -k "auth"        Further filter by keyword
```

---

## How it works

```
git diff master
    ↓
Look up changed lines in .delta/test_mapping.db
    ↓
Select only tests covering those lines
    ↓
Run selected tests + any unmapped (new) tests
    ↓
Update mapping with new coverage data
```

The mapping is a compact SQLite database stored at `.delta/test_mapping.db`. It stores line *ranges* (not individual lines) so it stays small even for large repos.

Each entry: `test_name → [(file, start_line, end_line), ...]`

---

## CI Integration

```yaml
# .github/workflows/test.yml
- name: Run affected tests
  run: pytest --delta --delta-base ${{ github.base_ref }}
```

For full suite validation (e.g. on merge to main), just run `pytest` without `--delta`.

---

## Subprocess / xdist support

`pytest-deltatest` works with `pytest-xdist` for parallel execution:

```bash
pytest --delta -n auto
```

Large test lists are passed via a temp JSON file so subprocess boundaries don't truncate argument lists.

---

## Commands

### `delta build-mapping`

Build (or resume building) the test mapping database.

```bash
delta build-mapping [--verbose] [--test-dir tests] [--local]
```

### `delta run`

Run affected tests via the CLI (alternative to `pytest --delta`).

```bash
delta run [--dry-run] [--base-branch develop] [--explain] [-v]
```

### `delta status`

Inspect the mapping database.

```bash
delta status
```

Example output:
```
Local DB: .delta/test_mapping.db
  Tests:    10,350
  Files:    892
  Mappings: 4,231,890
  Size:     18.4 MB
```

### `delta install`

Install a git pre-commit hook that automatically runs `pytest --delta` before every commit.

```bash
delta install
```

---

## VS Code Extension

The **Python DeltaTest** extension shows inline coverage glows and lets you run affected tests from the editor sidebar.

[Install from VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=deltatest.delta-coverage) — search `DeltaTest`.

---

## Cloud sync (Team mode)

Share mapping databases across your team and CI/CD via [deltatest.dev](https://deltatest.dev).

```toml
# ~/.delta/config.toml
[cloud]
api_key  = "pt_live_..."
repo_id  = "my-org/my-repo"
branch   = "main"
```

---

## Installation requirements

- Python 3.8+
- pytest ≥ 7.0
- pytest-cov ≥ 4.0
- Git

---

## Contributing

Pull requests welcome. See [github.com/deltatest-org/delta](https://github.com/deltatest-org/delta).

---

## License

MIT
