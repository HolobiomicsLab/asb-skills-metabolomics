---
name: extracted-ion-chromatogram-targeted-analysis
description: Use when you must verify how many distinct isomers co-elute under a single
  precursor m/z, confirm cluster membership of specific isomers (e.g., 2 Da-shifted
  type E spectra), or validate an annotation such as CCMSLIB00000840365 for m/z 607.32.
when_to_use_negative:
- When the goal is untargeted bioactivity ranking of many ions (e.g., BCC scoring
  of clean consensus spectra), not verification of a single targeted m/z
- When the input is raw vendor LC–MS/MS data that has not yet been preprocessed by
  NP3 MS Workflow (steps 2–5) — no per-sample MGF or cluster tables exist yet
- When isomers are chromatographically unresolvable in the chosen retention window,
  peak counting will not enumerate them reliably
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: NP3 MS Workflow
  role: Produces the per-sample MGF files, cluster/quantification tables, and clean
    consensus spectra from which the target-m/z ion chromatogram is extracted
  repo: https://github.com/danielatrivella/NP3_MS_Workflow
- name: Cytoscape
  role: Visualizes the filtered SSMN to confirm that target isomers (IX and XI) fall
    in distinct spectral clusters
- name: GNPS (Global Natural Products Social Molecular Networking)
  role: Spectral library used to annotate the target m/z ions, including 4-deoxyphorbol
    ester references CCMSLIB00000840551/552 and the m/z 607.32 entry CCMSLIB00000840365
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c05829
    title: 'NP<sup>3</sup> MS Workflow: An Open-Source Software System to Empower
      Natural Product-Based Drug Discovery Using Untargeted Metabolomics'
  source_content_hash: sha256:960709a9bd9be8de219578a0e0b8990f1165bc04824f7ee296abd618a5b6f483
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/danielatrivella/NP3_MS_Workflow
  repo_url: https://github.com/danielatrivella/NP3_MS_Workflow
  tools:
  - Cytoscape
  - GNPS (Global Natural Products Social Molecular Networking)
  - Mascot Generic Format (MGF)
  - NP3 MS Workflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# extracted-ion-chromatogram-targeted-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Targeted interrogation of a specific precursor ion (e.g., m/z 589.31 or 607.32) in LC–MS/MS metabolomics data by extracting its ion chromatogram within a defined retention window, counting distinct isomer peaks, and cross-checking spectral-cluster membership and GNPS annotations.

## When to use

Use when you must verify how many distinct isomers co-elute under a single precursor m/z, confirm cluster membership of specific isomers (e.g., 2 Da-shifted type E spectra), or validate an annotation such as CCMSLIB00000840365 for m/z 607.32.

## When NOT to use

- When the goal is untargeted bioactivity ranking of many ions (e.g., BCC scoring of clean consensus spectra), not verification of a single targeted m/z
- When the input is raw vendor LC–MS/MS data that has not yet been preprocessed by NP3 MS Workflow (steps 2–5) — no per-sample MGF or cluster tables exist yet
- When isomers are chromatographically unresolvable in the chosen retention window, peak counting will not enumerate them reliably

## Inputs

- Per-sample MGF files and quantification/cluster tables from NP3 MS Workflow preprocessing of MSV000080502 (Euphorbia dendroides LC–MS/MS, 16 samples)
- Target precursor m/z value and retention window (e.g., m/z 589.31, 1400–1800 s)
- GNPS library annotation results for clean consensus spectra (e.g., CCMSLIB00000840365)
- Filtered SSMN network file for Cytoscape visualization

## Outputs

- Isomer analysis table (e.g., mz58931_isomer_analysis.csv) listing distinct isomer peaks, spectral-cluster membership, NP3-exclusive-feature percentage, and GNPS annotations
- Extracted ion chromatogram traces for the target m/z over the specified retention window
- Cytoscape visualization of the filtered SSMN showing target-cluster separation

## How to apply

After running the NP3 MS Workflow preprocessing (step 2) and clustering/quantification/cleaning (steps 3–5, using the NP3-shifted cosine function), extract the ion chromatogram of the target m/z over a restricted retention window — for the Euphorbia dendroides dataset MSV000080502, m/z 589.31 is extracted over 1400–1800 s — and count the distinct chromatographic peaks to enumerate isomers. For each peak, check its clean consensus spectrum's cluster membership (e.g., isomers IX and XI should fall in distinct clusters corresponding to 2 Da-shifted spectra type E) and its library annotation against the GNPS spectral database (e.g., the m/z 607.32 clean consensus spectrum to CCMSLIB00000840365, a 4-deoxyphorbol ester analogue). Compute the percentage of NP3-exclusive features within the target spectral cluster (39% reported for m/z 589.31) and compile isomer list, cluster membership, exclusive-feature percentage, and annotations into a table (mz58931_isomer_analysis.csv). Open the filtered SSMN in Cytoscape for visual confirmation of cluster separation.

## Related tools

- **NP3 MS Workflow** (Produces the per-sample MGF files, cluster/quantification tables, and clean consensus spectra from which the target-m/z ion chromatogram is extracted) — https://github.com/danielatrivella/NP3_MS_Workflow
- **Cytoscape** (Visualizes the filtered SSMN to confirm that target isomers (IX and XI) fall in distinct spectral clusters)
- **GNPS (Global Natural Products Social Molecular Networking)** (Spectral library used to annotate the target m/z ions, including 4-deoxyphorbol ester references CCMSLIB00000840551/552 and the m/z 607.32 entry CCMSLIB00000840365)

## Evaluation signals

- The results table contains exactly 11 distinct m/z 589.31 isomer peaks within the 1400–1800 s retention window
- The m/z 607.32 entry is annotated to GNPS entry CCMSLIB00000840365
- Isomers IX and XI map to distinct spectral clusters consistent with the 2 Da-shifted spectra type E
- The NP3-exclusive-feature percentage within the m/z 589.31 spectral cluster is consistent with the reported 39%
- Visual inspection of the filtered SSMN in Cytoscape confirms the target isomers' cluster separation

## Limitations

- Isomer counting depends on chromatographic resolution in the selected retention window; unresolved co-eluting isomers will be undercounted
- Annotation quality is bounded by the coverage of the GNPS spectral database; ions without deposited references (e.g., 4-deoxyphorbol ester entries CCMSLIB00000840551/552) cannot be confirmed
- Exclusive-feature percentages are specific to the processed dataset and NP3 parameters (mirroring the original GNPS processing parameters); they are not transferable across datasets without reprocessing

## Evidence
