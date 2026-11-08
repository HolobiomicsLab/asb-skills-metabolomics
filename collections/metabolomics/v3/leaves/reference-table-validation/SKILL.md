---
name: reference-table-validation
description: Use when a paper claims a ranking result (e.g., one method placing 6
  unreported compounds in the top 10 of a 95-compound library) and you must verify
  it by re-deriving the rank table from primary tool outputs.
when_to_use_negative:
- Input is raw MS/MS data requiring annotation in the first place — this skill validates
  published tables and assumes per-compound annotation outputs already exist
- You need to prioritize unknowns in a new sample rather than benchmark/reproduce
  a published rank comparison (use MS2DECIDE K_estimation directly instead)
- The published table lacks spot-checkable reference values (counts, ranks, thresholds)
  to diff against — validation would be unfalsifiable
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: MS2DECIDE
  role: Source of the knownness score k outputs being validated against the published
    Table 4
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Baseline ranks recomputed under GNPS O (q(6,0.02), 1e-4 threshold) and default
    cosine 0.7 cutoff; reference value 19/89 known matches
- name: Sirius
  role: Baseline ranks recomputed from the Sirius 6 confidence (approximate) score
- name: ISDB-LOTUS
  role: Baseline ranks recomputed from the reported ISDB-LOTUS cosine values
- name: FERMO
  role: Baseline ranks recomputed from the FERMO novelty score
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:94be1dfbdd24b989943348ab00bcc0c9b9496f1f1f7c46ec2c9daa55f18817fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - FERMO
  - GNPS
  - ISDB-LOTUS
  - MS2DECIDE
  - Sirius
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# reference-table-validation

## Summary

Reproduces a published benchmark comparison table (e.g., Table 4 of the MS2DECIDE paper) by recomputing per-method ranks from raw annotation outputs and diff-checking regenerated values against the article's reference numbers before trusting downstream claims.

## When to use

Use when a paper claims a ranking result (e.g., one method placing 6 unreported compounds in the top 10 of a 95-compound library) and you must verify it by re-deriving the rank table from primary tool outputs.

## When NOT to use

- Input is raw MS/MS data requiring annotation in the first place — this skill validates published tables and assumes per-compound annotation outputs already exist
- You need to prioritize unknowns in a new sample rather than benchmark/reproduce a published rank comparison (use MS2DECIDE K_estimation directly instead)
- The published table lacks spot-checkable reference values (counts, ranks, thresholds) to diff against — validation would be unfalsifiable

## Inputs

- Per-compound annotation outputs for the 95-NP chemodiverse library from the MejriY/MS2DECIDE repository (GNPS, Sirius, ISDB-LOTUS results)
- Quantitative feature table (.csv, MZmine FBMN export) and tandem MS spectra (.mgf)
- Published reference table (article Table 4) with per-method ranks and spot-check values
- Baseline method configurations: q(6,0.02) filtering with 1e-4 threshold (GNPS O), cosine 0.7 cutoff (default GNPS), Sirius confidence score, ISDB-LOTUS cosine, FERMO novelty score

## Outputs

- table_4.csv: regenerated rank table for all evaluated methods (GNPS O, default GNPS 0.7, GNPS I, Sirius, ISDB-LOTUS, FERMO, MS2DECIDE k) over compounds 90-95
- Diff report of regenerated ranks vs. published Table 4 values
- Spot-check confirmation log (e.g., 19/89 known GNPS matches at cosine >= 0.7)

## How to apply

Load the per-compound annotation outputs (GNPS, Sirius, ISDB-LOTUS) for the 95-natural-product chemodiverse library from the MejriY/MS2DECIDE repository, then recompute ranks for the six unreported compounds (ids 90-95) under each baseline: GNPS O with the q(6,0.02) filter at the 1e-4 threshold, default GNPS cosine with the 0.7 cutoff, GNPS I, the Sirius confidence (approximate) score, the ISDB-LOTUS reported cosine, and the FERMO novelty score. Where the original authors applied fallback values (e.g., 0.7 default T_gs/T_gi when GNPS-Sirius Tanimoto cannot be computed, 0.5 when Sirius proposes no annotation), mirror those defaults so ranks are comparable. Write the regenerated ranks for all methods to table_4.csv and diff against the published Table 4. Rationale: validating the single most load-bearing claim — that MS2DECIDE's knownness score k is the only method ranking all six unreported NPs in the top ten — by spot-checking intermediate reference values (e.g., GNPS matching only 19 of 89 known features at cosine >= 0.7) catches silent pipeline drift before the table is reused.

## Related tools

- **MS2DECIDE** (Source of the knownness score k outputs being validated against the published Table 4) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Baseline ranks recomputed under GNPS O (q(6,0.02), 1e-4 threshold) and default cosine 0.7 cutoff; reference value 19/89 known matches)
- **Sirius** (Baseline ranks recomputed from the Sirius 6 confidence (approximate) score)
- **ISDB-LOTUS** (Baseline ranks recomputed from the reported ISDB-LOTUS cosine values)
- **FERMO** (Baseline ranks recomputed from the FERMO novelty score)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- MS2DECIDE knownness score k is the only method among all baselines ranking all six unreported compounds (ids 90-95) within the top ten of the 95-compound library
- Spot-check invariant: GNPS matches exactly 19 of 89 known features at cosine >= 0.7
- Regenerated table_4.csv rank columns diff cleanly against the published Table 4 values (no rank shifts beyond rounding)
- Fallback values are correctly reproduced in the recomputation: default 0.7 for T_gs/T_gi when the GNPS-Sirius Tanimoto is uncomputable, and 0.5 for unannotated Sirius features
- ISDB-LOTUS recomputation respects the strict 0.02 Da library-search setting used by the authors

## Limitations

- Sirius in some cases fails to propose an annotation; the authors patched this with a 0.5 matching-score default, which must be replicated or rank comparisons will diverge
- When there is no GNPS or Sirius match, the GNPS-Sirius Tanimoto cannot be calculated and a default value of 0.7 was assigned to T_gs and T_gi — reproductions ignoring this fallback will not match Table 4
- Iterative weighted GNPS analog search can produce empty annotations; a separate report (empty.tsv) may be needed to complete missing InChIs/SMILES before ranks are comparable
- Redundant features in the Sirius structure_identifications.tsv (duplicated mappingFeatureId values) can raise 'DataFrame index must be unique' errors and must be de-duplicated before recomputation

## Evidence

- [abstract] We demonstrate, through two case studies, that MS2DECIDE reliably captures the novelty of natural products from their tandem mass spectra.: "We demonstrate, through two case studies, that MS2DECIDE reliably captures the novelty of natural products from their tandem mass spectra."
- [readme] a default value of 0.7 was assigned to $T_{gs}$ and $T_{gi}$ in these instances.: [evidence span withheld — anchor status: external_doc]
- [readme] in some cases, Sirius was not able to propose an annotation. To remedy, we associated a value of 0.5 to Sirius matching score.: [evidence span withheld — anchor status: external_doc]
- [readme] For ISDB-LOTUS, since a strict library search was applied (0.02 Da), we considered a zero answer as an important information: [evidence span withheld — anchor status: external_doc]
