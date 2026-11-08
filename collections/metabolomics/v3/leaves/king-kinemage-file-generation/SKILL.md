---
name: king-kinemage-file-generation
description: Use when producing KiNG-format 3D kinemage plots from a QIIME principal
  coordinates file plus a metadata mapping file — e.g. as the legacy export for discrete
  or gradient coloring, or as the size/load-time baseline benchmarked against EMPeror.
when_to_use_negative:
- 'The goal is interactive, shareable exploration of thousands-of-sample PCoA plots:
  EMPeror''s native output (26 MB, loads in seconds, embeddable in emails or web pages
  without plugins) is the right path, not a KiNG kinemage export that balloons to
  1.85 GB and loads in many minutes.'
- Input is a beta diversity distance matrix that has not yet been ordinated — run
  PCoA first; this skill consumes a QIIME principal coordinates file plus a metadata
  mapping file, not raw sequencing data or an un-ordinated distance matrix.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3941
tools:
- name: QIIME
  role: Supplies the KiNG export path that converts a principal coordinates file +
    metadata mapping file into 3D kinemage plots, replicating coordinates for each
    metadata category
- name: KiNG
  role: 'Target format and viewer: the generated kinemage files are loaded in KiNG
    in the browser to measure load time for the discrete and gradient coloring variants'
- name: EMPeror
  role: 'Comparator baseline: EMPeror''s visualization generated from the same principal
    coordinates and mapping files (26 MB, seconds to load) benchmarks the KiNG export
    (1.85 GB, many minutes)'
  repo: http://emperor.colorado.edu
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/2047-217x-2-16
    title: 'EMPeror: a tool for visualizing high-throughput microbial community data'
  source_content_hash: sha256:9ec5a4faa0818de583f0c810263649f055bac01d36e01ebfe4c4b05a5467996c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPeror
  - KiNG
  - QIIME
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# king-kinemage-file-generation

## Summary

Generates KiNG-viewable 3D kinemage files from QIIME principal coordinates and metadata mapping files in discrete and gradient coloring variants — the legacy QIIME 3D-plot export path whose per-category coordinate replication inflates file sizes (1.85 GB for 5,740 samples) and served as the baseline EMPeror was benchmarked against.

## When to use

Use when producing KiNG-format 3D kinemage plots from a QIIME principal coordinates file plus a metadata mapping file — e.g. as the legacy export for discrete or gradient coloring, or as the size/load-time baseline benchmarked against EMPeror.

## When NOT to use

- The goal is interactive, shareable exploration of thousands-of-sample PCoA plots: EMPeror's native output (26 MB, loads in seconds, embeddable in emails or web pages without plugins) is the right path, not a KiNG kinemage export that balloons to 1.85 GB and loads in many minutes.
- Input is a beta diversity distance matrix that has not yet been ordinated — run PCoA first; this skill consumes a QIIME principal coordinates file plus a metadata mapping file, not raw sequencing data or an un-ordinated distance matrix.

## Inputs

- QIIME principal coordinates file (PCoA of a beta diversity distance matrix), e.g. for the combined 5,740-sample dataset
- QIIME metadata mapping file (e.g. 120 metadata columns for the 5,740-sample dataset)
- Coloring specification: discrete metadata categories and/or gradient metadata values

## Outputs

- KiNG-format 3D kinemage plot file per coloring variant (discrete and gradient); ≈1.85 GB total for the 5,740-sample, 120-column example
- load_time_comparison.csv recording measured load times and file sizes for the KiNG outputs (and the EMPeror comparator)

## How to apply

Start from the two QIIME artifacts used in the paper's example (the combined 5,740-sample, 120-column dataset): a principal coordinates file — PCoA of a beta diversity distance matrix — and a metadata mapping file, obtainable from the GigaScience Database or the EMPeror FTP site. Run QIIME's KiNG export path, which replicates the sample coordinates for each metadata category to build the kinemage, and emit two variants — discrete category coloring and gradient coloring — since recoloring at view time is not possible, each variant must be encoded in the file. Expect file size to scale with samples × metadata columns because of this replication: for the paper's dataset both variants total 1.85 GB, versus 26 MB (1.3%) for EMPeror from the same inputs. Load each kinemage in KiNG in the browser (Chrome, per the paper's stated requirements) and time it, recording the measurement for both coloring variants. Write all timings and file sizes to load_time_comparison.csv and compute the EMPeror/KiNG size ratio; accept the result if EMPeror is ≈1.3% of the KiNG file size and KiNG loads in many minutes while EMPeror loads in seconds.

## Related tools

- **QIIME** (Supplies the KiNG export path that converts a principal coordinates file + metadata mapping file into 3D kinemage plots, replicating coordinates for each metadata category)
- **KiNG** (Target format and viewer: the generated kinemage files are loaded in KiNG in the browser to measure load time for the discrete and gradient coloring variants)
- **EMPeror** (Comparator baseline: EMPeror's visualization generated from the same principal coordinates and mapping files (26 MB, seconds to load) benchmarks the KiNG export (1.85 GB, many minutes)) — http://emperor.colorado.edu

## Evaluation signals

- For the 5,740-sample × 120-metadata-column dataset, the KiNG kinemage files for both the discrete and gradient coloring variants total ≈1.85 GB; a far smaller file indicates the per-category coordinate replication was skipped.
- EMPeror generated from the same principal coordinates and mapping files is ≈26 MB, i.e. ~1.3% of the KiNG file size — compute and record the ratio in load_time_comparison.csv and check it against ~1.3%.
- KiNG load time in the browser is many minutes while EMPeror loads in seconds; if KiNG loads in seconds, the dataset was likely reduced or replication failed.
- Each kinemage opens in KiNG and renders the PCoA points colored by mapping-file categories for the discrete variant and by gradient values for the gradient variant.
- Both coloring variants (discrete and gradient) were generated separately, matching the workflow's two-variant measurement design.

## Limitations

- Coordinate replication per metadata category makes KiNG file size scale with samples × metadata columns: 5,740 samples × 120 columns → 1.85 GB (discrete and gradient each), versus EMPeror's 26 MB, and browser loading takes many minutes vs seconds.
- Recoloring requires regenerating a file per variant, because QIIME's KiNG export path replicates coordinates for each metadata category rather than recoloring points dynamically at view time.
- The pipeline depends on the QIIME python libraries in the paper's stated Python 2.7 environment with Chrome as the specified browser; the associated command line interface is stated for OS X (10.6 and higher) and Linux only.

## Evidence

- [abstract] QIIME typically generates 3D plots using KiNG: "QIIME [7], an open source framework for upstream and downstream analysis of microbial community samples generated via high-throughput sequencing instruments, typically generates 3D plots using KiNG"
- [abstract] KiNG files for discrete and gradient coloring reach 1.85 GB vs 26 MB (1.3%) in EMPeror: "In KiNG, the resulting files for both the discrete and gradient coloring result in a size of 1.85 GB, but in EMPeror only 26 MB [25], meaning only 1.3% of the original size"
- [abstract] Load-time gap: seconds (EMPeror) versus many minutes (KiNG): "scale to thousands of points with minimal load times (seconds versus many minutes in KiNG)"
- [discussion] Supporting example files are available in the GigaScience Database and EMPeror FTP site: "The example files and additional data sets supporting the results of this article are available in the GigaScience Database [24], as well as from the EMPeror FTP site [25]."
- [discussion] Stated requirements include Python 2.7, Chrome and the QIIME python libraries: "Other Requirements: Python 2.7, Chrome, QIIME (python libraries only), NumPy, BIOM 1.1.0 and PyCogent."
