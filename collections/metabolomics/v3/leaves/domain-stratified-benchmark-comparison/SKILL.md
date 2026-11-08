---
name: domain-stratified-benchmark-comparison
description: Use when a predictor's claimed improvement must be verified per application
  domain — e.g. testing whether adding a rule-based module (MSRB) helps on foodomics/metabolomics-like
  molecule sets but not exposomics-like ones.
when_to_use_negative:
- The goal is a single domain-agnostic accuracy figure on one held-out test set —
  domain-stratified resampling would distort the aggregate benchmark rather than serve
  it.
- No experimental reference spectra are available — the Dice-coefficient comparison
  against Metlin experimental spectra cannot be performed.
- The evaluated module's scope does not overlap the stratification axis (e.g. rule-based
  modules covering only acylcarnitines, acylcholines, flavonols, flavones, flavanones,
  and flavonoid glycosides will show no gain on exposomics-like non-modular compound
  sets).
tools:
- name: CFM-ID 4.0
  role: 'Runs both predictor configurations being compared: the machine-learned module
    (MSML) and the combined MSML+MSRB configuration with the SMIRKS rule-based module
    first'
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: ClassyFire
  role: Assigns chemical superclasses used both for stratifying the evaluation subsets
    by domain and for identifying classes suitable for MSRB fragmentation
- name: MSRB fragmenter
  role: Standalone source for the hand-written SMIRKS-based rule-based module (313
    rules, 11 chemical classes, 3 adduct types) whose contribution is quantified per
    domain
  repo: https://bitbucket.org/wishartlab/msrb-fragmenter
- name: CFM-ID 4.0 Docker image
  role: Deployment vehicle for running the prediction configurations reproducibly
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:d4f3618f4819de6a1a6befaa292cee2c0894360398b7958c59f050dd51afc3c8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - CFM-ID 4.0 Docker image
  - ClassyFire
  - MSRB fragmenter
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# domain-stratified-benchmark-comparison

## Summary

Validates whether a predictor improvement holds per research domain by building evaluation subsets that replicate domain-specific chemical-class distributions, then comparing metric scores (Dice) per domain between model variants. Used in CFM-ID 4.0 to test the MSRB rule-based module against the MSML machine-learned module.

## When to use

Use when a predictor's claimed improvement must be verified per application domain — e.g. testing whether adding a rule-based module (MSRB) helps on foodomics/metabolomics-like molecule sets but not exposomics-like ones.

## When NOT to use

- The goal is a single domain-agnostic accuracy figure on one held-out test set — domain-stratified resampling would distort the aggregate benchmark rather than serve it.
- No experimental reference spectra are available — the Dice-coefficient comparison against Metlin experimental spectra cannot be performed.
- The evaluated module's scope does not overlap the stratification axis (e.g. rule-based modules covering only acylcarnitines, acylcholines, flavonols, flavones, flavanones, and flavonoid glycosides will show no gain on exposomics-like non-modular compound sets).

## Inputs

- Metlin 2019 experimental ESI-MS/MS spectra (QTOF, positive and negative ionization modes) as the ground-truth reference
- Molecule structures from Metlin 2019 with ClassyFire superclass annotations for stratified sampling
- ClassyFire superclass distributions of HMDB, FooDB, and ContaminantDB as stratification targets
- Two predictor configurations to compare (e.g. MSML alone and MSML+MSRB)

## Outputs

- Three stratified evaluation subsets whose ClassyFire superclass distributions match HMDB, FooDB, and ContaminantDB
- Predicted QTOF ESI-MS/MS spectra at 10, 20, and 40 eV for [M+H]+ and [M-H]- per configuration per subset
- Per-domain table of Dice scores comparing MSML versus MSML+MSRB (cf. Table 2), optionally per-domain detail plots (cf. Supplemental Figures S1-S6)

## How to apply

First, build stratified subsets by sampling molecules from the Metlin 2019 dataset so each subset matches the ClassyFire superclass distribution of a domain-defining database: HMDB (human metabolomics), FooDB (foodomics), and ContaminantDB (exposomics). Next, run each predictor configuration on every sampled molecule — here MSML alone versus MSML+MSRB, where the hand-written SMIRKS-based rule-based module (313 rules over 11 chemical classes and 3 adduct types) runs first and the machine-learned module handles the remainder — predicting QTOF ESI-MS/MS spectra at 10, 20, and 40 eV collision energies in both [M+H]+ and [M-H]- modes. Score each configuration against the experimental Metlin spectra using the Dice coefficient over peak presence, counting a peak pair as a match when the m/z difference is smaller than 0.01 Da or 10 ppm (whichever occurs first). Finally, tabulate per-domain Dice scores for the two configurations (optionally exporting per-domain detail plots corresponding to Supplemental Figures S1-S6) and check that the improvement pattern is domain-specific, not uniform.

## Related tools

- **CFM-ID 4.0** (Runs both predictor configurations being compared: the machine-learned module (MSML) and the combined MSML+MSRB configuration with the SMIRKS rule-based module first) — https://bitbucket.org/wishartlab/cfm-id-code/
- **ClassyFire** (Assigns chemical superclasses used both for stratifying the evaluation subsets by domain and for identifying classes suitable for MSRB fragmentation)
- **MSRB fragmenter** (Standalone source for the hand-written SMIRKS-based rule-based module (313 rules, 11 chemical classes, 3 adduct types) whose contribution is quantified per domain) — https://bitbucket.org/wishartlab/msrb-fragmenter
- **CFM-ID 4.0 Docker image** (Deployment vehicle for running the prediction configurations reproducibly) — https://hub.docker.com/r/wishartlab/cfmid

## Evaluation signals

- Each stratified subset must match its target database's ClassyFire superclass distribution by construction (HMDB, FooDB, ContaminantDB).
- MSML+MSRB must show higher Dice scores than MSML alone on the foodomics and metabolomics domains, matching the qualitative pattern in Table 2.
- The exposomics domain must show no MSRB improvement — reproducing this null result is part of correct validation.
- Peak-matching tolerance applied consistently: m/z difference smaller than 0.01 Da or 10 ppm, whichever occurs first.
- Both configurations evaluated identically: 10, 20, and 40 eV collision energies and both [M+H]+ and [M-H]- modes.

## Limitations

- Improvements are domain-limited: gains appear only for domains rich in modular molecules (lipids, carnitines, polyphenols); exposomics-like sets see no benefit.
- Restricted to QTOF ESI-MS/MS spectra; CFM-ID does not yet handle spectra from other instrument types.
- Dice coefficient considers peak presence only, ignoring intensity information.

## Evidence

- [results] MSRB gain is domain-specific: [evidence span withheld — anchor status: not_found]
- [methods] Domain-stratified subset construction: [evidence span withheld — anchor status: not_found]
- [methods] Dice scoring with match tolerance: [evidence span withheld — anchor status: not_found]
