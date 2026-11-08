---
name: ion-identity-network-group-counting
description: Use when the analysis goal is to reproduce or verify the graph-level
  statistics of a GNPS FBMN job (feature count, IIN group count, node count, singleton
  count, edge count) from the deposited MZmine quant table and GNPS job ID.
when_to_use_negative:
- Input is already a cleaned, post-filtered annotation table rather than raw GNPS
  FBMN outputs — the counting logic depends on the raw node/edge/IIN structure.
- The goal is to prioritize extracts for novelty (that is Inventa's FC/LC/CC/SC ranking),
  not to reproduce network counts.
- Data are LC-MS/MS features without ion identity networking enabled in MZmine/GNPS,
  so IIN groups cannot be tallied.
edam_operation: http://edamontology.org/operation_3639
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Runs the Feature Based Molecular Networking job that produces the ion identity
    network, nodes, singletons, and edges being counted.
- name: MZmine2
  role: Produces the feature quantitative table ('Peak area', 'row m/z', 'row retention
    time') used as FBMN input.
- name: MZmine3
  role: Alternative producer of the feature quantitative table; Inventa takes input
    directly from MZmine2 or MZmine 3.
- name: SIRIUS
  role: Provides in-silico ISDB annotations optionally processed with min_ZodiacScore
    = 0.9 and min_ConfidenceScore = 0.0 to complement library matches.
- name: Inventa
  role: Consumes the same GNPS job, metadata, and quantitative table to compute downstream
    novelty scores; supplies the workflow context for the counting task.
  repo: https://github.com/luigiquiros/inventa
- name: timaR
  role: Supplies taxonomically weighted in-silico annotation files optionally used
    alongside the cleaned GNPS annotations.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:dc1fe2778bc52d1ec5cb8061663ab4069935769dc903207893e284f1cb3a8535
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/luigiquiros/inventa
  tools:
  - GNPS
  - Inventa
  - MZmine2
  - MZmine3
  - SIRIUS
  - timaR
  edam_operation: http://edamontology.org/operation_3639
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# ion-identity-network-group-counting

## Summary

Reproduces the node, edge, and ion-identity-network (IIN) group statistics reported for a GNPS Feature Based Molecular Network, by counting features, IIN groups, nodes, singletons, and edges from the deposited GNPS/MZmine inputs (e.g. 16,139 features, 14,554 IIN groups, 22,656 edges for the Celastraceae set).

## When to use

Use when the analysis goal is to reproduce or verify the graph-level statistics of a GNPS FBMN job (feature count, IIN group count, node count, singleton count, edge count) from the deposited MZmine quant table and GNPS job ID.

## When NOT to use

- Input is already a cleaned, post-filtered annotation table rather than raw GNPS FBMN outputs — the counting logic depends on the raw node/edge/IIN structure.
- The goal is to prioritize extracts for novelty (that is Inventa's FC/LC/CC/SC ranking), not to reproduce network counts.
- Data are LC-MS/MS features without ion identity networking enabled in MZmine/GNPS, so IIN groups cannot be tallied.

## Inputs

- GNPS FBMN job identifier
- Metadata table in GNPS format (e.g. with ATTRIBUTE_Species and ATTRIBUTE_Organe headers)
- MZmine2/MZmine3 feature quantitative table ('Peak area', 'row m/z', 'row retention time' columns)
- GNPS spectral match annotation results
- Optional: SIRIUS/ISDB annotation files and CANOPUS chemical taxonomy summary

## Outputs

- TSV summary of node, edge, feature, IIN group, singleton, and library-annotation counts before and after cleaning
- Overall combined annotation rate across the feature set

## How to apply

Obtain the deposited GNPS FBMN inputs: the metadata table in GNPS format and the MZmine2/MZmine3 quantitative table restricted to 'Peak area', 'row m/z', and 'row retention time' columns. Run the GNPS feature-based molecular networking job to generate the ion identity network and annotation results. Clean the GNPS spectral-match annotations with thresholds ppm_error = 5, shared_peaks = 10, cosine = 0.7, ionisation_mode = 'pos' to reduce library-annotated nodes (2,494 → 1,751 for the Celastraceae set); optionally complement with SIRIUS/ISDB results using min_ZodiacScore = 0.9 and min_ConfidenceScore = 0.0. Count features, IIN groups, nodes, singletons, and edges, compute the combined annotation rate (~68%), and write the summary to a TSV.

## Related tools

- **GNPS** (Runs the Feature Based Molecular Networking job that produces the ion identity network, nodes, singletons, and edges being counted.)
- **MZmine2** (Produces the feature quantitative table ('Peak area', 'row m/z', 'row retention time') used as FBMN input.)
- **MZmine3** (Alternative producer of the feature quantitative table; Inventa takes input directly from MZmine2 or MZmine 3.)
- **SIRIUS** (Provides in-silico ISDB annotations optionally processed with min_ZodiacScore = 0.9 and min_ConfidenceScore = 0.0 to complement library matches.)
- **Inventa** (Consumes the same GNPS job, metadata, and quantitative table to compute downstream novelty scores; supplies the workflow context for the counting task.) — https://github.com/luigiquiros/inventa
- **timaR** (Supplies taxonomically weighted in-silico annotation files optionally used alongside the cleaned GNPS annotations.)

## Examples

```
python src/process_data.py  # counts features, IIN groups, nodes, singletons, edges from the GNPS FBMN outputs; e.g. git clone https://github.com/luigiquiros/inventa.git then conda env create -n inventa -f inventa/environment.yml
```

## Evaluation signals

- Reproduced feature count matches the reference (16,139 features) for the Celastraceae set.
- Reproduced IIN group count matches the reference (14,554 IIN groups).
- Reproduced node and singleton counts match the references (16,139 nodes; 5,922 singletons).
- Reproduced edge count matches the reference (22,656 edges).
- Library-annotated nodes drop from 2,494 to 1,751 after applying ppm_error = 5, shared_peaks = 10, cosine = 0.7, ionisation_mode = 'pos', and the combined annotation rate is ~68%.

## Limitations

- Counts are only reproducible from the deposited GNPS job inputs; the reported statistics are dataset-specific and cannot be transferred to other extract sets without re-running the FBMN.
- The MZmine quantitative table must be limited to 'Peak area', 'row m/z', and 'row retention time' columns; 'Peak height' requires editing the quant_table() function and only one column type is considered at a time.
- Ion Identity Networking must be enabled in the MZmine/GNPS run for IIN group counts to be meaningful.

## Evidence

- [methods] GNPS FBMN job input files (metadata table in GNPS format, MZmine quantitative table with Peak area/row m/z/row retention time columns, and the GNPS job identifier): [evidence span withheld — anchor status: not_found]
- [methods] ppm_error = 5, shared_peaks = 10, cosine = 0.7, ionisation_mode = 'pos': [evidence span withheld — anchor status: not_found]
- [methods] 16,139 features, 14,554 IIN groups, 16,139 nodes, 5,922 singletons, 22,656 edges: [evidence span withheld — anchor status: not_found]
- [readme] quantitative_data_filename: MZmine output format using only the 'Peak area', 'row m/z', and 'row retention time' columns: "`quantitative_data_filename`: MZmine output format using only the 'Peak area', 'row m/z', and 'row retention time' columns."
- [abstract] Inventa calculates multiple scores that estimate the structural novelty potential of the natural extracts.: "Inventa calculates multiple scores that estimate the structural novelty potential of the natural extracts."
