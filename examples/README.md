# Examples

Starter assets for kicking the tires on LEAF.

## `metabolite-list-primary-metabolism.csv`

A 20-compound starter list covering glycolysis, TCA cycle, amino acids, and
adenylates — useful as a smoke test for the targeted extract pipeline. Use it
in the **Extract** view (Targeted mode) by pointing the metabolite-list picker
at this file.

### Schema

| Column | Required | Description |
|---|---|---|
| `Metabolite` | yes | Display name (any string) |
| `Formula` | yes | Molecular formula (e.g. `C6H12O6`); used to compute monoisotopic mass when `Mass` is missing |
| `RetentionTime` | yes | Expected RT in minutes |
| `Adduct` | yes | Ionization adduct (`M-H`, `M+H`, `M+Na`, `M+NH4`, etc.) |
| `Mass` | optional | Monoisotopic m/z; computed from `Formula` if absent |

The retention times here are illustrative — replace with values calibrated for
your column and gradient before using on real data.

## Bring your own list

Local working lists belong in `packages/analyzer/List/` (gitignored). The CSV
schema above is the only contract; column order doesn't matter.

## `marginal-topology-sample-sheet.csv`

A 14-sample fixture sheet for the **Marginal Topology** pipeline (sample-sheet-driven
untargeted feature discovery). The sheet exercises every supported column type:

- 2×2 factorial design (`factor:treatment` ∈ `{ctrl, drug}`, `factor:organ` ∈ `{liver, kidney}`)
- One ordered axis with three slots (`ordered:time` ∈ `{0, 1, 2}`)
- One sample with `run_order` for each row
- Two blank samples (one `solvent`, one `matrix`) that the Stage 2c blank filter consumes

Usage:

```bash
leaf marginal-topology results/ --sample-sheet examples/marginal-topology-sample-sheet.csv \
                                --files-relative-to /path/to/raw_files/
```

### Schema

| Column | Required | Description |
|---|---|---|
| `file` | yes | Absolute or relative RAW path |
| `factor:<name>` | optional | Factor column; substring after `factor:` is the factor name |
| `ordered:<name>` | optional | Ordered axis column (time, batch, injection_order, dose) |
| `run_order` | optional | Integer chronological injection position |
| `blank_role` | optional | `solvent` or `matrix`; empty = biological sample |
| `sample_id` | optional | Explicit integer ID (defaults to row index); **must be unique** |

See [docs/leaf/api/marginal-topology.md](../docs/leaf/api/marginal-topology.md)
for the full algorithm overview and parameter reference, and
[docs/leaf/api/cli.md](../docs/leaf/api/cli.md#leaf-marginal-topology-options)
for the CLI option surface.

## Installation

LEAF ships as a single platform-tagged `leaf` wheel per OS. On macOS and Linux
a `seed` sidecar wheel (Rust RAW reader) is also required. Download both from
a LEAF GitHub release and install with pip:

```bash
# macOS arm64
pip install leaf-<ver>-cp312-cp312-macosx_*_arm64.whl \
            seed-*-cp312-cp312-macosx_*_arm64.whl

# Linux x86_64
pip install leaf-<ver>-cp312-cp312-manylinux*_x86_64.whl \
            seed-*-cp312-cp312-manylinux*_x86_64.whl

# Windows x64 (seed not needed — the bundled .NET reader handles RAW files)
pip install leaf-<ver>-cp312-cp312-win_amd64.whl
```

See `.github/workflows/release.yml` for the canonical wheel naming and the
full list of release artifacts (desktop installers, `.mint` server bundle).
