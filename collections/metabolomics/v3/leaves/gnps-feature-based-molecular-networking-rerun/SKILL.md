---
name: gnps-feature-based-molecular-networking-rerun
description: Use when a downstream prioritization step (e.g. Inventa FC score) needs
  a regenerated GNPS FBMN job — node/edge/IIN counts and cleaned spectral annotations
  — from a MZmine quant table and metadata file for an untargeted LC-HRMS/MS extract
  set.
when_to_use_negative:
- Input is already a cleaned annotation table — rerunning FBMN would duplicate work;
  go straight to Inventa's FC computation.
- No MZmine quantification table is available (e.g. data processed only in another
  tool) — FBMN expects the MZmine output format with 'Peak area', 'row m/z' and 'row
  retention time' columns.
- The analysis needs RT-aware sample comparison rather than feature annotation/network
  statistics — use the MEMO metric instead, which is explicitly RT-agnostic and compares
  MS2 fingerprints.
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS
  role: Runs the feature-based molecular networking job that produces the ion identity
    network and spectral annotations from the MZmine quant table and metadata
  repo: https://www.frontiersin.org/articles/10.3389/fmolb.2022.1028334/full
- name: MZmine2
  role: Source of the feature quantitative table consumed by the GNPS FBMN job
  repo: http://mzmine.github.io/
- name: MZmine3
  role: Alternative source of the feature quantitative table (data_process_origin
    parameter selects 'MZMine2' or 'MZmine3')
  repo: http://mzmine.github.io/
- name: SIRIUS
  role: Optional in-silico annotation of features, filtered with min_ZodiacScore =
    0.9 and min_ConfidenceScore = 0.0
  repo: https://bio.informatik.uni-jena.de/software/sirius/
- name: timaR
  role: Provides taxonomically reponderated in-silico annotation results merged into
    the cleaned annotation set
  repo: https://taxonomicallyinformedannotation.github.io/tima-r/index.html
- name: Inventa
  role: Downstream consumer of the FBMN outputs — FC score uses the ratio of specific
    non-annotated features over total features per extract
  repo: https://github.com/luigiquiros/inventa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fmolb.2022.1028334
    title: 'Inventa: A computational tool to discover structural novelty in natural
      extracts libraries'
  source_content_hash: sha256:dfee10fa09511f5bf531324f33a56649bf74b07a1180cf84fc82e252dc902d60
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - Inventa
  - MZmine2
  - MZmine3
  - SIRIUS
  - timaR
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# GNPS Feature Based Molecular Networking rerun

## Summary

Re-run the GNPS Feature Based Molecular Networking (FBMN) job on a MZmine2/MZmine3 feature quantification table plus GNPS-format metadata to reproduce the ion identity network (nodes, IIN groups, edges, singletons) and library/in-silico spectral annotations needed downstream by Inventa's Feature Component scoring.

## When to use

Use when a downstream prioritization step (e.g. Inventa FC score) needs a regenerated GNPS FBMN job — node/edge/IIN counts and cleaned spectral annotations — from a MZmine quant table and metadata file for an untargeted LC-HRMS/MS extract set.

## When NOT to use

- Input is already a cleaned annotation table — rerunning FBMN would duplicate work; go straight to Inventa's FC computation.
- No MZmine quantification table is available (e.g. data processed only in another tool) — FBMN expects the MZmine output format with 'Peak area', 'row m/z' and 'row retention time' columns.
- The analysis needs RT-aware sample comparison rather than feature annotation/network statistics — use the MEMO metric instead, which is explicitly RT-agnostic and compares MS2 fingerprints.

## Inputs

- GNPS-format metadata table with mandatory ATTRIBUTE_Species and ATTRIBUTE_Organe headers
- MZmine2 or MZmine3 feature quantification table (Peak area, row m/z, row retention time columns; Ion Identity flag)
- GNPS job identifier
- Optional: SIRIUS/ISDB annotation files and timaR-weighted in-silico annotation output
- Optional: Sirius CANOPUS recomputed project space (canopus_npc_summary_filename)

## Outputs

- GNPS FBMN ion identity network (nodes, edges, IIN groups, singletons)
- Library spectral match annotations (raw count and post-cleanup count)
- Cleaned annotation TSV (e.g. 2,494 annotated nodes filtered to 1,751 valid at ppm_error=5, shared_peaks=10, cosine=0.7)
- Summary statistics TSV (feature count, IIN groups, nodes, singletons, edges, annotation rates)

