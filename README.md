<div align="center">

# MIZAN · ميزان

**An adaptive evaluation engine for AI entering public service.**

</div>

MIZAN evaluates a candidate AI model against a register of governance controls,
stops testing each control as soon as the evidence settles it, and issues a
bilingual (English and Arabic) certificate in which every verdict links to the
probe that produced it and every probe carries a SHA-256 hash.

## How it works

1. A model is submitted to the registry and an intended use case is chosen.
2. An adaptive engine (UCB1 allocation with sequential stopping) spends its
   probe budget on the tests that settle the decision fastest.
3. Every probe result is appended to an immutable, hash-chained evidence log.
4. When all mandatory controls are decided, a signed bilingual certificate is
   issued, stating for each control whether it was decided by a confidence
   bound or at budget exhaustion.

## Quick start

```bash
uv sync --frozen --extra dev && (cd web && npm ci)
make seed     # populate the registry
make dev      # API on 8000, interface on 5173
make test     # full test suite
```

The evaluation path runs offline: model endpoints resolve to deterministic
mocks and fonts are self-hosted, so nothing leaves the machine.

## Layout

```
engine/db/schema.sql   schema, immutability triggers, hash chain
mizan/engine           bandit allocator, stopping rules, strategy search, data access
mizan/agents           suite runners, scorers, endpoint adapters, red-team probes
mizan/api              FastAPI service and the websocket evaluation stream
suites/                control register, use cases, Arabic-native items, cached datasets
web/                   React + Vite interface, bilingual with RTL mirroring
scripts/audit          CI gates
docs/                  design notes and evidence
```

## Gates

Each of these runs in CI on every push:

| Gate | Command |
|---|---|
| Tests | `make test` |
| Register lint | `python3 scripts/audit/register_lint.py` |
| Contrast | `python3 scripts/audit/verify_contrast.py` |
| Evidence chain | `uv run python scripts/verify_evidence.py` |
| Grounding | `python3 scripts/audit/verify_grounding.py` |
| Adaptive vs exhaustive proof | `make prove` |

## Documentation

- [`docs/FLOW.md`](docs/FLOW.md): start here: what runs in what order
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md): schema and module boundaries
- [`docs/CHARTER.md`](docs/CHARTER.md): scope and principles
- [`docs/DECISIONS.md`](docs/DECISIONS.md): choices and rejected alternatives
- [`docs/RISKS.md`](docs/RISKS.md): risks and mitigations
- [`CONTRIBUTING.md`](CONTRIBUTING.md): how to work here

## Status

Pilot-scale research work. The probe corpus is smaller than full statistical
backing requires, so most passing controls are currently decided at budget
rather than by a confidence bound; the certificate states this per control.

## Licence

Source code and original documentation: [Apache License 2.0](LICENSE). Cached
UAE open government datasets and self-hosted typefaces keep their original
licences, see [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md).
