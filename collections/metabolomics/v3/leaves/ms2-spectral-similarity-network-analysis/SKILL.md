---
name: ms2-spectral-similarity-network-analysis
description: Use when an aligned LC-MS/MS feature table (.csv), consensus MS2 spectra
  (.mgf), and ion-identity supplementary edges are available and network connectivity
  or feature redundancy across adducts and isotopes is the bottleneck.
when_to_use_negative:
- Input is already a fully processed feature table without raw MS1 peak-shape data,
  so no ion-identity grouping or peak-shape correlation can be computed.
- The dataset contains only MS1 features without consensus MS2 spectra, so no MS2
  spectral similarity edges can be formed.
- The analysis goal is single-spectrum library search rather than network-level connectivity
  analysis across features.
edam_operation: http://edamontology.org/operation_3620
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Web server running the FBMN/IIMN workflow that builds MS2 spectral similarity
    networks from feature tables and MS2 summaries
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
- name: MZmine
  role: Feature-finding tool that performs MS1-based feature grouping, ion identity
    annotation, and exports the feature quantification table, MGF, and IIN supplementary
    edges
  repo: https://github.com/mzmine/mzmine
- name: XCMS
  role: Alternative R-based feature-finding tool interfaced with IIMN; its CAMER-derived
    adduct annotations feed IIN edges via jorainer/xcms-gnps-tools
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: MS-DIAL
  role: Alternative open source feature-finding tool that can generate the aligned
    feature list and IIN files for IIMN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:15ddb4e4cc0de0f93fc0ef64aa527ac5a71bdce5eee713dd58ed7ea540e3b9b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MS-DIAL
  - MZmine
  - XCMS
  edam_operation: http://edamontology.org/operation_3620
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# MS2 spectral similarity network analysis

## Summary

Builds molecular networks by connecting LC-MS/MS feature nodes through MS2 spectral similarity edges, and fuses them with MS1 ion-identity edges (IIMN) so that ion species of the same compound collapse into one node while structurally related molecules gain improved connectivity.

## When to use

Use when an aligned LC-MS/MS feature table (.csv), consensus MS2 spectra (.mgf), and ion-identity supplementary edges are available and network connectivity or feature redundancy across adducts and isotopes is the bottleneck.

## When NOT to use

- Input is already a fully processed feature table without raw MS1 peak-shape data, so no ion-identity grouping or peak-shape correlation can be computed.
- The dataset contains only MS1 features without consensus MS2 spectra, so no MS2 spectral similarity edges can be formed.
- The analysis goal is single-spectrum library search rather than network-level connectivity analysis across features.

## Inputs

- Aligned feature quantification table (.csv) from MZmine, MS-DIAL, or XCMS
- Consensus MS2 spectral summary file (.mgf)
- IIN supplementary edge files (.csv) from MS1-based feature grouping
- Raw LC-MS/MS data converted to open format (e.g. mzML)

## Outputs

- Ion identity molecular network with two layers of feature connectivity (MS1 ion identity and MS2 spectral similarity)
- Network graph with collapsed representative nodes and reduced feature redundancy
- Annotation/propagation of compound-class information across connected ion species
- Node and edge counts before and after IIN collapse for comparison

## How to apply

Process raw LC-MS/MS files into an aligned feature list with MZmine, MS-DIAL, or XCMS, exporting a feature quantification table (.csv), an MS2 spectral summary (.mgf), and IIN supplementary edge files (.csv). Upload these to GNPS and run Feature-Based Molecular Networking (FBMN), optionally supplying the IIN supplementary edges to generate Ion Identity Molecular Networking (IIMN). Use MS2 spectral similarity to link structurally related compounds and MS1-derived ion-identity edges (e.g. RT tolerance 0.1 min, 0.001 m/z / 10 ppm, feature-shape Pearson correlation ≥ 0.85 with at least five data points) to connect ion species of the same compound. In the IIMN output, collapse ion-identity nodes to reduce feature redundancy and compare node/edge counts before and after collapsing; propagate annotations across the two network layers to enhance annotation of structurally related molecules.

