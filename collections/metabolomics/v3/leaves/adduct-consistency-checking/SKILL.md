---
name: adduct-consistency-checking
description: Use when enumerating within-library compound pairs from MS/MS reference
  libraries and a pair must be discarded unless both spectra were measured under the
  same protonated ion (M+H) condition.
when_to_use_negative:
- Pairs measured under different instrument conditions or with mismatched precursor
  ion types (e.g. one M+H, one M+Na) — the shifted-peak logic of ModiFinder's MS/MS
  alignment would be invalid
- Single-compound tasks (e.g. combinatorial fragmentation or peak assignment) that
  do not involve comparing two library spectra
- Cross-library pair enumeration where the source libraries differ — the paper restricts
  this filter to within-library pairs
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ModiFinder
  role: Downstream consumer of the adduct-consistent pair benchmark; the filter ensures
    its MS2-known vs MS2-unknown peak alignment operates on chemically comparable
    spectra.
  repo: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- name: MS/MS library search
  role: Supplies the reference library entries (spectra + SMILES + adduct annotations)
    from which candidate pairs are enumerated and adduct-checked.
- name: Analog library search
  role: Provides the analog-pair enumeration context in which the adduct-consistency
    filter is applied during benchmark construction.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/jasms.4c00061
    title: 'ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification
      Site Localization'
  source_content_hash: sha256:1f0704d2bcfeb24461f6c8a861a12a65a074a9a6e50e2600934ec9a326eb1a8c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
  tools:
  - Analog library search
  - MS/MS library search
  - ModiFinder
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# adduct-consistency-checking

## Summary

A benchmark-construction sub-step that verifies both members of a candidate compound pair share the same M+H adduct so their MS/MS spectra are directly comparable for modification site localization.

## When to use

Use when enumerating within-library compound pairs from MS/MS reference libraries and a pair must be discarded unless both spectra were measured under the same protonated ion (M+H) condition.

## When NOT to use

- Pairs measured under different instrument conditions or with mismatched precursor ion types (e.g. one M+H, one M+Na) — the shifted-peak logic of ModiFinder's MS/MS alignment would be invalid
- Single-compound tasks (e.g. combinatorial fragmentation or peak assignment) that do not involve comparing two library spectra
- Cross-library pair enumeration where the source libraries differ — the paper restricts this filter to within-library pairs

## Inputs

- MS/MS library spectra records from GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, and BERKELEY-LAB (spectra plus 2D structures in SMILES)
- Adduct annotation per library entry (precursor ion type, e.g. M+H)

## Outputs

- Filtered set of analog pairs annotated as sharing the same M+H adduct (subset of the 12,909-pair benchmark candidate table)
- Pair table entries flagged for inclusion/exclusion based on adduct consistency

## How to apply

When building the ModiFinder analog benchmark from the four reference MS/MS libraries (GNPS-MSMLS, GNPS-NIH-NATURALPRODUCTSLIBRARY_ROUND2_POSITIVE, GNPS-NIH-SMALLMOLECULEPHARMACOLOGICALLYACTIVE, BERKELEY-LAB), enumerate every possible within-library compound pair and check the adduct annotation of each member. Keep the pair only if both compounds share the same M+H adduct and were measured under matched instrument conditions; discard any pair where one member is a different adduct species (e.g. Na+, NH4+, neutral). This consistency filter runs alongside the precursor-mass checks (precursor mass < 2000 Da, precursor mass difference < 50% of the smaller compound's precursor mass) and before the RDKit GetSubstructMatch substructure test that determines the number of modification sites. The rationale is that ModiFinder compares the MS/MS alignment of MS2-known and MS2-unknown peaks, so shifted-peak interpretation is only valid when the observed precursor ions are chemically identical species.

## Related tools

- **ModiFinder** (Downstream consumer of the adduct-consistent pair benchmark; the filter ensures its MS2-known vs MS2-unknown peak alignment operates on chemically comparable spectra.) — https://github.com/Wang-Bioinformatics-Lab/ModiFinder_base
- **MS/MS library search** (Supplies the reference library entries (spectra + SMILES + adduct annotations) from which candidate pairs are enumerated and adduct-checked.)
- **Analog library search** (Provides the analog-pair enumeration context in which the adduct-consistency filter is applied during benchmark construction.)

## Evaluation signals

- The surviving pair count after adduct filtering must contribute to the final total of 12,909 analog pairs when combined with the other eligibility filters (precursor mass < 2000 Da, mass difference < 50%, single modification site).
- Every retained pair must have identical adduct strings on both members (both 'M+H'); any mismatch is a validation failure.
- Instrument metadata (collision energy / instrument type) should match between the two members of a retained pair, since the paper requires matched instrument conditions.
- Downstream ModiFinder runs on retained pairs should produce plausible likelihood landscapes, whereas runs on adduct-mismatched pairs would show systematically inconsistent shifted-peak annotations.

## Limitations

- The filter is defined only for protonated M+H adducts in this benchmark; other adduct species are simply discarded rather than normalized or converted.
- The paper reports the filter as part of benchmark assembly from four specific GNPS/Berkeley libraries; its behavior on libraries with missing or non-standard adduct annotations is not characterized.
- Matching instrument conditions is an additional implicit requirement, so adduct consistency alone is insufficient to guarantee spectral comparability.

## Evidence

- [other] Both members share the same M+H adduct: [evidence span withheld — anchor status: not_found]
- [methods] Benchmark eligibility filters: "(i) their precursor mass is checked to be less than 2000 Da, (ii) the difference in precursor masses is less than 50%"
- [abstract] MS/MS alignment basis: "leverages the alignment of peaks in MS/MS spectra between structurally related known and unknown small molecules"
- [other] Matched instrument conditions requirement: [evidence span withheld — anchor status: not_found]
- [readme] README citation confirming library source: "ModiFinder: Tandem Mass Spectral Alignment Enables Structural Modification Site Localization"
