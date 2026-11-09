---
name: retention-time-alignment-and-feature-linking
description: Use when multiple LC-MS/MS or DIA/SWATH maps must be made comparable
  — after feature detection and database searching, before protein inference — so
  peptide quantities refer to the same molecular feature in every run.
when_to_use_negative:
- Data has not yet been centroided and searched — alignment on raw profile spectra
  will link noise instead of peptide features
- A single-run experiment with no cross-run comparison — linking is unnecessary and
  can merge distinct co-eluting isobaric peptides
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: OpenMS
  role: Provides the retention time alignment and cross-run feature linking algorithms
    executed after feature detection and before protein inference
  repo: https://github.com/OpenMS/OpenMS
- name: OpenMS 2.0
  role: Version of OpenMS used to run the OpenSWATH RT alignment and feature linking
    chain in the twin-cohort plasma SWATH-MS workflow
- name: Python
  role: Used downstream for protein inference and per-protein variance component modelling
    after linked peptide quantities are exported
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1038/nmeth.3959
    title: 'OpenMS: a flexible open-source software platform for mass spectrometry
      data analysis'
  source_content_hash: sha256:5ff0f78be98fdc7f390160fbd28ac7397613595e900c4c761c8f76a6da90cdf2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/OpenMS/OpenMS
  tools:
  - OpenMS
  - OpenMS 2.0
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Retention Time Alignment and Feature Linking

## Summary

Align LC-MS/MS retention times across acquisition runs with OpenMS and link the same peptide features between maps so quantities are comparable run-to-run, as used in the OpenSWATH plasma twin-cohort quantification and the GENCODE proteogenomics re-annotation workflows.

## When to use

Use when multiple LC-MS/MS or DIA/SWATH maps must be made comparable — after feature detection and database searching, before protein inference — so peptide quantities refer to the same molecular feature in every run.

## When NOT to use

- Data has not yet been centroided and searched — alignment on raw profile spectra will link noise instead of peptide features
- A single-run experiment with no cross-run comparison — linking is unnecessary and can merge distinct co-eluting isobaric peptides

## Inputs

- Processed LC-MS/MS feature maps (mzML / featureXML) from OpenMS feature detection
- Assay / transition library for OpenSWATH targeted extraction (TSV or TraML)
- Decoy-based scoring output per run (for FDR downstream)
- Multiple maps from the same cohort (e.g. twin-pair plasma SWATH-MS runs)

## Outputs

- RT-aligned feature maps (featureXML) with consistent retention times
- Consensus / linked feature table mapping each peptide to per-run quantities
- Per-peptide abundance matrix ready for protein inference and FDR control

## How to apply

After per-run file conversion, centroiding, database searching, and feature detection with OpenMS, run retention-time alignment to warp each map's RT axis onto a reference (a master map or assay-library-driven alignment for DIA). Then apply cross-run feature linking (e.g. FeatureLinkerUnlabeledQT / OpenSWATH mProphet-style consensus) to group chromatographic features that carry the same precursor/transition signal in every run, producing a consistent peptide quantity per run. Rationale: RT drift and run-to-run differences otherwise make the same peptide appear as unrelated features, breaking downstream protein inference and statistical error control. In the Liu et al. twin-cohort workflow, OpenSWATH targeted extraction of peptide transitions is followed by RT alignment and feature linking across all twin-pair maps before aggregation and FDR control; in the Wright et al. proteogenomics workflow the same alignment/linking step precedes protein inference over the >55 million public tandem spectra. Decision point: if an assay/transition library exists (OpenSWATH), align via transitions; for label-free DDA, use feature-based (unlabeled) linking with quality thresholds to reject low-confidence links.

## Related tools

- **OpenMS** (Provides the retention time alignment and cross-run feature linking algorithms executed after feature detection and before protein inference) — https://github.com/OpenMS/OpenMS
- **OpenMS 2.0** (Version of OpenMS used to run the OpenSWATH RT alignment and feature linking chain in the twin-cohort plasma SWATH-MS workflow)
- **Python** (Used downstream for protein inference and per-protein variance component modelling after linked peptide quantities are exported)

## Evaluation signals

- The three downstream variance components (heritable, environmental, longitudinal) sum to approximately 1 per protein in variance_components.csv
- All workflow steps from conversion through error control complete without error before linking results are accepted
- Exported peptide tables reproduce the reported set of novel protein-coding gene annotations relative to the GENCODE reference annotation
- Per-run peptide identifications are consistent across twin-pair maps after alignment (same peptide maps to the same linked feature group)

## Limitations

- RT alignment and linking depend on quality of upstream centroiding, database searching and feature detection; errors propagate silently
- For proteogenomics, accept results only if the exported annotations match the reference annotation; residual discrepancies must be resolved by expert review
- Variance decomposition residuals beyond expected partitioning are resolved by expert review rather than automated correction

## Evidence

- [other] RT alignment before linking is part of the twin-cohort workflow: [evidence span withheld — anchor status: not_found]
- [other] Linking across runs precedes protein inference: [evidence span withheld — anchor status: not_found]
- [readme] OpenMS supports targeted and DIA workflows: [evidence span withheld — anchor status: external_doc]
