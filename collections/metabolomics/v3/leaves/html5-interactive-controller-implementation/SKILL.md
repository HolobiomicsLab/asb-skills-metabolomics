---
name: html5-interactive-controller-implementation
description: Use when building a web-browser 3D visualization of PCoA-embedded microbial
  community samples where visual features (sphere size, transparency, color, visibility)
  must be driven interactively by metadata categories or gradient values.
when_to_use_negative:
- Input is not QIIME-generated ordination/mapping data, e.g. raw reads or feature
  tables that still need PCoA dimensionality reduction of a beta diversity distance
  matrix first
- The goal is a static figure or a generic 3D plot in MATLAB, R, Matplotlib or SpotFire
  rather than an interactive, metadata-controllable browser visualization
- The deliverable must run outside Chrome or without WebGL-capable hardware acceleration,
  e.g. as a command-line-only tool on Windows
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMPeror
  role: Reference implementation whose interactive per-category controllers for sphere
    size, transparency and color this skill re-implements; provides the open-source,
    web-browser enabled package with HTML5/WebGL/JavaScript/Python architecture
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream source of the principal coordinates file and metadata mapping file
    that the Python parser consumes before rendering
- name: PyCogent
  role: Python library requirement for the EMPeror software environment that hosts
    the controller implementation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:39ff057c1721a4039ff8c7fe479bd8366b5fb7ccf14b69129654d2c395aaa5e0
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
yes_rate: 1.0
n_claims: 13
---

# html5-interactive-controller-implementation

## Summary

Implements EMPeror-style interactive visualization controllers in which sphere size and transparency are modified as a function of sample metadata on a per-category basis, using hardware-accelerated HTML5/WebGL graphics backed by Python-parsed QIIME data.

## When to use

Use when building a web-browser 3D visualization of PCoA-embedded microbial community samples where visual features (sphere size, transparency, color, visibility) must be driven interactively by metadata categories or gradient values.

## When NOT to use

- Input is not QIIME-generated ordination/mapping data, e.g. raw reads or feature tables that still need PCoA dimensionality reduction of a beta diversity distance matrix first
- The goal is a static figure or a generic 3D plot in MATLAB, R, Matplotlib or SpotFire rather than an interactive, metadata-controllable browser visualization
- The deliverable must run outside Chrome or without WebGL-capable hardware acceleration, e.g. as a command-line-only tool on Windows

## Inputs

- QIIME principal coordinates file (PCoA coordinates for samples)
- QIIME metadata mapping file (per-sample category memberships, e.g. body site, age)

## Outputs

- Self-contained HTML5/WebGL web page rendering samples as spheres with metadata-driven per-category size and transparency
- Interactive controller panel for selecting metadata categories and adjusting size/transparency on a per-category basis

## How to apply

Parse the QIIME principal coordinates file and metadata mapping file in Python to extract per-sample 3D coordinates and category memberships, then render each sample as a sphere in an HTML5 canvas using WebGL so that sphere size and transparency are exposed as per-vertex attributes rather than baked-in constants. Implement a controller that maps each selected metadata category (e.g. body site or age) to a size and transparency value applied to all spheres belonging to that category, mirroring EMPeror's rich set of metadata-driven controllers. Package the renderer and controller as a self-contained web page deliverable independent of the command line tool, since EMPeror's interface is web-browser enabled and requires Chrome. The rationale is that hardware-accelerated graphics let the visualization scale to thousands of points with minimal load times, whereas naive approaches such as KiNG balloon to 1.85 GB for 5,740 samples across 120 metadata columns versus EMPeror's 26 MB.

## Related tools

- **EMPeror** (Reference implementation whose interactive per-category controllers for sphere size, transparency and color this skill re-implements; provides the open-source, web-browser enabled package with HTML5/WebGL/JavaScript/Python architecture) — http://emperor.colorado.edu
- **QIIME** (Upstream source of the principal coordinates file and metadata mapping file that the Python parser consumes before rendering)
- **PyCogent** (Python library requirement for the EMPeror software environment that hosts the controller implementation)

## Examples

```
make_3d_plots.py -i pcoa.txt -m mapping.txt -t emperor --biplot --n_components 3
```

## Evaluation signals

- The self-contained web page exists and loads headlessly without a command-line tool dependency
- Selecting a different metadata category in the controller visibly changes sphere size and/or transparency for exactly the spheres in that category (expert review in Chrome)
- Output file size stays near the small-output regime (e.g. ~26 MB for 5,740 samples and 120 metadata columns, ~1.3% of KiNG's 1.85 GB), confirming per-attribute rather than per-sample-expansion encoding
- Page load time for thousands of points remains on the order of seconds rather than many minutes
- Spheres reflect per-sample 3D PCoA coordinates parsed from the QIIME principal coordinates file and respect per-sample category assignments from the mapping file

## Limitations

- Graphical user interface requires Chrome; compatibility with other browsers is not established
- Command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support stated
- Requires Python 2.7, an end-of-life Python version, plus QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent
- No version number, code repository URL, or archival DOI/accession IDs are provided for the supporting example files and data sets

## Evidence

- [abstract] Rich controllers for metadata-driven visual properties: "EMPeror includes a rich set of controllers to modify features as a function of the metadata."
- [abstract] Hardware-accelerated HTML5/WebGL/Python stack: "EMPeror is a thoroughly tested and open-source software package with an interactive user interface and hardware-accelerated graphics, implemented with HTML5, WebGL, Javascript and Python"
- [abstract] Per-category size and transparency of spheres: [evidence span withheld — anchor status: not_found]
- [discussion] Example data available from GigaScience Database and FTP: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] Browser and Python requirements for the GUI: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
- [abstract] Dramatically smaller output files than KiNG: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] Seconds-scale load for thousands of points: "scale to thousands of points with minimal load times (seconds versus many minutes in KiNG)"
