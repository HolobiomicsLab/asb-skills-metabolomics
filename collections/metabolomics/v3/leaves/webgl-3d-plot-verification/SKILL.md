---
name: webgl-3d-plot-verification
description: Use when a QIIME principal coordinates file plus metadata mapping file
  must become an embedded, interactive WebGL 3D PCoA plot whose sample spheres can
  be verified to render in Chrome.
when_to_use_negative:
- Input has not yet been reduced to principal coordinates — raw reads or a QIIME feature
  table must first go through beta diversity and PCoA before EMPeror can visualize
  it.
- The target viewer must use a browser other than Chrome, or generation must run on
  Windows — the paper's GUI requires Chrome and the CLI is OS X (10.6+)/Linux only.
- A static, non-interactive figure suffices and no metadata-driven exploration is
  needed; generic packages such as MATLAB, R or Matplotlib can be used instead.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: EMPeror
  role: Produces the embeddable WebGL 3D PCoA visualization via its command line interface;
    the generated output is what gets embedded and verified
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Source of the principal coordinates file and metadata mapping file consumed
    by EMPeror; QIIME Python libraries are a stated runtime requirement
- name: Chrome
  role: Required browser used to open the embedded page and verify the 3D plot renders
    the sample spheres
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:773683f5cc259f62a3d00fab979a06331117a01197fb8c7bbe38203a1afd1626
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Chrome
  - EMPeror
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# webgl-3d-plot-verification

## Summary

Generates an interactive, hardware-accelerated WebGL 3D PCoA visualization with EMPeror from a QIIME principal coordinates file and metadata mapping file, embeds it in a host web page, and verifies the sample spheres render in Chrome without extra plugins.

## When to use

Use when a QIIME principal coordinates file plus metadata mapping file must become an embedded, interactive WebGL 3D PCoA plot whose sample spheres can be verified to render in Chrome.

## When NOT to use

- Input has not yet been reduced to principal coordinates — raw reads or a QIIME feature table must first go through beta diversity and PCoA before EMPeror can visualize it.
- The target viewer must use a browser other than Chrome, or generation must run on Windows — the paper's GUI requires Chrome and the CLI is OS X (10.6+)/Linux only.
- A static, non-interactive figure suffices and no metadata-driven exploration is needed; generic packages such as MATLAB, R or Matplotlib can be used instead.

## Inputs

- QIIME principal coordinates file (PCoA ordinates of a beta diversity distance matrix)
- QIIME metadata mapping file (sample metadata, e.g. 120 columns, driving coloring/visibility)
- Host web page / HTML container into which the EMPeror output is embedded as a reusable component

## Outputs

- Self-contained interactive WebGL 3D PCoA visualization (HTML5/JavaScript) rendering samples as spheres
- Embedded page (e.g. embedded_emperor.html) that loads in Chrome with no extra plugins
- MB-scale deliverable shareable by email or web page (~26 MB for 5,740 samples with 120 metadata columns)

## How to apply

Obtain a QIIME principal coordinates file and its paired metadata mapping file (the paper deposits example files in the GigaScience Database and on the EMPeror FTP site), then run the EMPeror command line interface on both to generate an interactive, hardware-accelerated HTML5/WebGL/JavaScript 3D visualization. Embed the generated output into a host web page as a reusable component (Evident-style); because rendering is WebGL-based, no extra plugins are required. Open the embedded page in Chrome — the only browser named in EMPeror's requirements — and confirm the 3D PCoA plot draws the sample spheres and that metadata controllers recolor points or toggle visibility for gradient and categorical columns. Validate scale and weight: the deliverable should remain MB-scale (26 MB for 5,740 samples with 120 metadata columns, i.e. 1.3% of KiNG's 1.85 GB) and load in seconds rather than the many minutes KiNG needed. If generation fails, check the stated environment: OS X (10.6+) or Linux for the CLI, with Python 2.7, QIIME Python libraries, NumPy, BIOM 1.1.0 and PyCogent installed.

## Related tools

- **EMPeror** (Produces the embeddable WebGL 3D PCoA visualization via its command line interface; the generated output is what gets embedded and verified) — http://emperor.colorado.edu
- **QIIME** (Source of the principal coordinates file and metadata mapping file consumed by EMPeror; QIIME Python libraries are a stated runtime requirement)
- **Chrome** (Required browser used to open the embedded page and verify the 3D plot renders the sample spheres)

## Evaluation signals

- The embedded page (e.g. embedded_emperor.html) loads in Chrome and the WebGL 3D PCoA plot displays the sample spheres.
- Output size stays MB-scale — expect roughly 26 MB for 5,740 samples with 120 metadata columns (1.3% of KiNG's 1.85 GB).
- Load time is seconds, not minutes, at thousand-point scale, matching the paper's EMPeror-versus-KiNG benchmark.
- No extra browser plugins are required — rendering relies on HTML5, WebGL and JavaScript.
- Metadata controllers work as expected: gradient and categorical coloring, principal coordinates axis switching, and point visibility toggles respond to mapping-file columns.

## Limitations

- Command line interface runs on OS X (10.6 and higher) and Linux only; no Windows support is stated.
- The GUI requires Chrome, suggesting compatibility with other browsers is untested.
- Depends on Python 2.7 (an end-of-life version), QIIME Python libraries, NumPy, BIOM 1.1.0 and PyCogent.
- No software version number, code repository URL, or archival DOI/accession IDs are given in the paper for Emperor and its example files (only the GigaScience Database and EMPeror FTP site).

## Evidence

- [abstract] EMPeror is a browser-based 3D visualization tool with a CLI: "an open source and web browser enabled tool with a versatile command line interface that allows researchers to perform rapid exploratory investigations of 3D visualizations"
- [abstract] Rendering stack is HTML5/WebGL/JavaScript/Python with hardware acceleration: "hardware-accelerated graphics, implemented with HTML5, WebGL, Javascript and Python"
- [abstract] MB-scale output benchmark: 26 MB vs KiNG's 1.85 GB (1.3%): "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] Embeddable without plugins for sharing via email or web pages: "allowing users to embed these visualizations via emails or web pages without the need for extra plugins"
- [discussion] Stated environment requirements constrain generation and verification: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
- [discussion] CLI OS support is limited, a caveat when generating the embedded plot: "OS X (10.6 and higher) and Linux only for the command line interface."
