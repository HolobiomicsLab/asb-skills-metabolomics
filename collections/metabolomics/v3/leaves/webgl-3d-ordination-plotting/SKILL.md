---
name: webgl-3d-ordination-plotting
description: Use when you have a QIIME principal coordinates file plus a sample metadata
  mapping file and need an interactive 3D PCoA plot that scales to thousands of samples
  (e.g. 5,740 samples, 120 metadata columns) and supports metadata-driven recoloring
  and visibility toggles.
when_to_use_negative:
- Input is a raw feature table or un-ordinated data — PCoA of a beta-diversity distance
  matrix must be computed first (e.g. via QIIME) before plotting
- Only generic static 3D plotting is needed and commercial/generic packages (SpotFire,
  MATLAB, R, Matplotlib) already suffice; this skill targets web-browser-enabled,
  metadata-driven exploratory visualization
- The target environment cannot run Chrome or the Python 2.7 / QIIME / NumPy / BIOM
  1.1.0 / PyCogent dependency stack, or Windows CLI support is required (EMPeror CLI
  is OS X 10.6+ and Linux only)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3300
tools:
- name: EMPeror
  role: 'Primary tool: Python backend generates the standalone HTML5/WebGL page with
    coordinates and embedded metadata columns for interactive 3D PCoA plotting'
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Python libraries used to parse the principal coordinates file and metadata
    mapping file; also the upstream source of the PCoA ordination being visualized
- name: PyCogent
  role: Dependency used with QIIME Python libraries for parsing coordinates and metadata
    during page generation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:b7f2fa6b723a8328c16131277fb146e294935daf83f250f0c50d58df5df0b588
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - PyCogent
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3300
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# webgl-3d-ordination-plotting

## Summary

Generates interactive, hardware-accelerated 3D principal coordinates (PCoA) visualizations in a standalone HTML5/WebGL page using EMPeror, so microbial community ordination plots can be recolored, shown, hidden, and shared by metadata without plugins.

## When to use

Use when you have a QIIME principal coordinates file plus a sample metadata mapping file and need an interactive 3D PCoA plot that scales to thousands of samples (e.g. 5,740 samples, 120 metadata columns) and supports metadata-driven recoloring and visibility toggles.

## When NOT to use

- Input is a raw feature table or un-ordinated data — PCoA of a beta-diversity distance matrix must be computed first (e.g. via QIIME) before plotting
- Only generic static 3D plotting is needed and commercial/generic packages (SpotFire, MATLAB, R, Matplotlib) already suffice; this skill targets web-browser-enabled, metadata-driven exploratory visualization
- The target environment cannot run Chrome or the Python 2.7 / QIIME / NumPy / BIOM 1.1.0 / PyCogent dependency stack, or Windows CLI support is required (EMPeror CLI is OS X 10.6+ and Linux only)

## Inputs

- QIIME principal coordinates file (PCoA output from beta-diversity distance-matrix ordination)
- QIIME sample metadata mapping file with per-sample metadata columns (including MIxS-family GSC fields)
- Optional: jackknifed beta-diversity coordinate data or procrustes-aligned coordinate comparisons for specialized plots

## Outputs

- Standalone HTML5/WebGL visualization page embedding coordinates and metadata, viewable in Chrome without extra plugins
- Interactive 3D PCoA plot with per-category point recoloring and show/hide controls, gradients/categorical coloring, biplots, and parallel-coordinates views

## How to apply

Parse the QIIME principal coordinates file and metadata mapping file with Python (QIIME python libraries and PyCogent) to obtain sample coordinates and per-sample metadata columns. Run EMPeror's Python backend to emit a standalone HTML5/WebGL page that embeds the coordinates and the full set of metadata columns, so no static per-category coloring files are generated. Build the JavaScript control panel that enumerates every metadata category (including MIxS-family GSC fields) and recolors the sample spheres as a function of the selected category. Keep the coloring logic separate from the visibility logic, adding per-category show/hide toggles so recoloring and point visibility vary independently. Open the generated page in Chrome and confirm the combined example dataset (5,740 samples, 120 metadata columns from the GigaScience Database / EMPeror FTP site) loads in seconds, not minutes.

## Related tools

- **EMPeror** (Primary tool: Python backend generates the standalone HTML5/WebGL page with coordinates and embedded metadata columns for interactive 3D PCoA plotting) — http://emperor.colorado.edu
- **QIIME** (Python libraries used to parse the principal coordinates file and metadata mapping file; also the upstream source of the PCoA ordination being visualized)
- **PyCogent** (Dependency used with QIIME Python libraries for parsing coordinates and metadata during page generation)

## Evaluation signals

- Generated page loads the 5,740-sample / 120-metadata-column example dataset in Chrome within seconds (not the many minutes KiNG requires)
- Output file size is ~26 MB for 5,740 samples and 120 metadata columns (~1.3% of KiNG's 1.85 GB discrete+gradient coloring output)
- Selecting any metadata category (including MIxS-family fields) in the control panel recolors the sample spheres correctly for that category
- Changing the color category does not alter which points are shown — coloring and visibility logic remain independent, and per-category show/hide toggles work
- Page runs in Chrome with no plugins required and can be embedded in emails or web pages

## Limitations

- Command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support stated
- Requires Python 2.7 (end-of-life), Chrome, QIIME python libraries, NumPy, BIOM 1.1.0, and PyCogent; the GUI requires the Chrome browser specifically
- Only a project home page (http://emperor.colorado.edu) is given — no code repository URL or archival DOI/accession IDs for the example datasets are stated

## Evidence

- [abstract] EMPeror reduces file size and load time: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] WebGL browser-based, no plugins: "implemented with HTML5, WebGL, Javascript and Python, and tightly"
- [abstract] Metadata-driven interactive control: "EMPeror includes a rich set of controllers to modify features as a function of the metadata."
- [discussion] Data availability: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] Dependencies: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
