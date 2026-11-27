<p align="center">
  <img src="assets/leaf-icon.png" alt="LEAF" width="160" />
</p>

# LEAF (Targeted)

> Targeted LC-MS metabolomics pipeline — publication build of LEAF
> (LC-MS Extensible Analysis Framework).

This distribution accompanies the manuscript *High-Throughput Mass-Spectrometry-Based Quantification and Functional
Assessment of Polyamine Metabolism* and contains the targeted metabolomics
workflow used to generate the reported results: extract metabolites by reference list,
detect peaks, score peak quality, export results.

Untargeted feature discovery, interactive peak picking parameter tuning,
and visualization modules (PCA / heatmap / volcano / pathway network) are
NOT included — they remain part of the full LEAF research codebase under
private development.

---

## Citation

If you use this software, please cite:

> *High-Throughput Mass-Spectrometry-Based Quantification and Functional
> Assessment of Polyamine Metabolism.* (Journal reference follows on
> publication.)

---

## Install

Download the wheel matching your platform from this repository's
[Releases](https://github.com/MorscherLab/LEAF-targeted/releases) page, then:

```bash
pip install leaf_targeted-0.5.0+paper.2-<platform>.whl
```

| Platform | Wheel suffix |
|---|---|
| Linux x86_64 | `manylinux_*_x86_64.whl` |
| macOS arm64 (Apple Silicon) | `macosx_*_arm64.whl` |
| Windows x64 | `win_amd64.whl` |

Verify installation:

```bash
pip show leaf-targeted   # Version: 0.5.0+paper.2
leaf --help
```

### System requirements

- Python **3.12** (the wheels are built for CPython 3.12 only)
- macOS 12+ (arm64) / Ubuntu 22.04+ (x86_64) / Windows 10+ (x64)
- 4 GB RAM minimum, 8 GB recommended for batches of 50+ files
- No non-standard hardware (no GPU, no instrument connection)
- For Windows: the bundled .NET 8 runtime is included; no separate
  install needed

### Tested versions

| OS | Python | How |
|---|---|---|
| macOS 27.2, Apple M5 Pro (arm64) | 3.12.13 | Build, smoke tests and demo |
| Ubuntu 22.04 (x86_64, container) | 3.12 | Build and smoke tests |
| Windows 11 Pro 24H2 (x64) | 3.12 | Build and smoke tests |

The smoke tests install the wheel into a clean virtual environment and run
`leaf targeted` end to end on a synthetic mzML file.

### Install time

About 20 s with `pip` and 5 s with `uv pip`, on a desktop computer with
a broadband connection (measured on macOS, Apple M5 Pro, empty package
cache). Most of the time is the download of NumPy, SciPy, pandas and
PyArrow.

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
leaf inspect ./output/<stem>_extracted.msd
```

The pipeline writes one `<stem>_extracted.msd` archive for the whole run
and one `<stem>_result.csv` quantification table, where `<stem>` is the
name of the input file or folder. See [Output format](#output-format)
below.

---

## Demo

The demo runs the manuscript's command (Table 5) on one RAW file of the
paper's dataset.

1. Download one RAW file from MassIVE dataset
   [MSV000101858](https://massive.ucsd.edu/ProteoSAFe/dataset.jsp?accession=MSV000101858)
   (doi:10.25345/C5Z31P31Q), for example
   `exp159_251023_LCK_PA_cells_POS_C1_A_108.raw` (126 MB), into a folder
   `data/`.
2. Download [`examples/polyamine-list.csv`](examples/polyamine-list.csv)
   (107 AQC-derivatized polyamines, amino acids and related compounds).
3. Run:

   ```bash
   leaf targeted data/ polyamine-list.csv results/ \
       --polarity POS --tolerance 5 --rt-window 0.1 --method v4 \
       --intensity-threshold 1e5 --report --save-extract
   ```

**Expected output** in `results/`:

| File | Content |
|---|---|
| `data_extracted.msd` | EICs, peaks and scores; open it in `leaf webui run` |
| `data_result.csv` | 321 rows: one row per compound and isotopologue, peak area per sample |
| `report.pdf` | Summary pages and one EIC panel per compound |

The terminal summary reports `Compounds scored: 107 (good 26 · warning
11 · poor 0 · insufficient 70)`. With one sample most compounds are
`insufficient`, because the verdict needs detections across samples.

**Expected run time:** about 10 s for one file (8.4 s wall time, 0.4 GB
peak memory, macOS, Apple M5 Pro). Run time increases linearly with the
number of files.

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

A zstd-compressed Arrow/Parquet bundle containing, for all inputs of the run:

- **`tensor`** — sparse intensity matrix `[n_rt × n_metabolites]`
- **`peaks_dict`** — detected peak RT indices for each (sample, metabolite)
- **`matched_peak`** — selected peak per (sample, metabolite)
- **`compound_scores`** — quality verdict per metabolite (`good` / `warning` / `poor`)
- **`metabolites_index`** — metadata DataFrame (Formula, RT, Adduct, calculated mass)

Load programmatically:

```python
from leaf.analyzer.data import Samples
samples = Samples.load("results/data_extracted.msd")
print(len(samples.tensor), samples.tensor[0].shape)
print(samples.compound_scores["glucose"].verdict)
```

### Tables and report

| File | Content |
|---|---|
| `<stem>_result.csv` | Peak area per compound, isotopologue and sample |
| `report.pdf` | EIC panels and verdicts (only with `--report`) |

---

## Reproducing paper results

The manuscript's automatic processing uses the command in the
[Demo](#demo) section (manuscript Table 5) on all RAW files of an
experiment:

1. Download the RAW files of the experiment from MassIVE
   [MSV000101858](https://massive.ucsd.edu/ProteoSAFe/dataset.jsp?accession=MSV000101858)
   into one folder.
2. Run the Demo command on that folder with
   [`examples/polyamine-list.csv`](examples/polyamine-list.csv).
3. Open `<folder>_extracted.msd` in `leaf webui run` for the optional
   peak correction described in the manuscript.

The version used in the manuscript is archived on Zenodo
(doi:10.5281/zenodo.19954098).

---

## Troubleshooting

**`leaf targeted` says "No RAW reader backend available"**
Run `leaf doctor`. On macOS arm64 the Rust backend is bundled. On Linux /
Windows the .NET backend ships with the wheel; if `pythonnet` failed
to install, reinstall the wheel after `pip install pythonnet>=3.0.5`.

**`pip install` fails with "no matching distribution"**
You're on an unsupported platform/Python. The wheels target Python 3.12
on Linux x86_64, macOS arm64, and Windows x64. Other
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

LEAF (Targeted) is free software under the **GNU Affero General Public
License v3.0 only** (AGPL-3.0-only, OSI-approved), with additional terms
under AGPL-3.0 section 7: keep the attribution notices, mark modified
versions, no use of the authors' names for endorsement, and no rights to
the LEAF name or logo. See [`LICENSE`](LICENSE) for the license text and
[`NOTICE.md`](NOTICE.md) for the copyright notice and the additional
terms.

For other licensing terms, contact raphael.morscher@kispi.uzh.ch.

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