## Related tools

- **GNPS** (Web server running the FBMN/IIMN workflow that builds MS2 spectral similarity networks from feature tables and MS2 summaries) — http://github.com/CCMS-UCSD/GNPS_Workflows
- **MZmine** (Feature-finding tool that performs MS1-based feature grouping, ion identity annotation, and exports the feature quantification table, MGF, and IIN supplementary edges) — https://github.com/mzmine/mzmine
- **XCMS** (Alternative R-based feature-finding tool interfaced with IIMN; its CAMER-derived adduct annotations feed IIN edges via jorainer/xcms-gnps-tools) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **MS-DIAL** (Alternative open source feature-finding tool that can generate the aligned feature list and IIN files for IIMN)

## Examples

```
# Run FBMN with IIN supplementary edges on GNPS
# Upload: feature_table.csv, ms2.mgf, iin_edges.csv
# Workflow: FEATURE-BASED-MOLECULAR-NETWORKING + Ion Identity Networking
```

## Evaluation signals

- The resulting network contains two distinct layers of node connectivity: MS1-derived ion identity edges linking ion species of the same compound, and MS2 spectral similarity edges linking structurally related compounds.
- Collapsing ion-identity nodes reduces node count while retaining the same number of MS2 similarity edges among representative nodes.
- The bile-acid example: FBMN alone yields only one edge between free and conjugated bile-acid subnetworks, whereas IIMN merges them into one large network.
- IIN edges satisfy the grouping thresholds: RT tolerance ≤ 0.1 min, 0.001 m/z / 10 ppm, feature-shape Pearson correlation ≥ 0.85 with ≥ 5 data points, and ≥ 50% overlapping intensity in at least two samples.
- GNPS library matches on nodes meet a minimum cosine similarity of 0.9 and at least 4–6 shared fragment ions.

## Limitations

- IIN supplementary edge files must follow a generic comma-separated format with columns ID1 and ID2; no formal schema validator is provided.
- Variable feature detection and alignment parameters across datasets are documented only in Supplementary Table 4, so cross-dataset comparisons require manual parameter reconciliation.
- The IIMN build in MZmine is described as a 'special IIMN build'; availability of the two IIMN modules in released MZmine versions is not guaranteed.
- Ions with unassigned charge states and isotope peaks must be excluded from MS2 acquisition upstream; dynamic precursor exclusion (e.g. 5 s) affects MS2 spectrum quality.
- Feature-shape correlation requires at least five data points with two on each edge; low-resolution or short-peak datasets may fail this minimum.

## Evidence

- [abstract] IIMN connects ion species and collapses them into representative nodes, improving network connectivity and reducing feature redundancy.: "integrates chromatographic peak shape correlation analysis into molecular networks to connect and collapse different ion species of the same molecule"
- [intro] FBMN establishes MS2 spectral similarity edges between structurally related compounds; IIMN adds a second layer of MS1-based ion identity edges.: "The resulting ion identity molecular networks contain two layers of feature (node) connectivity, linking ion identities of the same compound by MS1 characteristics and structurally similar compounds"
- [discussion] The IIN grouping thresholds that gate whether two features are connected as the same ion identity.: "a feature shape Pearson correlation greater equals 0.85 with at least five data points and two data points on each edge"
- [discussion] File formats that must be exported from MZmine/MS-DIAL/XCMS and uploaded to GNPS for the IIMN workflow.: [evidence span withheld — anchor status: not_found]
- [readme] Interoperability of the IIMN workflow across the three supported feature-finding tools.: "we interfaced the IIMN workflow with three widely used open source MS processing tools (MZmine17, MS-DIAL18, and XCMS13,16)"
- [discussion] GNPS spectral library match threshold used for node annotation within the network.: "contain a match to the GNPS spectral libraries with a minimum cosine similarity of 0.9 and a minimum number of shared fragment ions of 4–6"
