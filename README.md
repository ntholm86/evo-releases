<p align="center">
  <strong>evo</strong><br>
  <em>Autonomous Software Evolution Engine</em>
</p>

<p align="center">
  <a href="https://github.com/ntholm86/evo-releases/releases/latest"><img alt="Latest Release" src="https://img.shields.io/github/v/release/ntholm86/evo-releases?style=flat-square&label=latest"></a>
  <a href="https://github.com/ntholm86/evo-releases/releases"><img alt="Downloads" src="https://img.shields.io/github/downloads/ntholm86/evo-releases/total?style=flat-square"></a>
  <img alt="Python" src="https://img.shields.io/badge/python-3.11%2B-blue?style=flat-square">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-green?style=flat-square">
</p>

---

**Most AI coding tools wait for you to ask.** evo doesn't wait.

Point it at any git repo. Tell it what "better" means — more tests, fewer bugs, higher coverage. Walk away. evo runs a **closed-loop evolution pipeline** that analyzes weaknesses, writes code, verifies nothing regressed, and merges — autonomously, with a cryptographic proof trail.

It doesn't just generate code. It **evolves codebases**.

---

## What makes evo different

- **Zero human intervention** — evo runs end-to-end: analyze → propose → implement → verify → decide → merge. No prompting, no copy-paste, no review queue.
- **Regression-proof** — Every change passes a Pareto gate. If *any* metric regresses — tests, lint, coverage, security — the change is rejected. Period.
- **Self-improving** — evo uses its own merge rate as a fitness signal and can evolve *its own source code*. The engine that writes code can rewrite itself.
- **Tamper-evident audit trail** — A hash-chained proof ledger records every decision. Immutable. Verifiable.
- **Multi-language** — Python, TypeScript, JavaScript, C#. Auto-detects test frameworks, linters, and security scanners.
- **Statistically rigorous** — Noisy metrics (benchmarks, timing) use Welch's t-test instead of naive comparison. No false positives from measurement noise.

## Real results

v0.23.0, 3 iterations per repo, Claude Haiku (~$0.04/run):

| Repo | Language | Tests Before → After | Merge Rate | Cost | Time |
|------|----------|---------------------|------------|------|------|
| mathkit | Python | 43 → 99 | 100% | $0.13 | 103s |
| datakit | Python | 7 → 73 | 100% | $0.13 | 102s |
| i18nkit | Python | 5 → 27 | 100% | $0.08 | 78s |
| textkit | Python | 12 → 41 | 67% | $0.15 | 128s |
| clikit | Python | 16 → 32 | 67% | $0.20 | 179s |

**87% merge rate. +186 tests added. $0.69 total. ~10 minutes.**

## How it works

```
┌─────────┐     ┌─────────┐     ┌───────────┐     ┌────────┐     ┌────────┐     ┌─────────┐
│ ANALYZE │────▶│ PROPOSE │────▶│ IMPLEMENT │────▶│ VERIFY │────▶│ DECIDE │────▶│ RELEASE │
│ find    │     │ plan    │     │ code +    │     │ tests  │     │ Pareto │     │ merge + │
│ weakness│     │ fix     │     │ auto-fix  │     │ lint   │     │ gate   │     │ proof   │
└─────────┘     └─────────┘     └───────────┘     └────────┘     └────────┘     └─────────┘
                                      ▲                │
                                      └── DIAGNOSE ◀───┘  (retry on failure)
```

Each iteration creates a branch, makes one focused change, and only merges if it's a strict improvement. Failed attempts are rejected and logged — evo learns from them.

## Branch naming & workflow

Every evo iteration creates a branch with a deterministic name:

```
evo/{NNN}-{slug}
```

- `NNN` — zero-padded iteration number (001, 002, …)
- `slug` — sanitized proposal description (e.g. `add-edge-case-tests-for-divide`)

**Examples:**
```
evo/001-add-input-validation-to-parse-config
evo/002-test-coverage-for-metric-aggregation
evo/003-harden-error-handling-in-git-module
```

**Lifecycle:** branch is created → LLM writes code → build check → tests run → Pareto gate evaluates. On **MERGE**, the branch is squash-merged into `master` with a structured commit message containing metrics, proof hash, and cost. On **REJECT**, the branch is deleted — nothing touches `master`.

**Strategy** determines *what kind* of proposals get generated:

| Strategy | Branch proposals target |
|----------|------------------------|
| `test_coverage` | Missing tests, edge cases, untested paths |
| `hardening` | Bug fixes, input validation, error handling |
| `balanced` | Mix of tests + hardening + quality |
| `full_auto` | LLM decides autonomously what to improve |

## Install

Download the latest `.whl` from [Releases](https://github.com/ntholm86/evo-releases/releases/latest), then:

```bash
pip install evo_engine-*.whl
```

### Prerequisites

- Python 3.11+
- Git (target repo must be a git repository)
- Anthropic API key

### Quick start

```bash
# Set your API key
echo "ANTHROPIC_API_KEY=your-key" > .env

# Run against any repo
evo run --repo /path/to/your/project --iterations 5 --no-sandbox
```

### Configuration

Drop a `.evo.yaml` in your repo:

```yaml
project: my-app
language: python
strategy: balanced          # test_coverage | hardening | balanced | full_auto

metrics:
  test: "pytest --tb=short -q"

allowed_scopes:
  - "src/**"
  - "tests/**"
```

## Strategies

| Strategy | Focus | Best For |
|----------|-------|----------|
| `test_coverage` | Missing tests, edge cases | Safe first pass — grow coverage without touching source |
| `hardening` | Bugs, error handling | Making existing code robust |
| `balanced` | Tests + bugs + quality | General improvement |
| `full_auto` | Everything — LLM decides | Maximum autonomy |

## Safety guarantees

- **Scope enforcement** — AI can only touch files in `allowed_scopes`
- **Pareto gate** — no metric can regress, ever
- **Hash-chained proof ledger** — every decision is immutable and auditable
- **Cost caps** — configurable per-run spending limits
- **Docker sandbox** — optional isolated execution environment

## License

MIT
