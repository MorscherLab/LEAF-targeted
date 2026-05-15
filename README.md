<p align="center">
  <img src="assets/leaf-icon.png" alt="LEAF" width="160" />
</p>

# LEAF (Targeted)

> Targeted LC-MS metabolomics pipeline — publication build of LEAF
> (LC-MS Extensible Analysis Framework).

This distribution accompanies the manuscript *&lt;paper title — fill in
before release&gt;* and contains the targeted metabolomics workflow used to
generate the reported results: extract metabolites by reference list,
detect peaks, score peak quality, export results.

Untargeted feature discovery, interactive peak picking parameter tuning,
and visualization modules (PCA / heatmap / volcano / pathway network) are
NOT included — they remain part of the full LEAF research codebase under
private development.

---

## Citation

If you use this software, please cite:

> &lt;citation block — to be filled in upon publication&gt;

---

## Install

Download the wheel matching your platform from this repository's
[Releases](https://github.com/MorscherLab/LEAF-targeted/releases) page, then:

```bash
pip install leaf_targeted-<version>-<platform>.whl
```

| Platform | Wheel suffix |
|---|---|
| Linux x86_64 | `manylinux_2_28_x86_64.whl` |
| macOS arm64 (Apple Silicon) | `macosx_*_arm64.whl` |
| Windows x64 | `win_amd64.whl` |

Verify installation:

```bash
leaf --version
```

### System requirements

- Python **3.12 or 3.13**
- macOS 12+ (arm64) / Ubuntu 22.04+ (x86_64) / Windows 10+ (x64)
- 4 GB RAM minimum, 8 GB recommended for batches of 50+ files
- For Windows: the bundled .NET 8 runtime is included; no separate
  install needed

---

## Quickstart

### 1. Prepare your metabolite list

A CSV with four required columns:

```csv
Metabolite,Formula,RetentionTime,Adduct
glucose,C6H12O6,5.2,M-H
lactate,C3H6O3,3.1,M-H
citrate,C6H8O7,8.5,M-H
```

| Column | Meaning |
|---|---|
| `Metabolite` | Display name (any string) |
| `Formula` | Neutral chemical formula (e.g. `C6H12O6`) |
| `RetentionTime` | Expected RT in minutes |
| `Adduct` | Ion form (e.g. `M-H`, `M+H`, `M+Na`) |

A starter list (primary-metabolism set, ~30 compounds) is at
[`examples/metabolite-list-primary-metabolism.csv`](examples/metabolite-list-primary-metabolism.csv).

### 2. Run the targeted pipeline

```bash
# Single RAW file
leaf targeted SAMPLE.raw metabolite-list.csv ./output/

# Directory of RAW files
leaf targeted ./raw_files/ metabolite-list.csv ./output/

# With explicit parameters
leaf targeted ./raw_files/ metabolite-list.csv ./output/ \
    --polarity NEG \
    --tolerance 5 \
    --method v4 \
    --rt-window 0.3 \
    --intensity-threshold 1e5
```

### 3. Inspect results

```bash
leaf inspect ./output/SAMPLE.msd
```

The pipeline writes one `.msd` file per RAW input, plus optional
aggregated summary tables in CSV / Parquet form. See [Output format](#output-format)
below.

---

## Web UI

For interactive review of targeted results:

```bash
leaf webui run
```

This starts a local server on `http://127.0.0.1:18008` and opens it in
your default browser. Upload `.msd` files via drag-and-drop, browse
extracted ion chromatograms (EICs), and inspect peak picks per sample.

Stop with `Ctrl+C` (foreground mode). For long-running review sessions
on a workstation, see `leaf webui start --help` for daemon mode.

---

## CLI reference

| Command | Purpose |
|---|---|
| `leaf targeted <raw> <list> <out>` | The core pipeline: extract + peak-pick + score |
| `leaf inspect <archive>` | Summarize a `.msd` result archive |
| `leaf validate <list>` | Validate a metabolite list CSV |
| `leaf init` | Create a starter project layout in CWD |
| `leaf doctor` | Diagnose your installation (Python, .NET, native libs) |
| `leaf convert <raw> <out>` | Convert Thermo RAW → mzML |
| `leaf watch run <dir>` | Watch a folder; auto-run targeted on new RAWs |
| `leaf webui run` | Browser UI for review |

Run any subcommand with `--help` for full option list.

### Common options

| Flag | Default | Meaning |
|---|---|---|
| `--polarity {NEG,POS}` | `NEG` | Ionisation mode |
| `--tolerance INT` | `5` | m/z tolerance in ppm |
| `--method {v1,v2,v4}` | `v4` | Peak detection algorithm |
| `--rt-window FLOAT` | `0.3` | RT search window in minutes |
| `--intensity-threshold FLOAT` | `1e5` | Minimum peak intensity |
| `--align-rt` | off | Enable RT alignment across samples |
| `--report` | off | Render PDF report with EIC plots |
| `--backend {auto,dotnet,rust}` | `auto` | RAW reader backend |

---

## Output format

### `.msd` archive

A zstd-compressed Arrow/Parquet bundle containing, per RAW input:

- **`tensor`** — sparse intensity matrix `[n_rt × n_metabolites]`
- **`peaks_dict`** — detected peak RT indices for each (sample, metabolite)
- **`matched_peak`** — selected peak per (sample, metabolite)
- **`compound_scores`** — quality verdict per metabolite (`good` / `warning` / `poor`)
- **`metabolites_index`** — metadata DataFrame (Formula, RT, Adduct, calculated mass)

Load programmatically:

```python
from leaf.analyzer.data import Samples
samples = Samples.load("output/SAMPLE.msd")
print(samples.tensor.shape)
print(samples.compound_scores["glucose"].verdict)
```

### Aggregated tables

When run over multiple RAWs, an aggregated set of CSV / Parquet exports
lands alongside the per-sample `.msd` files:

| File | Content |
|---|---|
| `quantification.csv` | Wide table: metabolites × samples, peak areas |
| `quality.csv` | Verdict + sub-scores per (sample, metabolite) |
| `peak_picks.parquet` | Detailed peak metrics |

---

## Reproducing paper results

The exact command line + metabolite list used for each figure in the
manuscript is documented in
[`examples/paper-runs/`](examples/paper-runs/), with one subdirectory per
figure. Each contains:

- `run.sh` — the literal command to reproduce the analysis
- `metabolite-list.csv` — the compound list as published
- `expected/` — golden output sketches for spot-checking

Note: the input RAW files themselves are deposited on Zenodo (DOI link
in the manuscript) due to size.

---

## Troubleshooting

**`leaf targeted` says "No RAW reader backend available"**
Run `leaf doctor`. On macOS arm64 the Rust backend is bundled. On Linux /
Windows the .NET backend ships with the wheel; if `pythonnet` failed
to install, reinstall the wheel after `pip install pythonnet>=3.0.5`.

**`pip install` fails with "no matching distribution"**
You're on an unsupported platform/Python. The wheels target Python 3.12
or 3.13 on Linux x86_64, macOS arm64, and Windows x64. Other
combinations are not supported in this release.

**Peak quality verdicts are mostly "poor"**
Check `leaf validate metabolite-list.csv` — wrong adduct sign for the
polarity is the most common cause. Then try `--tolerance 10` if your
instrument runs warmer than the default 5 ppm.

**Webui won't start on Windows**
Windows Defender / corporate firewalls sometimes block uvicorn's
loopback bind. Run `leaf webui run --host 127.0.0.1 --port 8765` to
try an alternate port.

---

## License

This software is distributed under a **strict publication license** —
academic / non-commercial research use only, no redistribution, no
modification, no reverse engineering of compiled components. See
[`LICENSE`](LICENSE) for the full terms.

Commercial licensing inquiries: see `LICENSE` section 1 and the contact
at the bottom of that file.

---

## Source code

This is a compiled distribution. The Python source for analyzer-core
modules (`align`, `extract`, `isotope`, `ms2`, `peaks`, `report`,
`score`) is compiled to platform-specific shared libraries via Cython
and not included in source form.

The CLI surface, public API, and integration glue (FastAPI routers,
reader bridges) are shipped as readable `.py` for inspection and
extension within the License terms.

Untargeted discovery and visualization modules remain in private
development and are not part of this release.

---

## Acknowledgements

LEAF builds on Thermo Fisher's MS File Reader, the `seed` Rust LC-MS
reader (from `oxion-core`), and the broader scientific Python ecosystem
(NumPy, SciPy, pandas, PyArrow, FastAPI, Vue 3). Full dependency
manifest is in the wheel's metadata.
