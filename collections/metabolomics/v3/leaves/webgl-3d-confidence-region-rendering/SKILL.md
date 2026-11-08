---
name: webgl-3d-confidence-region-rendering
description: Use when you have per-replicate QIIME principal coordinates files from
  jackknifed beta diversity resampling plus a metadata mapping file, and need to render
  per-sample confidence regions in an interactive 3D PCoA plot.
when_to_use_negative:
- You only need a static 2D scatter of PCoA points with no replicate/uncertainty data
  — plain ordination plotting suffices and the confidence-region aggregation step
  is unnecessary.
- The command line interface is required but the environment is Windows — EMPeror's
  CLI supports OS X (10.6 and higher) and Linux only.
- The visualization must run in a browser other than Chrome — the stated requirements
  specify Chrome for the graphical interface.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: Parses QIAME... parses jackknifed PCoA replicate files and metadata mapping
    files via its Python CLI and renders the per-sample confidence regions in a hardware-accelerated
    WebGL/HTML5 scene
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream source of the jackknifed beta diversity principal coordinates files
    and metadata mapping files consumed by EMPeror
- name: PyCogent
  role: Python dependency of EMPeror's command line interface for parsing the principal
    coordinates and mapping files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:74b67e77e6a021dcc84b616d33a0037251620d364ec2157d66a7089043f659ca
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
yes_rate: 0.7692307692307693
n_claims: 13
---

# webgl-3d-confidence-region-rendering

## Summary

Renders per-sample statistical confidence regions (e.g., spheroids/ellipsoids from jackknifed beta diversity PCoA replicates) as interactive WebGL/HTML5 3D scenes using EMPeror's command line interface, so clustering confidence can be assessed in a browser-shared visualization.

## When to use

Use when you have per-replicate QIIME principal coordinates files from jackknifed beta diversity resampling plus a metadata mapping file, and need to render per-sample confidence regions in an interactive 3D PCoA plot.

## When NOT to use

- You only need a static 2D scatter of PCoA points with no replicate/uncertainty data — plain ordination plotting suffices and the confidence-region aggregation step is unnecessary.
- The command line interface is required but the environment is Windows — EMPeror's CLI supports OS X (10.6 and higher) and Linux only.
- The visualization must run in a browser other than Chrome — the stated requirements specify Chrome for the graphical interface.

## Inputs

- Jackknifed beta diversity replicate principal coordinates files (QIIME PCoA format)
- QIIME metadata mapping file
- Python environment: Python 2.7, QIIME (python libraries only), NumPy, PyCogent, BIOM 1.1.0

## Outputs

- Self-contained WebGL/HTML5 web page rendering per-sample confidence regions (spheroids/ellipsoids) in a 3D PCoA scene
- Interactive UI controllers for visibility, transparency, and metadata-driven recoloring of jackknifed points

## How to apply

Parse the jackknifed replicate principal coordinates files and the metadata mapping file with the EMPeror Python command line interface, which accepts QIIME principal coordinates and mapping files and requires Python 2.7, QIIME python libraries, NumPy, PyCogent, and BIOM 1.1.0. Aggregate the per-replicate PCoA coordinates for each sample so that the dispersion of jackknife resamples defines a per-sample confidence region, while using sample metadata to drive coloring and visibility categories. Render each sample as a spheroid/ellipsoid confidence region in the hardware-accelerated WebGL/HTML5 scene, and expose UI controllers to toggle visibility, transparency, and metadata-based recoloring of the jackknifed points. Package the result as a small self-contained web page deliverable that is independent of the command line tool and can be embedded in emails or web pages without extra plugins.

## Related tools

- **EMPeror** (Parses QIAME... parses jackknifed PCoA replicate files and metadata mapping files via its Python CLI and renders the per-sample confidence regions in a hardware-accelerated WebGL/HTML5 scene) — http://emperor.colorado.edu
- **QIIME** (Upstream source of the jackknifed beta diversity principal coordinates files and metadata mapping files consumed by EMPeror)
- **PyCogent** (Python dependency of EMPeror's command line interface for parsing the principal coordinates and mapping files)

## Evaluation signals

- Open the generated page in Chrome and confirm jackknifed-resampled data appear as per-sample confidence regions (spheroids/ellipsoids), not single points.
- Verify visibility, color, and transparency of the jackknifed points can be modified interactively from metadata-driven UI controllers.
- Confirm the output file remains small and self-contained (EMPeror produced 26 MB vs 1.85 GB in KiNG for 5,740 samples and 120 metadata columns) and loads in seconds rather than minutes.
- Check the scene renders without extra browser plugins and is deliverable independently of the command line tool.

## Limitations

- Command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support stated.
- Requires Python 2.7, an end-of-life Python version, plus QIIME python libraries, NumPy, BIOM 1.1.0, and PyCogent.
- Graphical user interface requires Chrome, suggesting untested compatibility with other browsers.

## Evidence

- [abstract] display jackknifed-resampled data to assess statistical confidence in clustering: "display jackknifed-resampled data to assess statistical confidence in clustering"
- [abstract] EMPeror is a thoroughly tested and open-source software package with an interactive user interface and hardware-accelerated graphics, implemented with HTML5, WebGL, Javascript and Python, and tightly: "implemented with HTML5, WebGL, Javascript and Python"
- [discussion] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
- [discussion] OS X (10.6 and higher) and Linux only for the command line interface.: "OS X (10.6 and higher) and Linux only for the command line interface."
