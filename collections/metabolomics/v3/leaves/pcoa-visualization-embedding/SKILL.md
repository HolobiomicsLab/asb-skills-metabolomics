---
name: pcoa-visualization-embedding
description: Decision support for embedding interactive 3D principal coordinates plots
  of QIIME beta diversity PCoA output and metadata mapping files into a web page or
  email for sharing.
when_to_use_negative:
- Input is raw sequencing data or a feature/OTU table that has not yet been reduced
  to PCoA coordinates via beta diversity ordination — EMPeror consumes principal coordinates
  and mapping files, not upstream reads.
- The target environment does not support Chrome or WebGL — the paper lists Chrome
  as a hard requirement for the graphical user interface.
- You need a Windows-based command line workflow — the CLI is stated to run on OS
  X (10.6+) and Linux only.
- You want a static, publication-quality 2D figure — EMPeror's purpose is interactive
  3D exploration, not static image export.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3361
tools:
- name: EMPeror
  role: Generates the interactive 3D PCoA visualization from QIIME principal coordinates
    and metadata mapping files; provides the command line interface for batch generation
    and the embeddable web output
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream producer of the beta diversity distance matrix, PCoA ordination coordinates,
    and metadata mapping file that EMPeror consumes
- name: KiNG
  role: Legacy alternative for 3D PCoA plotting in QIIME; EMPeror replaces it with
    ~74x smaller output files and seconds-vs-minutes load times
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:f79534a1202cf733ee12f82cc6d031cbbb5a2c9781038a76e27007b7ad80365e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - KiNG
  - QIIME
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pcoa-visualization-embedding

## Summary

Generate and embed an interactive, hardware-accelerated 3D PCoA plot of microbial community beta diversity data as a reusable component in a host web page, using EMPeror's small-output web deliverable in place of heavyweight tools like KiNG.

## When to use

Decision support for embedding interactive 3D principal coordinates plots of QIIME beta diversity PCoA output and metadata mapping files into a web page or email for sharing.

## When NOT to use

- Input is raw sequencing data or a feature/OTU table that has not yet been reduced to PCoA coordinates via beta diversity ordination — EMPeror consumes principal coordinates and mapping files, not upstream reads.
- The target environment does not support Chrome or WebGL — the paper lists Chrome as a hard requirement for the graphical user interface.
- You need a Windows-based command line workflow — the CLI is stated to run on OS X (10.6+) and Linux only.
- You want a static, publication-quality 2D figure — EMPeror's purpose is interactive 3D exploration, not static image export.

## Inputs

- QIIME principal coordinates file (PCoA output from beta diversity distance matrix)
- QIIME metadata mapping file (sample metadata columns)
- Optional taxa or environmental sample annotations, biplot vectors, or procrustes coordinate files

## Outputs

- Interactive hardware-accelerated 3D PCoA visualization in a web page (HTML5/WebGL), embeddable as a reusable component
- Small standalone output file (~26 MB for 5,740 samples / 120 metadata columns) suitable for sharing via email or embedding in a web page

## How to apply

Start from QIIME's PCoA ordination of a beta diversity distance matrix and a metadata mapping file (mapping columns drive sample coloring, visibility, and shape). Run EMPeror's command line interface on these two files to generate a self-contained interactive 3D visualization built with HTML5, WebGL, Javascript and Python, which loads in Chrome without extra plugins. Embed the generated output directly into a host web page or email — EMPeror's output for 5,740 samples and 120 metadata columns is only ~26 MB (about 1.3% of the equivalent KiNG file), so it stays small enough to share. Use the built-in controllers to recolor or hide points as a function of metadata categories or gradients, display alternative principal coordinate axes, show taxa as well as environmental samples, overlay biplots or procrustes coordinate comparisons, or display jackknifed-resampled data to assess statistical confidence in clustering. Validate that the embedded page renders sample spheres in Chrome with sub-second-to-seconds load time rather than the many minutes KiNG requires at equivalent scale.

## Related tools

- **EMPeror** (Generates the interactive 3D PCoA visualization from QIIME principal coordinates and metadata mapping files; provides the command line interface for batch generation and the embeddable web output) — http://emperor.colorado.edu
- **QIIME** (Upstream producer of the beta diversity distance matrix, PCoA ordination coordinates, and metadata mapping file that EMPeror consumes)
- **KiNG** (Legacy alternative for 3D PCoA plotting in QIIME; EMPeror replaces it with ~74x smaller output files and seconds-vs-minutes load times)

## Examples

```
python emperor/scripts/make_emperor.py -i pcoa.txt -m mapping.txt -o embedded_emperor.html
```

## Evaluation signals

- The embedded page loads in Chrome and renders the 3D PCoA plot with sample spheres, without requiring any additional plugins.
- Output file size stays in the MB range (paper reports 26 MB for 5,740 samples and 120 metadata columns, ~1.3% of the KiNG equivalent) — order-of-magnitude reduction confirms correct generation.
- Interactive load time is measured in seconds, not the many minutes reported for KiNG at equivalent sample counts.
- Recoloring or toggling sample visibility in the embedded visualization correctly reflects categories/gradients from the supplied metadata mapping file.
- For jackknifed runs, the displayed ellipsoids reflect the resampled beta diversity coordinates, allowing visual assessment of clustering confidence.

## Limitations

- Command line interface is limited to OS X (10.6 and higher) and Linux; Windows is not supported.
- Graphical user interface requires the Chrome browser specifically, so cross-browser behavior is untested/unverified.
- Depends on Python 2.7 (end-of-life), NumPy, BIOM 1.1.0, PyCogent, and QIIME (python libraries only).
- The paper provides no version number for EMPeror itself and no GitHub/code-repository URL beyond the project home page.
- Supporting example files are referenced via the GigaScience Database and an EMPeror FTP site but no archival DOIs or accession IDs are given in the text.

## Evidence

- [abstract] EMPeror's small output file size, enables agile collaboration by allowing users to embed these visualizations via emails or web pages without the need for extra plugins: "ease of sharing, given EMPeror's small output file size, enables agile collaboration by allowing users to embed these visualizations via emails or web pages without the need for extra plugins"
- [abstract] 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] hardware-accelerated graphics, implemented with HTML5, WebGL, Javascript and Python: "EMPeror is a thoroughly tested and open-source software package with an interactive user interface and hardware-accelerated graphics, implemented with HTML5, WebGL, Javascript and Python"
- [abstract] EMPeror includes a rich set of controllers to modify features as a function of the metadata: "EMPeror includes a rich set of controllers to modify features as a function of the metadata. By being specifically tailored to the requirements of microbial ecologists, EMPeror thus increases the"
- [abstract] scale to thousands of points with minimal load times (seconds versus many minutes in KiNG): "scale to thousands of points with minimal load times (seconds versus many minutes in KiNG)"
- [discussion] The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25].: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] OS X (10.6 and higher) and Linux only for the command line interface.: "OS X (10.6 and higher) and Linux only for the command line interface."
- [discussion] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
- [abstract] display jackknifed-resampled data to assess statistical confidence in clustering: "display jackknifed-resampled data to assess statistical confidence in clustering"
- [abstract] ability to visualize gradients and categorical data, visualize different principal coordinates axes: "ability to visualize gradients and categorical data, visualize different principal coordinates axes"
