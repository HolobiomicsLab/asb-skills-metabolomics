---
name: false-discovery-rate-qvalue-estimation
description: Use when rescored PSM scores exist and you must compute q-values from
  target/decoy counts, enforce a 1% training-FDR threshold, and filter lower-rank
  PSMs before final FDR calculation.
when_to_use_negative:
- Input PSMs have not yet been rescored — no rescored score distribution exists to
  compute q-values from
- No target/decoy labels or decoy pattern are available in the input identification
  file, making target-decoy FDR estimation impossible
- The user only needs raw search-engine scores without FDR control; q-value estimation
  adds post-processing overhead with no benefit
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MS2Rescore
  role: Filters lower-ranking PSMs before final FDR calculation and writes the per-PSM
    q-value output table
  repo: https://github.com/compomics/ms2rescore
- name: ristretto
  role: Trains the semi-supervised classifier (svm or lda) and reports scores, q-values,
    and PEPs from the target/decoy-labelled feature table
  repo: https://github.com/CompOmics/ristretto
- name: Percolator
  role: Reference implementation of the semi-supervised PSM rescoring and FDR estimation
    procedure that ristretto reimplements
- name: Mumble
  role: Generates decoy-site PSM candidates for site-localisation models and produces
    candidate PSMs that must be filtered by FDR before output
  repo: https://github.com/compomics/mumble
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/pmic.202300336
    title: Machine learning‐based peptide‐spectrum match rescoring opens up the immunopeptidome
  source_content_hash: sha256:c052679c91874931cc203fa0aa2431eee0dbb4ceb0cf5bb6c1b8d96bfce4ba69
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compomics/ms2rescore
  repo_url: https://github.com/compomics/ms2rescore
  tools:
  - MS2Rescore
  - Mumble
  - Percolator
  - Ristretto
  - ristretto
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# false-discovery-rate-qvalue-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates per-PSM q-values and enforces target-decoy FDR control over a rescored peptide-spectrum match (PSM) score distribution, filtering lower-ranking PSMs before the final FDR calculation so that reported identifications at a 1% FDR threshold are trustworthy.

## When to use

Use when rescored PSM scores exist and you must compute q-values from target/decoy counts, enforce a 1% training-FDR threshold, and filter lower-rank PSMs before final FDR calculation.

## When NOT to use

- Input PSMs have not yet been rescored — no rescored score distribution exists to compute q-values from
- No target/decoy labels or decoy pattern are available in the input identification file, making target-decoy FDR estimation impossible
- The user only needs raw search-engine scores without FDR control; q-value estimation adds post-processing overhead with no benefit

## Inputs

- Table of peptide-spectrum matches (PSMs) with rescoring features and a target/decoy label (e.g. identified via id_decoy_pattern regex)
- Spectrum identifier column linking each PSM to its original spectrum (configurable spectrum_id_pattern / psm_id_pattern regexes)
- Configuration parameters: train_fdr (default 0.01), max_psm_rank_output (default 1), max_psm_rank_input

## Outputs

- Per-PSM table of rescored scores, q-values, and posterior error probabilities (PEPs) at PSM, peptide, and protein level
- Filtered PSM set retaining only PSMs up to max_psm_rank_output before final FDR calculation (default: top-ranked only)

## How to apply

Split input PSMs into positives (top-ranked target PSMs passing the 1% train_fdr threshold) and negatives (decoy PSMs identified via a configurable id_decoy_pattern regex), then train a semi-supervised classifier (Percolator-style svm via ristretto, or the faster lda) to rescore all matches. After rescoring, compute per-PSM q-values from the target-decoy counts over the rescored score distribution. Before the final FDR calculation, filter out lower-ranking PSMs — MS²Rescore respects max_psm_rank_output (default 1) so a correct FDR control is achieved; removing lower-rank matches first is mandatory because multi-rank rescoring can promote lower-ranking candidates to top-ranked, which would otherwise inflate reported q-values. Validation requires all q-values to lie in [0, 1] and decoy-derived FDR control at the reported 0.01 train_fdr threshold.

## Related tools

- **MS2Rescore** (Filters lower-ranking PSMs before final FDR calculation and writes the per-PSM q-value output table) — https://github.com/compomics/ms2rescore
- **ristretto** (Trains the semi-supervised classifier (svm or lda) and reports scores, q-values, and PEPs from the target/decoy-labelled feature table) — https://github.com/CompOmics/ristretto
- **Percolator** (Reference implementation of the semi-supervised PSM rescoring and FDR estimation procedure that ristretto reimplements)
- **Mumble** (Generates decoy-site PSM candidates for site-localisation models and produces candidate PSMs that must be filtered by FDR before output) — https://github.com/compomics/mumble

## Examples

```
ms2rescore -i psms.tsv --modification-annotation-labels 'sample/blank > 3' --ristretto-config '{"train_fdr": 0.01, "model": "svm", "max_psm_rank_output": 1}'
```

## Evaluation signals

- Every input PSM receives a finite rescored score
- All computed q-values lie within [0, 1]
- Decoy-derived FDR control at the reported 0.01 train_fdr threshold is respected
- Lower-ranking PSMs above max_psm_rank_output (default 1) are removed before the final FDR calculation and output writing
- Reported peptide ID count at a given FDR threshold is higher than the unrescored baseline, or the same count at a more stringent FDR threshold

## Limitations

- Mumble is under active development and should be considered beta software; occasional errors can still occur, especially on unusual input files or uncommon modifications, and results should be reviewed before downstream use
- MS²Rescore filters out lower-ranking PSMs before final FDR calculation to ensure correct FDR control — if this filter is bypassed, multi-rank rescoring can promote lower-ranking candidates and inflate FDR
- Mumble-generated candidate PSMs whose matched_ions_pct drops more than a defined threshold are removed, so candidates with a large drop in matched fragment ion fraction cannot contribute to FDR estimates

## Evidence

- [intro] Filter lower-ranking PSMs before final FDR calculation and output writing: "To ensure a correct FDR control after rescoring, MS²Rescore filters out lower-ranking PSMs before"
- [intro] Semi-supervised rescoring with configurable training FDR and model choice: [evidence span withheld — anchor status: not_found]
- [readme] Ristretto trains a classifier and reports scores, q-values, and PEPs: [evidence span withheld — anchor status: external_doc]
- [methods] Decoy PSMs are identified via a configurable regex pattern: "Regex pattern used to identify the decoy PSMs in identification file"
- [methods] max_psm_rank_output parameter controls rank filtering before final FDR calculation: "Maximum rank of PSMs to return after rescoring, before final FDR calculation"
