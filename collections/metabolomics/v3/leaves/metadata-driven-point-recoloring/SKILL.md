---
name: metadata-driven-point-recoloring
description: Use when a QIIME PCoA plot of beta-diversity ordination must be recolored
  or its points shown/hidden per arbitrary metadata column (categorical, gradient,
  or MIxS standard) for exploratory visual analysis of large microbial community datasets.
when_to_use_negative:
- Input is not a PCoA/beta-diversity ordination paired with a metadata mapping file,
  e.g. raw sequencing reads or a 2D heatmap where ordination-based recoloring does
  not apply.
- The task requires heavy custom computation inside the visualization (EMPeror is
  an exploratory WebGL viewer in Chrome, not a statistics engine for post-hoc analysis).
- A reproducible static figure for publication is needed rather than an interactive
  browser-based exploration; generic plotting packages like MATLAB, R, Excel, or Matplotlib
  would be more suitable.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3697
tools:
- name: EMPeror
  role: Generates the standalone HTML5/WebGL ordination page and implements the metadata-driven
    recoloring and show/hide controllers.
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Supplies the Python libraries used to parse the principal coordinates and
    mapping files and produces the PCoA plots that EMPeror visualizes.
- name: PyCogent
  role: Required dependency alongside QIIME's Python libraries for parsing inputs
    on the command line.
- name: KiNG
  role: 'Baseline tool superseded for this skill: EMPeror reduces its 1.85 GB file
    output to 26 MB (1.3%) and loads in seconds instead of minutes.'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:3c6e8eeb07642943f47293d29febae27001550e76ebd7d4ec1789045eef87b87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - KiNG
  - PyCogent
  - QIIME
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# metadata-driven-point-recoloring

## Summary

Recolor and toggle visibility of 3D principal-coordinates plot points as a function of per-sample metadata categories (including MIxS-family GSC fields), using EMPeror's decoupled coloring and visibility controllers embedded in a standalone HTML5/WebGL page.

## When to use

Use when a QIIME PCoA plot of beta-diversity ordination must be recolored or its points shown/hidden per arbitrary metadata column (categorical, gradient, or MIxS standard) for exploratory visual analysis of large microbial community datasets.

## When NOT to use

- Input is not a PCoA/beta-diversity ordination paired with a metadata mapping file, e.g. raw sequencing reads or a 2D heatmap where ordination-based recoloring does not apply.
- The task requires heavy custom computation inside the visualization (EMPeror is an exploratory WebGL viewer in Chrome, not a statistics engine for post-hoc analysis).
- A reproducible static figure for publication is needed rather than an interactive browser-based exploration; generic plotting packages like MATLAB, R, Excel, or Matplotlib would be more suitable.

## Inputs

- QIIME principal coordinates file (PCoA of a beta-diversity distance matrix)
- QIIME metadata mapping file with per-sample metadata columns (including MIxS-family GSC standards)

## Outputs

- Standalone HTML5/WebGL visualization page (26 MB for 5,740 samples and 120 metadata columns) with interactive recoloring and show/hide controls
- 3D ordination plot in which point color and visibility are driven by the selected metadata category

## How to apply

Parse the QIIME principal coordinates file together with the metadata mapping file in Python using the QIIME and PyCogent libraries to obtain sample coordinates and per-sample metadata columns. Generate the standalone EMPeror HTML5/WebGL visualization page via its Python command line interface, embedding the coordinates and all metadata columns (including MIxS-family fields) in the page rather than emitting static per-category files. In the resulting JavaScript control panel, select a metadata category to recolor the sample spheres as a function of their values for that category, keeping the coloring logic separate from the visibility logic so color and show/hide can be changed independently. Optionally extend the same controllers to visualize gradients and categorical data, biplots, procrustes coordinate comparisons, and jackknifed-resampled beta diversity plots. Load the page in Chrome and confirm the panel responds interactively to category switches.

## Related tools

- **EMPeror** (Generates the standalone HTML5/WebGL ordination page and implements the metadata-driven recoloring and show/hide controllers.) — http://emperor.colorado.edu
- **QIIME** (Supplies the Python libraries used to parse the principal coordinates and mapping files and produces the PCoA plots that EMPeror visualizes.)
- **PyCogent** (Required dependency alongside QIIME's Python libraries for parsing inputs on the command line.)
- **KiNG** (Baseline tool superseded for this skill: EMPeror reduces its 1.85 GB file output to 26 MB (1.3%) and loads in seconds instead of minutes.)

## Examples

```
python emperor.py -i pcoa.txt -m mapping.txt -o emperor_output
```

## Evaluation signals

- The generated page opens in Chrome without extra plugins and the metadata control panel enumerates all metadata categories including MIxS-family fields.
- Selecting a category recolors sample spheres as a function of their values for that column without altering which points are shown (coloring and visibility remain decoupled).
- For a 5,740-sample, 120-column dataset the output file is ~26 MB, not the 1.85 GB per-category files KiNG would produce (1.3% of KiNG size).
- Page load completes in seconds rather than the many minutes KiNG requires at this scale.
- Toggling a category's show/hide control changes point visibility independently of the color category.

## Limitations

- Depends on Python 2.7 (end-of-life), Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0, and PyCogent; the command line interface runs only on OS X 10.6+ and Linux.
- No Windows support for the CLI, and the GUI is only validated in Chrome, so cross-browser behavior is untested.
- Supports at most two features enabled simultaneously in some controller combinations; separate per-category files are avoided by embedding metadata in the page, but the tool relies on WebGL support.
- No version number or archival DOI is stated for the software itself in the article.

## Evidence

- [abstract] EMPeror includes a rich set of controllers to modify features as a function of the metadata.: "EMPeror includes a rich set of controllers to modify features as a function of the metadata."
- [abstract] ability to visualize gradients and categorical data, visualize different principal coordinates axes: "ability to visualize gradients and categorical data, visualize different principal coordinates axes, present the data in the form of parallel coordinates, show taxa as well as environmental samples,"
- [abstract] ease of sharing, given EMPeror's small output file size, enables agile collaboration by allowing users to embed these visualizations via emails or web pages without the need for extra plugins: "ease of sharing, given EMPeror's small output file size, enables agile collaboration by allowing users to embed these visualizations via emails or web pages without the need for extra plugins"
- [discussion] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
- [discussion] Project home page: http://emperor.colorado.edu: "Project home page: http://emperor.colorado.edu"
