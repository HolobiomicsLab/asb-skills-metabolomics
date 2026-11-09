---
name: mixs-metadata-standards-handling
description: Use when you have a QIIME principal coordinates file plus a metadata
  mapping file with per-sample MIxS or GSC-standard columns and want to visually recolor
  or hide/show 3D PCoA points according to an arbitrary metadata category.
when_to_use_negative:
- The goal is a static publication figure of a single metadata category rather than
  an interactive exploration — a generic 3D plotting package (MATLAB, R, Matplotlib)
  may suffice.
- The input is a set of pre-rendered per-category image files rather than coordinates
  plus a mapping file — EMPeror's dynamic metadata-driven control cannot be driven
  from static images.
- The user environment has no Chrome browser available, since EMPeror's GUI explicitly
  requires Chrome.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: EMPeror
  role: Generates the standalone HTML5/WebGL visualization and provides the interactive
    control panel that drives metadata-driven show/hide and recoloring of principal
    coordinates points.
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Provides the principal coordinates file and mapping-file parsers used to load
    sample coordinates and per-sample metadata columns into the Python backend.
- name: PyCogent
  role: Python library dependency used together with QIIME to parse the mapping file
    and coordinate data in the EMPeror backend.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:b06acffbd83367651f6652c23a09fe5517e112263f792c8e5064c9ce932bb1ad
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mixs-metadata-standards-handling

## Summary

Treat MIxS-family metadata from the Genomic Standards Consortium as a first-class categorical layer in an EMPeror principal coordinates visualization: enumerate every metadata column, then use it to drive show/hide visibility and point recoloring independently so samples can be explored from any environmental or clinical angle.

## When to use

Use when you have a QIIME principal coordinates file plus a metadata mapping file with per-sample MIxS or GSC-standard columns and want to visually recolor or hide/show 3D PCoA points according to an arbitrary metadata category.

## When NOT to use

- The goal is a static publication figure of a single metadata category rather than an interactive exploration — a generic 3D plotting package (MATLAB, R, Matplotlib) may suffice.
- The input is a set of pre-rendered per-category image files rather than coordinates plus a mapping file — EMPeror's dynamic metadata-driven control cannot be driven from static images.
- The user environment has no Chrome browser available, since EMPeror's GUI explicitly requires Chrome.

## Inputs

- QIIME principal coordinates file (sample coordinates from PCoA of a beta diversity distance matrix)
- QIIME metadata mapping file with per-sample columns, including MIxS-family Genomic Standards Consortium metadata fields
- Optional auxiliary datasets such as biplot vectors, jackknifed beta diversity results, or procrustes comparison coordinates

## Outputs

- Standalone HTML5/WebGL visualization page (EMPeror output) embedding coordinates and metadata columns
- Interactive control panel enumerating all metadata categories with per-category recoloring and show/hide toggles
- On-screen 3D point cloud colored by the selected metadata category, with visibility toggles per category

## How to apply

Parse the QIIME principal coordinates file and the accompanying mapping file with Python using the QIIME/PyCogent libraries to obtain sample coordinates and per-sample metadata columns. Run EMPeror's Python backend to generate a standalone HTML5/WebGL page that embeds the coordinates together with all metadata columns, so no static per-category image files are needed. In the emitted JavaScript control panel, enumerate every metadata category (including MIxS-family fields) and, on selection, recolor the sample spheres as a function of their value for that category. Keep the coloring logic decoupled from the visibility logic: changing the color category must not alter which points are shown, and each category must expose its own per-category show/hide toggle. Validate by loading the combined example dataset (5,740 samples and 120 metadata columns from the GigaScience Database / EMPeror FTP site) in Chrome and confirming that show, hide, and recolor respond interactively.

## Related tools

- **EMPeror** (Generates the standalone HTML5/WebGL visualization and provides the interactive control panel that drives metadata-driven show/hide and recoloring of principal coordinates points.) — http://emperor.colorado.edu
- **QIIME** (Provides the principal coordinates file and mapping-file parsers used to load sample coordinates and per-sample metadata columns into the Python backend.)
- **PyCogent** (Python library dependency used together with QIIME to parse the mapping file and coordinate data in the EMPeror backend.)

## Examples

```
python -m qiime.scripts.make_emperor.py -i pcoa.txt -m mapping_file.txt -o emperor_output
```

## Evaluation signals

- Output page runs in Chrome without any extra plugins and loads in seconds rather than minutes.
- Selecting each metadata category in the control panel recolors the sample spheres according to their value for that category.
- Changing the color category does not change which points are visible — coloring and visibility must be independently controllable.
- Each metadata category exposes a working per-category show/hide toggle that visually hides and re-shows its points.
- The large example dataset (5,740 samples, 120 metadata columns) loads and the control remains interactive, confirming scaling beyond toy datasets.

## Limitations

- Graphical user interface requires Chrome specifically; compatibility with other browsers is not stated.
- Command-line interface is limited to OS X (10.6 and higher) and Linux; no Windows support is mentioned.
- Depends on Python 2.7, NumPy, BIOM 1.1.0 and PyCogent, an aging dependency stack.
- No archival DOI or accession IDs are given for the supporting example files, which are only available via the GigaScience Database and EMPeror FTP site.

## Evidence

- [other] EMPeror implements metadata-driven visual control by letting the user choose point coloring dynamically and separating coloring from visibility, so points can be recolored and shown/hidden independently per metadata category.: [evidence span withheld — anchor status: not_found]
- [other] Parse the QIIME principal coordinates file and the metadata mapping file with Python (QIIME/PyCogent libraries) to obtain sample coordinates and per-sample metadata columns.: [evidence span withheld — anchor status: not_found]
- [discussion] Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent.: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
- [abstract] EMPeror includes a rich set of controllers to modify features as a function of the metadata.: "EMPeror includes a rich set of controllers to modify features as a function of the metadata."
