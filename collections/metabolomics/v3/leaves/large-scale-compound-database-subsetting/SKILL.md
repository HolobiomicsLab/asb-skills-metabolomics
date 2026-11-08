---
name: large-scale-compound-database-subsetting
description: Use when an enumerated synthetic NPS database (hundreds of millions of
  derivative SMILES with precomputed exact masses) must be narrowed to candidates
  plausibly matching an unidentified GC/EI-MS analyte.
when_to_use_negative:
- The analyte's mass spectrum already matches an established reference library (NIST,
  Wiley, or SWGDRUG) — PS2MS enumeration and filtering is only triggered when no reference
  match is found in conventional screening
- The chemical structure of the NPS is already known — in that case spectrum can be
  predicted directly (e.g. NEIMS) with no need to subset a derivative database
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2258
tools:
- name: PS2MS
  role: Implements the enumeration and rule-based filtering pipeline that generates
    and subsets the synthetic NPS database
  repo: https://github.com/jhhung/PS2MS
- name: rdkit
  role: Computes molecular properties used by the filters (exact masses, smallest
    sets of smallest rings) and supports the enumeration step
  repo: https://www.rdkit.org/docs/Install.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c05019
    title: 'PS<sup>2</sup>MS: A Deep Learning-Based Prediction System for Identifying
      New Psychoactive Substances Using Mass Spectrometry'
  source_content_hash: sha256:0b8a251a95ab75057b0c1d8d63532b56a0cd7f67b3c288f667ddc4cded466d8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jhhung/PS2MS
  tools:
  - PS2MS
  - python
  - rdkit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.96
n_claims: 25
---

# Large-scale compound database subsetting

## Summary

Reduces an extremely large enumerated derivative database (PS2MS enumerated ~422 million cathinone-like compounds) to a tractable candidate subset using rule-based filters: M+ mass-weight windows around the analyte's reference peak, drug-like criteria, ring-complexity rules, and synthesis-difficulty (SCScore) thresholds.

## When to use

Use when an enumerated synthetic NPS database (hundreds of millions of derivative SMILES with precomputed exact masses) must be narrowed to candidates plausibly matching an unidentified GC/EI-MS analyte.

## When NOT to use

- The analyte's mass spectrum already matches an established reference library (NIST, Wiley, or SWGDRUG) — PS2MS enumeration and filtering is only triggered when no reference match is found in conventional screening
- The chemical structure of the NPS is already known — in that case spectrum can be predicted directly (e.g. NEIMS) with no need to subset a derivative database

## Inputs

- Enumerated derivative database (SMILES plus precomputed exact masses, e.g. from task_001 enumeration of a cathinone-like core)
- Analyte SWGDRUG reference spectrum (GC/EI-MS, 70 eV) providing the M+ peak molecular weight
- Filter rule definitions: mwf tolerance settings (±1 Da, ±5 Da), 57 drug-like criteria (Virshup et al.), Messy rings threshold (>7 SSSR rings), ΔSCScore threshold αn

## Outputs

- CSV candidate list for each tolerance setting (mwf ±1 and mwf ±5 subsets)
- Summary record of candidate counts per tolerance/filter setting (counts equal to CSV row counts)
- Filtered synthetic NPS database passed onward to NEIMS spectrum prediction

## How to apply

Extract the molecular-weight reference from the M+ peak of the analyte's SWGDRUG reference spectrum, then scan the enumerated derivative database (built by substituting hydrogen atoms in the core structure with functional groups) and retain only compounds whose exact mass falls within the chosen tolerance window (mwf ±1 Da for a tight subset, mwf ±5 Da for a broader one), writing each subset to its own CSV. Apply the rule-based filters in layered fashion: remove compounds failing any of the 57 drug-like criteria (Virshup et al.), drop structures with more than 7 smallest sets of smallest rings (the most effective single rule in the paper), and restrict functional groups whose average synthesis difficulty differential ΔSCScore falls below the threshold αn so highly challenging-to-synthesize compounds never enter enumeration. The rationale is that mass proximity to the M+ peak is a hard physical constraint while SCScore and drug-likeness rules prune chemically implausible derivatives before expensive spectrum prediction.

## Related tools

- **PS2MS** (Implements the enumeration and rule-based filtering pipeline that generates and subsets the synthetic NPS database) — https://github.com/jhhung/PS2MS
- **rdkit** (Computes molecular properties used by the filters (exact masses, smallest sets of smallest rings) and supports the enumeration step) — https://www.rdkit.org/docs/Install.html

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('derivatives.csv'); df[abs(df.exact_mass - M_PLUS) <= 1].to_csv('mwf_pm1.csv', index=False)"
```

## Evaluation signals

- Every row in each output subset has an exact mass within the corresponding window (±1 Da or ±5 Da) of the reference M+ mass
- Reported candidate counts per tolerance setting exactly equal the row counts of the corresponding CSV files
- No retained compound has more than 7 smallest sets of smallest rings (Messy rings rule)
- All functional groups surviving enumeration have ΔSCScore below the threshold αn
- Pre-filter enumeration scale is consistent with the paper's ~422 million compounds; post-filter subsets are orders of magnitude smaller and mass-feasible relative to the analyte M+ peak

## Limitations

- The article specifies the mass filter only as a 'certain tolerance range' around the M+ peak; the ±1 Da and ±5 Da settings must be taken from the mwf task configuration, not from a stated paper threshold
- The exact number of enumeration levels and its impact are unspecified ('The number of enumeration levels also has a similar impact'), so filter outcomes depend on upstream enumeration depth
- Filtering depends on a valid M+ peak from a SWGDRUG reference spectrum; a misassigned M+ peak shifts the entire mass window and invalidates both subsets

## Evidence

- [discussion] M+ peak as mass filter reference: "utilizes the M+ peak as a reference for the mass weight filtering of enumerated compounds within a certain tolerance range"
- [methods] Scale of enumeration and filtering: "the process of enumeration and filtering resulted in a total of approximately 422 million compounds."
- [methods] Messy rings as most effective filter rule: "The most effective rule was Messy rings (more than 7 smallest sets of smallest rings)"
- [methods] SCScore-based functional-group screening: "retained only the functional groups whose average synthesis difficulty differential (represented as ΔSCScore) fell below a threshold of αn"
- [readme] Enumeration by hydrogen substitution on core structure: "PS2MS will generate a synthetic database by substituting hydrogen atoms in the core structure of a given drug with functional groups."
