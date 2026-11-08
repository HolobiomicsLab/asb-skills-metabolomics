---
name: metadata-driven-visual-property-mapping
description: Use when a QIIME principal coordinates file and metadata mapping file
  must be rendered as a 3D PCoA plot whose sphere size, transparency, or coloring
  changes as a function of metadata categories (e.g. body site).
when_to_use_negative:
- The goal is upstream ordination itself (computing PCoA from a beta diversity distance
  matrix) rather than visualizing existing coordinates — run the ordination step first,
  then apply this skill
- No metadata mapping file is available, so visual properties cannot be tied to categories
  and only default sphere rendering is possible
- The target environment cannot run Chrome or WebGL, since the interactive GUI depends
  on that specific browser and hardware-accelerated graphics
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: EMPeror
  role: Provides the HTML5/WebGL/JavaScript/Python rendering framework and the rich
    set of controllers that modify sphere size, transparency, and coloring as a function
    of metadata
  repo: http://emperor.colorado.edu
- name: QIIME
  role: Upstream source of the principal coordinates files and metadata mapping files
    consumed by the metadata-driven property mapping (QIIME python libraries are a
    stated requirement)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:72bf6d9deb0094ee2091ee23b85a0345249c4c3196bfc90812afe26c0dd0fde9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# metadata-driven-visual-property-mapping

## Summary

Maps sample metadata categories onto visual properties (size, transparency, color) of spheres in EMPeror's WebGL-based 3D principal coordinates plots, enabling per-category visual comparison of microbial community samples.

## When to use

Use when a QIIME principal coordinates file and metadata mapping file must be rendered as a 3D PCoA plot whose sphere size, transparency, or coloring changes as a function of metadata categories (e.g. body site).

## When NOT to use

- The goal is upstream ordination itself (computing PCoA from a beta diversity distance matrix) rather than visualizing existing coordinates — run the ordination step first, then apply this skill
- No metadata mapping file is available, so visual properties cannot be tied to categories and only default sphere rendering is possible
- The target environment cannot run Chrome or WebGL, since the interactive GUI depends on that specific browser and hardware-accelerated graphics

## Inputs

- QIIME principal coordinates file (PCoA coordinates of a beta diversity distance matrix)
- QIIME metadata mapping file (per-sample category memberships, e.g. body site)
- Sample-level metadata categories to drive visual properties

## Outputs

- Self-contained HTML5/WebGL web page rendering samples as spheres in 3D
- Per-category controller UI that adjusts sphere size and transparency from the metadata
- Optionally recolored/re-hidden points based on dynamic metadata-driven visibility settings

## How to apply

Parse the QIIME principal coordinates file and the metadata mapping file to extract per-sample 3D coordinates and per-sample category memberships. Render each sample as a sphere in an HTML5 canvas using WebGL hardware-accelerated graphics, exposing sphere size and transparency as per-vertex attributes rather than baked-in constants, so the browser can update them without regenerating the scene. Implement a per-category controller that maps each selected metadata category (e.g. body site, age) to a size value and a transparency value applied to all spheres belonging to that category. Deliver the renderer and controller as a self-contained web page viewable in Chrome, since EMPeror's GUI requires Chrome and its output can be embedded in emails or web pages without extra plugins. Rationale: per-category, metadata-driven control is what lets researchers rapidly perform exploratory investigations of 3D visualizations of high-throughput microbial community data.

## Related tools

- **EMPeror** (Provides the HTML5/WebGL/JavaScript/Python rendering framework and the rich set of controllers that modify sphere size, transparency, and coloring as a function of metadata) — http://emperor.colorado.edu
- **QIIME** (Upstream source of the principal coordinates files and metadata mapping files consumed by the metadata-driven property mapping (QIIME python libraries are a stated requirement))

## Evaluation signals

- The generated page loads and runs headlessly as a self-contained web page (no extra plugins required), consistent with EMPeror's web-browser-enabled design
- Expert review in Chrome confirms that selecting different metadata categories visibly changes sphere size and transparency on a per-category basis
- Sphere size and transparency are implemented as per-vertex attributes, so changing a category's values updates all spheres in that category without reloading the coordinates
- All samples from the QIIME principal coordinates file appear as spheres at their correct 3D coordinates, matched one-to-one with category memberships from the mapping file

## Limitations

- The graphical user interface requires the Chrome browser specifically, suggesting untested compatibility with other browsers
- The software depends on Python 2.7 (an end-of-life Python version), QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent
- The command line interface is limited to OS X (10.6 and higher) and Linux only; no Windows support is stated

## Evidence

- [abstract] abstract: "EMPeror includes a rich set of controllers to modify features as a function of the metadata."
- [discussion] discussion: "it contains several user interface features, enabling straightforward modifications and customization of perceptible aspects in the plot"