## How to apply

Assemble the three mandatory inputs: the GNPS-format metadata table (mandatory headers ATTRIBUTE_Species, cleaned to Open Tree of Life names, and ATTRIBUTE_Organe), the MZmine2/MZmine3 quantitative table restricted to the 'Peak area', 'row m/z' and 'row retention time' columns (specify whether Ion Identity networking was used), and the GNPS job identifier. Launch the GNPS feature-based molecular networking job to generate the ion identity network and spectral annotation results — for the Celastraceae UHPLC-HRMS/MS proof-of-concept set this yielded 16,139 features grouped into 14,554 IIN groups, 16,139 nodes (5,922 singletons) and 22,656 edges. Clean the raw library annotations by filtering with ppm_error = 5, shared_peaks = 10, cosine = 0.7 and ionisation_mode = 'pos', which reduced 2,494 library-annotated nodes to 1,751 valid ones. Optionally supplement with SIRIUS/ISDB in-silico annotations using min_ZodiacScore = 0.9 and min_ConfidenceScore = 0.0 cut-offs to raise the combined annotation rate to approximately 68%. Write the summary statistics (feature, IIN group, node, singleton, edge and annotation counts) to a TSV for downstream use.

## Related tools

- **GNPS** (Runs the feature-based molecular networking job that produces the ion identity network and spectral annotations from the MZmine quant table and metadata) — https://www.frontiersin.org/articles/10.3389/fmolb.2022.1028334/full
- **MZmine2** (Source of the feature quantitative table consumed by the GNPS FBMN job) — http://mzmine.github.io/
- **MZmine3** (Alternative source of the feature quantitative table (data_process_origin parameter selects 'MZMine2' or 'MZmine3')) — http://mzmine.github.io/
- **SIRIUS** (Optional in-silico annotation of features, filtered with min_ZodiacScore = 0.9 and min_ConfidenceScore = 0.0) — https://bio.informatik.uni-jena.de/software/sirius/
- **timaR** (Provides taxonomically reponderated in-silico annotation results merged into the cleaned annotation set) — https://taxonomicallyinformedannotation.github.io/tima-r/index.html
- **Inventa** (Downstream consumer of the FBMN outputs — FC score uses the ratio of specific non-annotated features over total features per extract) — https://github.com/luigiquiros/inventa

## Examples

```
conda env create -n inventa -f inventa/environment.yml; conda activate inventa; jupyter notebook notebook/inventa.ipynb
```

## Evaluation signals

- Reproduced FBMN node count equals the reference (16,139 nodes for the Celastraceae set), with 5,922 singletons and 22,656 edges
- Reproduced feature count equals 16,139 features grouped into 14,554 IIN groups
- Library annotation count before cleanup is 2,494 nodes; after applying ppm_error=5, shared_peaks=10, cosine=0.7 and ionisation_mode='pos' it drops to 1,751 valid annotations
- Combined library + in-silico annotation rate is approximately 68% across the feature set
- Summary TSV contains all required columns (features, IIN groups, nodes, singletons, edges, annotations before/after cleanup, annotation rate)

## Limitations

- Ion Identity handling must be explicitly specified ('data_process_origin = MZMine3' / Ion Identity flag) or the IIN group counts will not match
- If MZmine exported columns other than 'Peak area', 'row m/z' and 'row retention time' (e.g. identities), they must be removed manually or handled in quant_table() — only one of 'Peak height' or 'Peak area' is considered at a time
- MZmine2 and MZmine3 produce different table formats; data_process_origin must be set accordingly in src/process_data.py
- Binder instances shut down after 15 minutes of inactivity, so parameters and results must be saved locally

## Evidence

- [results] 16,139 nodes (5,922 singletons) and 22,656 edges; 2,494 nodes were library-annotated, with 1,751 nodes valid after cleaning: [evidence span withheld — anchor status: not_found]
- [methods] ppm_error = 5, shared_peaks = 10, cosine = 0.7, ionisation_mode = 'pos': "ppm_error = 5                     # min error in ppm to consider an annotation valable"
- [readme] quantitative_data_filename: MZmine output format using only the 'Peak area', 'row m/z', and 'row retention time' columns: "`quantitative_data_filename`: MZmine output format using only the 'Peak area', 'row m/z', and 'row retention time' columns."
- [methods] Inventa takes input directly from MZmine2 or MZmine 3: "Inventa takes input directly from MZmine2 or [MZmine 3](http://mzmine.github.io/)"
