# Changelog

LEAF (Targeted) — publication-build distribution.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
This changelog covers the paper-supplement distribution only; the internal
LEAF development history is not mirrored here.

---

## [0.5.0+paper.1] — 2026-05-15

First paper-supplement release. Companion to the manuscript
*&lt;paper title — to be filled in upon publication&gt;*.

### What this distribution ships

Three platform wheels:

- macOS (Apple Silicon, arm64)
- Linux x86_64
- Windows x64

Each wheel is self-contained:

- The targeted metabolomics pipeline (`leaf.analyzer`) ships as compiled
  extensions — Python source for analyzer-core modules (`align`, `extract`,
  `isotope`, `ms2`, `peaks`, `report`, `score`) is compiled to platform
  shared libraries via Cython.
- The SEED Rust LC-MS reader is statically linked into `leaf.core`; no
  separate sidecar wheel.
- On Windows, the Thermo `RawFileReader` .NET 8 assemblies and the
  `ParallelRawFileReader` helper ship alongside the wheel. macOS and
  Linux read RAW via the bundled SEED reader and do not require .NET.

### What this distribution does NOT ship

- Untargeted feature discovery (`leaf untargeted`, the `/untargeted`
  WebUI route, and the supporting routers / services).
- Marginal Topology untargeted pipeline (`leaf marginal-topology`).
- Visualization modules (PCA / heatmap / volcano / pathway network in
  the WebUI).
- Pathway-network gap-search service.

These remain part of the internal LEAF research codebase under
ongoing development.

### Installation

See [`README.md`](README.md) for `pip install` instructions per platform
and the `leaf targeted` quickstart. A starter metabolite list is at
[`examples/metabolite-list-primary-metabolism.csv`](examples/metabolite-list-primary-metabolism.csv).

### Citation

Citation block will be added upon publication.
