---
name: feature-based-molecular-networking
description: Use when you have an aligned feature table (.csv) plus MS2 spectral summary
  (.mgf) from MZmine/MS-DIAL/XCMS and want a molecular network that groups features
  by MS2 similarity, or compares MS2-only vs IIN-augmented topology.
when_to_use_negative:
- Input is already a molecular network or a collapsed feature table — no ion-species
  redundancy remains to resolve
- Data lack MS2 spectra or co-elution information across samples, so neither MS2 similarity
  nor feature-shape correlation can be computed
- The goal is GC-MS networking — use the dedicated GC workflows (MSHUB-GC deconvolution,
  Library Search/Molecular Networking GC) instead of FBMN
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MZmine
  role: Feature finding, aligned feature list creation, MS1-based ion identity annotation,
    and export of the .csv/.mgf/IIN edge files consumed by FBMN
  repo: https://github.com/mzmine/mzmine
- name: GNPS
  role: Web server that runs the FBMN/IIMN workflow on the uploaded feature table,
    MGF, and supplementary edges
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
- name: XCMS
  role: Alternative feature-finding tool interfaced with IIMN; xcms-gnps-tools provides
    formatSpectraForGNPS() and getFeatureAnnotations() to produce FBMN/IIN inputs
  repo: https://github.com/jorainer/xcms-gnps-tools
- name: MS-DIAL
  role: Alternative open-source feature-finding tool interfaced with the IIMN workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-23953-9
    title: Ion identity molecular networking for mass spectrometry-based metabolomics
      in the GNPS environment
  source_content_hash: sha256:9d7115b1e30720d4b2df1766231de78c274ba65e56772ed491d5e0be55b217db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS
  - MS-DIAL
  - MZmine
  - XCMS
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# feature-based-molecular-networking

## Summary

Run GNPS Feature-Based Molecular Networking (FBMN) on an aligned LC-MS/MS feature list to connect MS2 spectra by spectral similarity, optionally augmented with MS1-derived ion identity networking (IIN) supplementary edges (IIMN) to link and collapse ion species of the same compound.

## When to use

Use when you have an aligned feature table (.csv) plus MS2 spectral summary (.mgf) from MZmine/MS-DIAL/XCMS and want a molecular network that groups features by MS2 similarity, or compares MS2-only vs IIN-augmented topology.

## When NOT to use

- Input is already a molecular network or a collapsed feature table — no ion-species redundancy remains to resolve
- Data lack MS2 spectra or co-elution information across samples, so neither MS2 similarity nor feature-shape correlation can be computed
- The goal is GC-MS networking — use the dedicated GC workflows (MSHUB-GC deconvolution, Library Search/Molecular Networking GC) instead of FBMN

## Inputs

- Aligned feature quantification table (.csv) from MZmine/MS-DIAL/XCMS
- MS2 spectral summary file (.mgf) with consensus spectra per feature
- IIN supplementary edge files (.csv) with ion identity grouping (columns ID1, ID2)
- Raw LC-MS/MS data converted to open format (e.g., mzML) for upstream feature finding

## Outputs

- FBMN molecular network with nodes (features) and MS2-similarity edges
- IIMN network with two layers of connectivity (MS1 ion-identity edges plus MS2 similarity edges)
- Collapsed representative-node network with reduced node count and increased edge density
- Topology comparison table (e.g., iimn_fbmn_comparison.csv) of node/edge counts before and after collapsing

## How to apply

Process the raw LC-MS/MS files in MZmine (or MS-DIAL/XCMS) into an aligned feature list, then export the feature quantification table (.csv), the consensus MS2 spectral summary (.mgf), and the IIN supplementary edge files (.csv). Upload these to GNPS and run FBMN once without supplementary edges to obtain the MS2-similarity-only network. For the IIMN variant, run FBMN a second time supplying the IIN edges, using an RT tolerance of 0.1 min, an ion-identity m/z tolerance of 0.001 m/z or 10 ppm, and a feature-shape Pearson correlation ≥ 0.85 with at least five data points. Finally, collapse ion-identity nodes in the IIMN result and tabulate nodes and edges before/after collapsing; the rationale is that IIN edges merge redundant, disconnected sub-networks of different ion species (adducts, isotopes) of the same molecule while retaining MS2 similarity edges.

## Related tools

- **MZmine** (Feature finding, aligned feature list creation, MS1-based ion identity annotation, and export of the .csv/.mgf/IIN edge files consumed by FBMN) — https://github.com/mzmine/mzmine
- **GNPS** (Web server that runs the FBMN/IIMN workflow on the uploaded feature table, MGF, and supplementary edges) — https://github.com/CCMS-UCSD/GNPS_Workflows
- **XCMS** (Alternative feature-finding tool interfaced with IIMN; xcms-gnps-tools provides formatSpectraForGNPS() and getFeatureAnnotations() to produce FBMN/IIN inputs) — https://github.com/jorainer/xcms-gnps-tools
- **MS-DIAL** (Alternative open-source feature-finding tool interfaced with the IIMN workflow)

## Evaluation signals

- Both exported IIN edge files and the .mgf parse without errors and every IIN edge's ID1/ID2 refer to feature IDs present in the quantification table
- IIMN run merges previously disconnected sub-networks: FBMN-only established only one edge between free and conjugated bile-acid subnetworks, whereas IIMN produces one large network
- Collapsing ion-identity nodes reduces total node count while retaining MS2 similarity edges, increasing edge density among nodes annotated to the same compound class
- IIN edge formation respects thresholds: RT tolerance 0.1 min, 0.001 m/z / 10 ppm, feature-shape Pearson correlation ≥ 0.85 with ≥ 5 data points (two on each edge)
- GNPS workflow job completes with library annotations meeting the minimum cosine similarity of 0.9 and 4–6 shared fragment ions where present

## Limitations

- Specific MZmine/MS-DIAL/XCMS versions and per-dataset FBMN parameters were not pinned in the text (variable parameters in Supplementary Table 4), so exact reproduction requires supplementary material
- The MZmine IIMN build is described as a 'special IIMN build' — availability in a released MZmine version is unclear
- IIN supplementary edge files follow a generic CSV style (columns ID1, ID2) with no formal schema or validator tool

## Evidence

- [discussion] Export the feature list as a feature quantification table (.csv), an MS2 spectral summary file (.mgf), and a supplementary edges files (IIN files, .csv): [evidence span withheld — anchor status: not_found]
- [discussion] RT and correlation thresholds for IIN grouping: "A pair of features were grouped with a retention time tolerance of 0.1 min"
- [discussion] Feature shape Pearson correlation threshold: "a feature shape Pearson correlation greater equals 0.85 with at least five data points and two data points on each edge"
- [abstract] Redundant disconnected sub-networks motivating IIMN: "the fragmentation spectra of these ion species often remain unconnected in tandem mass spectrometry-based molecular networks, leading to redundant and disconnected sub-networks"
- [intro] Two layers of connectivity in the resulting network: "The resulting ion identity molecular networks contain two layers of feature (node) connectivity, linking ion identities of the same compound by MS1 characteristics and structurally similar compounds"
- [readme] FBMN workflow entry point on GNPS: [evidence span withheld — anchor status: external_doc]
