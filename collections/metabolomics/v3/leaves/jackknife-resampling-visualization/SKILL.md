---
name: jackknife-resampling-visualization
description: Use when QIIME principal coordinates files from replicate (jackknifed)
  beta diversity analyses plus a metadata mapping file must be rendered as per-sample
  confidence ellipsoids in an interactive 3D PCoA visualization.
when_to_use_negative:
- Input is a single (non-replicated) PCoA coordinate file with no jackknife resamples
  — there is no dispersion to define per-sample confidence regions, so plain point
  plotting suffices.
- The goal is upstream computation of jackknifed beta diversity distances rather than
  visualization; this skill only renders already-computed replicate PCoA coordinates
  via EMPeror.
- The analysis target is not microbial community beta diversity data (e.g. no QIIME-format
  principal coordinates or mapping file), since EMPeror is specifically tailored to
  microbial ecologists' QIIME outputs.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: Parses QIAME PCoA/mapping files via its Python CLI, aggregates replicate coordinates
    into per-sample confidence regions, and renders them in a hardware-accelerated
    WebGL/HTML5 3D scene.
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Produces the jackknifed beta diversity principal coordinates files and metadata
    mapping files consumed by EMPeror; only its python libraries are required.
- name: PyCogent
  role: Python dependency required by EMPeror's command line interface alongside QIIME
    libraries, NumPy, and BIOM 1.1.0.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:1ecaac45c9b341561a090dec3d3d543d1e3c91e23e05b264de30a3ed9b887820
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# jackknife-resampling-visualization

## Summary

Visualize jackknifed beta diversity resamples as per-sample confidence regions (spheroids/ellipsoids) in EMPeror's 3D WebGL PCoA plots, so researchers can assess the statistical confidence in clustering of microbial community samples.

## When to use

Use when QIIME principal coordinates files from replicate (jackknifed) beta diversity analyses plus a metadata mapping file must be rendered as per-sample confidence ellipsoids in an interactive 3D PCoA visualization.

## When NOT to use

- Input is a single (non-replicated) PCoA coordinate file with no jackknife resamples — there is no dispersion to define per-sample confidence regions, so plain point plotting suffices.
- The goal is upstream computation of jackknifed beta diversity distances rather than visualization; this skill only renders already-computed replicate PCoA coordinates via EMPeror.
- The analysis target is not microbial community beta diversity data (e.g. no QIIME-format principal coordinates or mapping file), since EMPeror is specifically tailored to microbial ecologists' QIIME outputs.

## Inputs

- Jackknifed beta diversity replicate principal coordinates (PCoA) files in QIIME format
- QIIME metadata mapping file

## Outputs

- Self-contained WebGL/HTML5 web page displaying each sample as a confidence region (spheroid/ellipsoid) in 3D PCoA space
- Interactive UI controllers for visibility, transparency, and metadata-driven recoloring of jackknifed points

## How to apply

Parse the jackknifed beta diversity replicate principal coordinates files and the metadata mapping file with EMPeror's Python command line interface, which accepts QIIME principal coordinates files and metadata mapping files (Python 2.7, QIIME python libraries, NumPy, PyCogent, BIOM 1.1.0, Chrome). Aggregate the per-replicate PCoA coordinates for each sample so the dispersion of the jackknife resamples defines a per-sample confidence region, while sample metadata drives coloring and visibility categories. Render each sample as a spheroid/ellipsoid in the hardware-accelerated WebGL/HTML5 scene, and expose UI controllers that toggle visibility, transparency, and metadata-driven recoloring of the jackknifed points. Package the result as a self-contained web page deliverable independent of the command line tool, exploiting EMPeror's small output size (e.g. 26 MB vs 1.85 GB for 5,740 samples in KiNG) for sharing via email or web pages.

## Related tools

- **EMPeror** (Parses QIAME PCoA/mapping files via its Python CLI, aggregates replicate coordinates into per-sample confidence regions, and renders them in a hardware-accelerated WebGL/HTML5 3D scene.) — http://emperor.colorado.edu
- **QIIME** (Produces the jackknifed beta diversity principal coordinates files and metadata mapping files consumed by EMPeror; only its python libraries are required.)
- **PyCogent** (Python dependency required by EMPeror's command line interface alongside QIIME libraries, NumPy, and BIOM 1.1.0.)

## Evaluation signals

- Jackknifed-resampled data are displayed as per-sample confidence regions (spheroids/ellipsoids) rather than discrete points.
- Visibility, color, and transparency of the jackknifed points can be modified interactively from the metadata-driven controllers.
- The generated visualization opens in Chrome without extra plugins, as a self-contained web page independent of the command line tool.
- The tool accepted genuine QIIME principal coordinates files and metadata mapping files as inputs.
- Output file size remains small (EMPeror-scale, e.g. tens of MB) rather than the KiNG-scale multi-GB files, confirming correct EMPeror rendering.

## Limitations

- EMPeror's command line interface runs only on OS X (10.6 and higher) and Linux; no Windows support is stated.
- Requires Python 2.7, an end-of-life Python version, plus Chrome specifically for the GUI, suggesting untested compatibility with other browsers.
- No version number for Emperor itself, no code repository URL (only the project home page), and no archival DOI or accession IDs for the supporting example files and datasets are provided.

## Evidence

- [abstract] Jackknifed plots assess clustering confidence: "display jackknifed-resampled data to assess statistical confidence in clustering"
- [abstract] Auxiliary data display includes jackknifed beta diversity: "ability to visualize gradients and categorical data, visualize different principal coordinates axes, present the data in the form of parallel coordinates, show taxa as well as environmental samples,"
- [abstract] PCoA for dimensionality reduction of distance matrices: "Ordination methods, such as Principal Coordinates Analysis (PCoA) [6] are useful for dimensionality reduction and widely used in different fields to conceptualize distance matrices"
- [discussion] Software requirements for the CLI and GUI: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
- [discussion] CLI platform limitation: "OS X (10.6 and higher) and Linux only for the command line interface."
