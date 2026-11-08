---
name: rule-based-fragmentation-module-evaluation
description: Use when a molecule set is enriched in modular metabolites (lipids, carnitines,
  polyphenols, flavonoids) and you must decide whether adding a rule-based fragmentation
  module to a learned predictor improves ESI-MS/MS spectra prediction.
when_to_use_negative:
- The evaluation domain is exposomics-style contaminant chemistry — the article reports
  no improvement of MSML+MSRB over MSML there, so the comparison will not show the
  expected signal
- The task is EI-MS/MS prediction or compound identification (MS2C) benchmarking rather
  than ESI-MS/MS spectral-prediction module comparison
- Predictions must cover non-QTOF instrument types — the article notes CFM-ID does
  not yet handle MS spectra collected using instruments other than QTOF instruments
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: CFM-ID 4.0
  role: Provides both the MSML machine-learned predictor and the MSRB rule-based module
    being compared; run in MSML-only and MSML+MSRB configurations
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: ClassyFire
  role: Classifies sampled molecules so MSRB can identify chemical classes suitable
    for rule-based fragmentation and so evaluation subsets match domain superclass
    distributions
- name: Docker
  role: Deployment vehicle for running CFM-ID 4.0 via the wishartlab/cfmid image for
    batch spectral prediction
  repo: https://hub.docker.com/r/wishartlab/cfmid
- name: Metlin
  role: Source of both the sampled molecules and the experimental ESI-MS/MS spectra
    used as prediction ground truth
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:23ac57562559af166b3f359c16f0e6f0fe3b37b6e22511599623ccc7d2d97edb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - ClassyFire
  - Docker
  - Metlin
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# rule-based-fragmentation-module-evaluation

## Summary

Benchmark protocol for assessing whether CFM-ID 4.0's hand-written SMIRKS-based rule module (MSRB, 313 rules over 11 chemical classes and 3 adduct types) improves ESI-MS/MS spectral prediction over the machine-learned MSML module alone, using Dice-coefficient comparison against experimental Metlin spectra.

## When to use

Use when a molecule set is enriched in modular metabolites (lipids, carnitines, polyphenols, flavonoids) and you must decide whether adding a rule-based fragmentation module to a learned predictor improves ESI-MS/MS spectra prediction.

## When NOT to use

- The evaluation domain is exposomics-style contaminant chemistry — the article reports no improvement of MSML+MSRB over MSML there, so the comparison will not show the expected signal
- The task is EI-MS/MS prediction or compound identification (MS2C) benchmarking rather than ESI-MS/MS spectral-prediction module comparison
- Predictions must cover non-QTOF instrument types — the article notes CFM-ID does not yet handle MS spectra collected using instruments other than QTOF instruments

## Inputs

- Molecules sampled from the Metlin 2019 dataset, stratified to match ClassyFire superclass distributions of HMDB, FooDB, and ContaminantDB
- Experimental QTOF ESI-MS/MS reference spectra from Metlin at 10, 20, and 40 eV in [M+H]+ and [M-H]- modes
- CFM-ID 4.0 MSRB rule set (313 SMIRKS rules covering 11 chemical classes and 3 adduct types)

## Outputs

- Per-domain Dice scores for MSML vs. MSML+MSRB configurations (exposomics, foodomics, human metabolomics)
- Per-domain detail plots analogous to Supplemental Figures S1-S6
- Decision on whether rule-based fragmentation adds value for a given chemical domain

## How to apply

Build domain-matched evaluation subsets by sampling Metlin 2019 compounds so each subset reproduces the ClassyFire superclass distribution of HMDB (human metabolomics), FooDB (foodomics), and ContaminantDB (exposomics). Run CFM-ID 4.0's MSML module to predict QTOF ESI-MS/MS spectra at 10, 20, and 40 eV collision energies in both [M+H]+ and [M-H]- modes, then repeat with the MSML+MSRB configuration in which the MSRB module (invoked after ClassyFire identifies chemical classes suitable for rule-based fragmentation) fires first and MSML handles the remainder. Score each prediction against the experimental Metlin spectrum with the Dice coefficient over peak presence, counting a peak pair as matched when the m/z difference is below 0.01 Da or 10 ppm, whichever occurs first. Compare per-domain Dice scores: MSML+MSRB should exceed MSML on foodomics and metabolomics subsets (rich in modular molecules such as lipids, carnitines and polyphenols) while showing no gain on exposomics.

## Related tools

- **CFM-ID 4.0** (Provides both the MSML machine-learned predictor and the MSRB rule-based module being compared; run in MSML-only and MSML+MSRB configurations) — https://bitbucket.org/wishartlab/cfm-id-code/
- **ClassyFire** (Classifies sampled molecules so MSRB can identify chemical classes suitable for rule-based fragmentation and so evaluation subsets match domain superclass distributions)
- **Docker** (Deployment vehicle for running CFM-ID 4.0 via the wishartlab/cfmid image for batch spectral prediction) — https://hub.docker.com/r/wishartlab/cfmid
- **Metlin** (Source of both the sampled molecules and the experimental ESI-MS/MS spectra used as prediction ground truth)

## Examples

```
docker pull wishartlab/cfmid:latest
```

## Evaluation signals

- MSML+MSRB Dice scores must be higher than MSML-alone Dice scores on the foodomics and metabolomics (HMDB-matched) subsets, matching the qualitative pattern of Table 2
- No improvement (or negligible difference) of MSML+MSRB over MSML should be observed on the exposomics (ContaminantDB-matched) subset
- Each evaluation subset's ClassyFire superclass distribution must match that of its target database (HMDB, FooDB, or ContaminantDB)
- Peak matches use the specified tolerance: m/z difference < 0.01 Da or < 10 ppm, whichever threshold is hit first
- Predictions must cover all three collision energies (10, 20, 40 eV) and both polarity modes ([M+H]+, [M-H]-)

## Limitations

- The MSRB advantage is confined to modular molecules (lipids, carnitines, polyphenols); rule coverage (313 SMIRKS rules, 11 chemical classes, 3 adduct types) does not extend the benefit to exposomics-type compounds
- CFM-ID currently handles only QTOF-collected spectra; non-QTOF instruments are out of scope
- Deeper fragmentation graphs and larger rule sets add computational complexity, and the authors note room for further improvement in both accuracy and runtime without exact benchmarks

## Evidence
