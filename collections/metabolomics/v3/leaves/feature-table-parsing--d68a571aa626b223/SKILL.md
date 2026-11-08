---
name: feature-table-parsing
description: Use when you have an MZmine3 feature list with representative spectra
  from positive ion mode LC–MS/MS data and need molecular formula and adduct assignment
  per feature, or need to reproduce a reported formula/adduct hit rate.
when_to_use_negative:
- The input is already a SIRIUS-annotated feature table with formula/adduct assignments
  — re-running assignment is unnecessary
- The data are negative ion mode only — this skill is scoped to positive ion mode,
  where SIRIUS considers common positive-ion adducts
- The goal is molecular candidate ranking or structure retrieval rather than formula/adduct
  assignment — use the MVP candidate-ranking workflow instead
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: MZmine3
  role: Upstream tool whose exported feature table (feature list with representative
    spectra) is the input to parse and assign
- name: SIRIUS v6.1.0
  role: Performs molecular formula and adduct assignment on each feature's representative
    spectrum under default settings with common positive-ion adducts
- name: MVP
  role: Framework in whose meat metabolomics replication workflow this feature-table
    parsing and SIRIUS assignment step is embedded
  repo: github.com/HassounLab/MVP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.11.12.688047
    title: 'Learning from All Views: A Multiview Contrastive Framework for Metabolite
      Annotation'
  source_content_hash: sha256:960d41ac92820ef35009b523e7c3c452306502449f2699ab6efea2c57aed99a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MVP
  - MZmine3
  - SIRIUS v6.1.0
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# feature-table-parsing

## Summary

Parse an MZmine3 feature table with representative MS/MS spectra and run SIRIUS v6.1.0 formula and adduct assignment over positive-ion-mode LC–MS/MS features, compiling per-feature assignment records to compute a formula/adduct hit rate (866/900 in the replicated cultured-vs-conventional meat metabolomics study).

## When to use

Use when you have an MZmine3 feature list with representative spectra from positive ion mode LC–MS/MS data and need molecular formula and adduct assignment per feature, or need to reproduce a reported formula/adduct hit rate.

## When NOT to use

- The input is already a SIRIUS-annotated feature table with formula/adduct assignments — re-running assignment is unnecessary
- The data are negative ion mode only — this skill is scoped to positive ion mode, where SIRIUS considers common positive-ion adducts
- The goal is molecular candidate ranking or structure retrieval rather than formula/adduct assignment — use the MVP candidate-ranking workflow instead

## Inputs

- MZmine3-exported feature table containing feature list rows with representative MS/MS spectra (from positive ion mode LC–MS/MS mzML data)
- Positive ion mode LC–MS/MS data (mzML format, as publicly available on the Metabolomics Workbench for the meat study)

## Outputs

- Per-feature formula and adduct assignment table keyed by feature id (one record per processed feature row)
- Count of features with both a molecular formula and an adduct assignment (expected 866 of 900 features in the meat study replication)

## How to apply

Start from the MZmine3 feature table (feature list with representative spectra) exported by the upstream MZmine3 feature-extraction step; in the meat study this yielded 900 detected features. Launch SIRIUS v6.1.0 with default formula and adduct assignment settings, letting it consider common positive-ion adducts appropriate for positive ion mode data. Parse the SIRIUS output and compile per-feature formula/adduct assignment records into a single table keyed by feature id. Count features for which SIRIUS returned both a molecular formula and an adduct assignment — in the replicated workflow this count was 866 of 900 (a ~96% assignment rate), which is the validation target when reproducing the article's meat metabolomics section.

## Related tools

- **MZmine3** (Upstream tool whose exported feature table (feature list with representative spectra) is the input to parse and assign)
- **SIRIUS v6.1.0** (Performs molecular formula and adduct assignment on each feature's representative spectrum under default settings with common positive-ion adducts)
- **MVP** (Framework in whose meat metabolomics replication workflow this feature-table parsing and SIRIUS assignment step is embedded) — github.com/HassounLab/MVP

## Evaluation signals

- Every processed feature row in the compiled table has either a complete formula + adduct assignment record or an explicit unassigned flag — no silent drops
- Total detected feature count matches the MZmine3 output (900 features in the meat study)
- Count of features with both formula and adduct assignment equals the reported 866 for the meat metabolomics replication (~96% hit rate)
- All assigned adducts are consistent with positive ion mode (common positive-ion adducts considered by SIRIUS v6.1.0)

## Limitations

- The original cultured-vs-conventional meat study provided neither the processed spectra nor the mapping to identified metabolites, so this workflow is a replication rather than a direct comparison against source outputs
- Downstream annotations built on these assignments (e.g., MVP's putative metabolite annotations for the meat study) are putative and require experimental validation
- Partial results may not generalize: SIRIUS 3.5 results on MassSpecGym were only partially available, and formula-assignment-dependent peak labeling (as in MIST) assumes every peak is charged with the adduct

## Evidence

- [results] Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0: "Feature extraction and representative spectra selection are performed using MZmine3,38 followed by formula and adduct assignment with SIRIUS v6.1.0"
- [results] For this evaluation, we focus on the positive ion mode data: "For this evaluation, we focus on the positive ion mode data"
- [other] In the replicated meat metabolomics workflow, 900 features were detected with MZmine3, of which 866 received formula and adduct assignments from SIRIUS (v6.1.0).: [evidence span withheld — anchor status: not_found]
- [results] Since neither the processed spectra nor the mapping to the identified metabolites was provided, we replicated the study: "Since neither the processed spectra nor the mapping to the identified metabolites was provided, we replicated the study"
