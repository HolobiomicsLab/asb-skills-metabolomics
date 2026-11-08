---
name: fluorinated-compound-annotation
description: Use when processing LC-HRMS/MS feature tables from AFFF or environmental/biological
  samples to discover known, uncommon, and unknown PFAS, especially fluorine-containing
  precursors lacking exact mass matches.
when_to_use_negative:
- Precursors already have exact mass matches to catalogued PFAS — use exact mass matching
  rather than reverse fragment screening
- Data are not LC-HRMS/MS feature tables from mass spectrometry (e.g. non-MS fluorine
  assays)
- Analysis goal is general metabolomics or lipidomics — FluoroMatch defaults target
  PFAS unless Lipid <- TRUE is explicitly set
edam_operation: http://edamontology.org/operation_3212
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: FluoroMatch 2.0
  role: Primary software automating the full non-targeted PFAS annotation workflow,
    including reverse fragment screening
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Modular
  role: Modular R implementation used to run the fragment-screening module on the
    blank-filtered feature table
  repo: github.com/InnovativeOmics/Core-Match
- name: FluoroMatch Flow 2.0
  role: End-to-end workflow version covering all PFAS data-processing steps from feature
    detection to annotation
  repo: github.com/InnovativeOmics/Core-Match
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s00216-021-03392-7
    title: FluoroMatch 2.0—making automated and comprehensive non-targeted PFAS annotation
      a reality
  source_content_hash: sha256:9aeb2d74ad89536d78264cae9ab7bd0271882253e3ca0ebf5904cccc129a6827
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - FluoroMatch 2.0
  - FluoroMatch Flow 2.0
  - FluoroMatch Modular
  edam_operation: http://edamontology.org/operation_3212
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# fluorinated-compound-annotation

## Summary

Automated non-targeted PFAS annotation using FluoroMatch 2.0 (LC-HRMS/MS), which chains feature detection, blank filtering, exact mass matching to the EPA PFAS list, mass defect filtering, homologous series detection, fragment screening, and reverse fragment screening into a single scored annotation workflow.

## When to use

Use when processing LC-HRMS/MS feature tables from AFFF or environmental/biological samples to discover known, uncommon, and unknown PFAS, especially fluorine-containing precursors lacking exact mass matches.

## When NOT to use

- Precursors already have exact mass matches to catalogued PFAS — use exact mass matching rather than reverse fragment screening
- Data are not LC-HRMS/MS feature tables from mass spectrometry (e.g. non-MS fluorine assays)
- Analysis goal is general metabolomics or lipidomics — FluoroMatch defaults target PFAS unless Lipid <- TRUE is explicitly set

## Inputs

- Blank-filtered feature table from LC-HRMS/MS feature detection (e.g. fragment_assignments pipeline input)
- EPA CompTox Chemicals Dashboard PFAS structure list (8000+ structures)
- Reconstructed library of 777 fluorine-containing PFAS fragments
- Experimental MS/MS spectra of precursor ions

## Outputs

- Per-feature fragment/substructure assignment table (fragment_assignments.csv)
- Annotated list of likely PFAS with confidence-level scores
- Flags for potential unknown PFAS classes (e.g. pentafluorosulfide + sulfonic acid terminus candidates)

## How to apply

Load the blank-filtered feature table from upstream feature detection into FluoroMatch 2.0 (FluoroMatch Modular or Flow) and exact mass match against the EPA CompTox Dashboard PFAS structure list (8000+ structures). Apply mass defect filtering in the PFAS window (mass defect −0.116 to 0.268 covering 98% of the EPA list) and m/z 117–1189, then detect homologous series and retention time patterns and run class-based MS/MS and fragment screening. For precursors with no exact mass match, run reverse fragment screening against a reconstructed library of 777 fluorine-containing PFAS fragments to assign substructures and flag potential PFAS, including candidates bearing diagnostic [SF5]− and [SO3F]− fragments. Integrate all evidence into FluoroMatch's confidence-level scoring framework to avoid overreporting before final annotation.

## Related tools

- **FluoroMatch 2.0** (Primary software automating the full non-targeted PFAS annotation workflow, including reverse fragment screening) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Modular** (Modular R implementation used to run the fragment-screening module on the blank-filtered feature table) — github.com/InnovativeOmics/Core-Match
- **FluoroMatch Flow 2.0** (End-to-end workflow version covering all PFAS data-processing steps from feature detection to annotation) — github.com/InnovativeOmics/Core-Match

## Evaluation signals

- fragment_assignments.csv exists and parses as a well-formed table
- The C8 AFFF candidate row contains [SF5]− and [SO3F]− fragment assignments (chemical plausibility deferred to expert review)
- A large majority of non-PFAS features are removed — the paper reports filtering out 96% of features likely not PFAS
- Coverage of likely PFAS is broadly consistent with the reported ~10x increase over the prior FluoroMatch release
- Annotated precursors fall within the PFAS mass defect window (−0.116 to 0.268) and m/z 117–1189

## Limitations

- Chemical plausibility of novel proposed PFAS classes (e.g. the pentafluorosulfide/sulfonic-acid candidate) is left to expert review; annotations are scored, not confirmed
- Comprehensive PFAS detection is challenging because of their vast structural diversity and trace-level occurrence, so infrequently measured PFAS may still be missed
- Reverse fragment screening flags potential PFAS only when diagnostic fluorine-containing fragments are matched; absence of a matched fragment does not rule out a PFAS

## Evidence
