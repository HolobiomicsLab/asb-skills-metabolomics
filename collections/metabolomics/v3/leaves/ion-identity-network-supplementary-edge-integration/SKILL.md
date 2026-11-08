---
name: ion-identity-network-supplementary-edge-integration
description: Use when you have an aligned feature list (feature quantification .csv
  + MS2 .mgf) plus IIN supplementary edge files and want to compare MS2-only FBMN
  against IIMN, or to merge redundant adduct/isotope subnetworks of the same compound.
when_to_use_negative:
- Input is already a collapsed network or an annotated feature table without raw-feature
  IIN edges — no ion-identity information is available to supply as supplementary
  edges
- Datasets lacking co-eluting ion species information or chromatographic peak shape
  data suitable for feature shape correlation (correlation needs at least five data
  points, two on each edge)
- Analysis goals that require GC-MS or MS2-only data without MS1 feature grouping
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine
  role: Feature finding, alignment, MS1-based ion identity annotation, and export
    of feature table, .mgf, and IIN supplementary edge files
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Web server running FBMN and FBMN-with-IIN (IIMN) workflows from the exported
    files
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: xcms-gnps-tools
  role: Extract CAMERA adduct annotations as IIN supplementary edges in the format
    required for FBMN/GNPS
  repo: https://github.com/jorainer/xcms-gnps-tools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:1d21de5390a05794e1837eef2f952de90ecd71f60cb77cc44d81458de263dfb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MZmine
  - xcms-gnps-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# ion-identity-network-supplementary-edge-integration

## Summary

Runs GNPS Feature-Based Molecular Networking a second time while supplying MS1-derived ion identity network (IIN) supplementary edges (IIMN), so that multiple ion species of the same compound are connected and can be collapsed, improving connectivity and annotation propagation compared to MS2-similarity-only FBMN.

## When to use

Use when you have an aligned feature list (feature quantification .csv + MS2 .mgf) plus IIN supplementary edge files and want to compare MS2-only FBMN against IIMN, or to merge redundant adduct/isotope subnetworks of the same compound.

## When NOT to use

- Input is already a collapsed network or an annotated feature table without raw-feature IIN edges — no ion-identity information is available to supply as supplementary edges
- Datasets lacking co-eluting ion species information or chromatographic peak shape data suitable for feature shape correlation (correlation needs at least five data points, two on each edge)
- Analysis goals that require GC-MS or MS2-only data without MS1 feature grouping

## Inputs

- Aligned feature list from MZmine (or MS-DIAL/XCMS) covering all samples
- Feature quantification table (.csv)
- MS2 spectral summary file (.mgf)
- IIN supplementary edge files (.csv) with columns ID1 and ID2

## Outputs

- FBMN-only molecular network (MS2 spectral similarity edges)
- IIMN result with two connectivity layers (MS1 ion-identity edges plus MS2 similarity edges)
- Collapsed IIMN network with reduced node count and retained MS2 edges
- Topology comparison table (e.g., iimn_fbmn_comparison.csv) of nodes and edges before/after collapsing

## How to apply

After processing LC-MS/MS data in MZmine to an aligned feature list, export the feature quantification table (.csv), the MS2 spectral summary (.mgf), and the IIN supplementary edge files (.csv). Run GNPS FBMN once without supplementary edges to obtain the MS2-similarity-only network, then run FBMN again supplying the IIN edges, using ion-species grouping with an RT tolerance of 0.1 min, an ion-identity m/z tolerance of 0.001 m/z or 10 ppm, and a feature shape Pearson correlation of at least 0.85 with at least five data points (two on each edge). In the IIMN result, collapse ion-identity nodes of the same compound and re-count nodes and edges, retaining the MS2 similarity edges. The rationale is that MS2-based networks leave ion species of one molecule in redundant, disconnected subnetworks, whereas the MS1 ion-identity layer links them and propagates annotations. A companion route for xcms users is to extract CAMERA-derived adduct annotations via jorainer/xcms-gnps-tools in the IIN edge format required by FBMN/GNPS.

## Related tools

- **MZmine** (Feature finding, alignment, MS1-based ion identity annotation, and export of feature table, .mgf, and IIN supplementary edge files) — https://github.com/mzmine/mzmine
- **GNPS** (Web server running FBMN and FBMN-with-IIN (IIMN) workflows from the exported files) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **xcms-gnps-tools** (Extract CAMERA adduct annotations as IIN supplementary edges in the format required for FBMN/GNPS) — https://github.com/jorainer/xcms-gnps-tools

## Evaluation signals

- Under MS2-only FBMN, ion species of the same compound appear in redundant, disconnected subnetworks; after IIN integration these merge into a single connected network
- Node count decreases after collapsing ion-identity nodes while MS2 similarity edges are retained among representative nodes
- Supplementary edge files follow the generic comma-separated format with ID1 and ID2 columns matching feature IDs in the quantification table and .mgf
- Grouping parameters satisfied: RT tolerance 0.1 min, m/z tolerance 0.001 m/z or 10 ppm, feature shape Pearson correlation ≥ 0.85 with ≥ 5 data points (2 on each edge)
- Collapsed network shows increased edge density among nodes annotated to the same compound class (e.g., free vs amino-acid/sulfate-conjugated bile acids)

## Limitations

- The IIN edge format (Supplementary Pairs text file) has no formal schema or validator tool; malformed ID1/ID2 pairs are not explicitly validated
- Specific MZmine/MS-DIAL/XCMS versions used for processing are not pinned in the article, and the MZmine IIMN build is described as a 'special IIMN build' — unclear whether it is part of a released version
- Annotations propagate only for structurally related molecules sharing ion-identity or MS2 similarity relationships; unannotated classes (e.g., unknown ion-ligand complexes) require follow-up inspection
- No container image or environment file is provided for reproducibility

## Evidence

- [abstract] abstract: "integrates chromatographic peak shape correlation analysis into molecular networks to connect and collapse different ion species of the same molecule"
- [discussion] workflow: [evidence span withheld — anchor status: not_found]
- [discussion] filter: "a maximum tolerance of 0.001 m/z or 10 ppm"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [discussion] edges-format: "The text format follows a generic comma-separated style with the columns ID1 and ID2"
