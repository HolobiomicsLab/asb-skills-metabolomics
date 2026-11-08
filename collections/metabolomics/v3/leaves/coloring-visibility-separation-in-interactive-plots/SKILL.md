---
name: coloring-visibility-separation-in-interactive-plots
description: Use when building or running an interactive 3D PCoA plot of microbial
  community samples where users must switch the color category without changing which
  points are displayed, or toggle per-category show/hide states.
when_to_use_negative:
- Input is a static figure or pre-rendered plot file rather than a principal coordinates
  file plus mapping file — no metadata-driven recoloring is possible.
- The visualization target is 2D or a non-browser format; the interactive HTML5/WebGL
  controls described here are specific to EMPeror's 3D plot workflow.
- The user only needs a single fixed color scheme baked into the final image, with
  no interactive metadata switching — a static plotting library (e.g. Matplotlib)
  suffices.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3309
tools:
- name: EMPeror
  role: Tool that implements the decoupled coloring/visibility control panel; its
    Python backend generates the standalone HTML5/WebGL page from the PCoA and mapping
    files
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Provides the Python libraries used to parse the principal coordinates file
    and the metadata mapping file
- name: PyCogent
  role: Supporting Python dependency listed with QIIME for parsing the input files
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:e9064ea2b1c4244ee358c9e5ae4fe4e88c230e84966765115c1e7c36391180a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - PyCogent
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3309
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# coloring-visibility-separation-in-interactive-plots

## Summary

Decouple point coloring from point visibility in interactive 3D principal coordinates visualizations, so that sample spheres can be recolored and shown/hidden independently as a function of arbitrary metadata categories (including MIxS-family standards).

## When to use

Use when building or running an interactive 3D PCoA plot of microbial community samples where users must switch the color category without changing which points are displayed, or toggle per-category show/hide states.

## When NOT to use

- Input is a static figure or pre-rendered plot file rather than a principal coordinates file plus mapping file — no metadata-driven recoloring is possible.
- The visualization target is 2D or a non-browser format; the interactive HTML5/WebGL controls described here are specific to EMPeror's 3D plot workflow.
- The user only needs a single fixed color scheme baked into the final image, with no interactive metadata switching — a static plotting library (e.g. Matplotlib) suffices.

## Inputs

- QIIME principal coordinates file (per-sample PCoA coordinates)
- QIIME metadata mapping file with per-sample metadata columns (including MIxS-family standards fields)
- Optional gradient vs. discrete coloring scheme specification per metadata category

## Outputs

- Standalone HTML5/WebGL visualization page with an interactive control panel for dynamic metadata-driven coloring and per-category show/hide toggles
- Recolored and/or hidden sample spheres reflecting the currently selected metadata category

## How to apply

Parse the QIIME principal coordinates file and the metadata mapping file with the QIIME/PyCogent Python libraries to obtain per-sample coordinates and every metadata column, including MIxS-family fields from the Genomic Standards Consortium. Generate the standalone HTML5/WebGL page with EMPeror's Python backend, embedding the coordinates and the full set of metadata columns directly in the page so no static per-category files are needed. Implement a JavaScript control panel that enumerates all metadata categories and, on selection, recolors the sample spheres as a function of their value for that category. Critically, keep the coloring logic separate from the visibility logic: changing the color category must never alter which points are shown, and each category gets its own show/hide toggle. Validate by loading a large combined example dataset in Chrome and confirming the controls respond interactively at scale.

## Related tools

- **EMPeror** (Tool that implements the decoupled coloring/visibility control panel; its Python backend generates the standalone HTML5/WebGL page from the PCoA and mapping files) — http://emperor.colorado.edu
- **QIIME** (Provides the Python libraries used to parse the principal coordinates file and the metadata mapping file)
- **PyCogent** (Supporting Python dependency listed with QIIME for parsing the input files)

## Examples

```
python scripts/make_emperor.py -i pcoa.txt -m mapping.txt -o emperor_output/  # then open emperor_output/emperor.html in Chrome
```

## Evaluation signals

- Changing the selected metadata color category does not alter the set of visible points (coloring and visibility are orthogonal).
- Each metadata category exposes an independent show/hide toggle that hides or reveals exactly the spheres with a value for that category.
- The produced page runs in Chrome without plugins and loads in seconds rather than minutes, even for thousands of samples.
- A combined example dataset of 5,740 samples and 120 metadata columns loads and the controls respond interactively.
- File size stays small (e.g. 26 MB vs 1.85 GB in KiNG for the same dataset), enabling embedding in emails or web pages.

## Limitations

- Graphical user interface requires the Chrome browser; compatibility with other browsers is untested per the article's stated requirements.
- The command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support stated.
- Depends on Python 2.7, an end-of-life Python version, plus QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.
- Comparison numbers are against KiNG static per-category files; this decoupled approach requires embedding all metadata in the page, which is the tradeoff EMPeror makes to avoid 1.85 GB files.

## Evidence

- [other] Coloring/visibility decoupling: [evidence span withheld — anchor status: not_found]
- [abstract] File-size advantage of embedding metadata: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25]"
- [abstract] Interactivity and scale: "scale to thousands of points with minimal load times (seconds versus many minutes in KiNG)"
- [abstract] Sharing without plugins: "ease of sharing, given EMPeror's small output file size, enables agile collaboration by allowing users to embed these visualizations via emails or web pages without the need for extra plugins"
- [discussion] Runtime requirements: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
