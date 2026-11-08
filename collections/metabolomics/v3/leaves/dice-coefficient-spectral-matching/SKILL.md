---
name: dice-coefficient-spectral-matching
description: Use when comparing predicted QTOF ESI-MS/MS spectra (e.g., CFM-ID MSML
  or MSML+MSRB output) against experimental reference spectra to produce a per-domain
  or per-compound similarity score for spectral-prediction benchmarking.
when_to_use_negative:
- The comparison involves intensity-profile similarity or normalized dot-product style
  scoring rather than binary peak-presence benchmarking — Dice over presence ignores
  intensity information by design.
- Spectra were acquired on instruments other than QTOF; CFM-ID 4.0 at this version
  only handles QTOF-style ESI-MS/MS prediction.
- The goal is compound identification (MS2C task) rather than spectral-prediction
  accuracy scoring; the article evaluates that task separately, e.g., via the CASMI
  2016 dataset.
tools:
- name: CFM-ID 4.0
  role: Generates the predicted ESI-MS/MS peak lists (MSML and MSML+MSRB configurations)
    that are scored against experimental spectra
  repo: https://bitbucket.org/wishartlab/cfm-id-code/
- name: MSRB-fragmenter
  role: Rule-based (SMIRKS) predictor supplying additional predicted peaks for molecules
    in covered chemical classes before Dice scoring
  repo: https://bitbucket.org/wishartlab/msrb-fragmenter
- name: ClassyFire
  role: Identifies the chemical classes of input molecules so the MSRB rule set is
    applied only where suitable
- name: Docker CFM-ID image
  role: Containerized installation of CFM-ID for running the predictions locally
  repo: https://hub.docker.com/r/wishartlab/cfmid
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c01465
    title: 'CFM-ID 4.0: More Accurate ESI-MS/MS Spectral Prediction and Compound Identification'
  source_content_hash: sha256:f32511509d92dedd4bb96c9dddbbfe1c97e96825839f1477f14a6022430552c7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CFM-ID 4.0
  - ClassyFire
  - Docker CFM-ID image
  - MSRB-fragmenter
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dice-coefficient-spectral-matching

## Summary

Scores predicted vs. experimental ESI-MS/MS spectra with the Dice coefficient over binary peak presence, counting a peak pair as a match when the m/z difference is smaller than 0.01 Da or 10 ppm (whichever occurs first). Used in CFM-ID 4.0 to quantify spectral-prediction accuracy per metabolomics domain (exposomics, foodomics, human metabolomics).

## When to use

Use when comparing predicted QTOF ESI-MS/MS spectra (e.g., CFM-ID MSML or MSML+MSRB output) against experimental reference spectra to produce a per-domain or per-compound similarity score for spectral-prediction benchmarking.

## When NOT to use

- The comparison involves intensity-profile similarity or normalized dot-product style scoring rather than binary peak-presence benchmarking — Dice over presence ignores intensity information by design.
- Spectra were acquired on instruments other than QTOF; CFM-ID 4.0 at this version only handles QTOF-style ESI-MS/MS prediction.
- The goal is compound identification (MS2C task) rather than spectral-prediction accuracy scoring; the article evaluates that task separately, e.g., via the CASMI 2016 dataset.

## Inputs

- Predicted ESI-MS/MS peak lists from CFM-ID 4.0 (MSML or MSML+MSRB configurations)
- Experimental QTOF ESI-MS/MS reference spectra from the Metlin MS database
- Collision-energy metadata (10, 20, 40 eV) and ionization mode ([M+H]+ or [M-H]-) per spectrum

## Outputs

- Per-compound Dice similarity scores over binary peak presence
- Per-domain (exposomics, foodomics, human metabolomics) tabulated Dice scores for MSML vs. MSML+MSRB
- Optionally, per-domain detail plots equivalent to Supplemental Figures S1-S6

## How to apply

Predict spectra for each molecule at 10, 20, and 40 eV collision energies in both [M+H]+ and [M-H]- modes (e.g., with CFM-ID 4.0's MSML module or the MSML+MSRB configuration, where the SMIRKS-based MSRB rules cover 313 rules over 11 chemical classes and 3 adduct types and ClassyFire identifies which classes are suitable for MSRB fragmentation). Extract the predicted peak list and the experimental Metlin spectrum, binarize to peak-presence sets, and compute the Dice coefficient over peak presence: 2|common peaks| / (|predicted| + |experimental|). When matching peaks, accept a pair only if the m/z difference is smaller than 0.01 Da or 10 ppm, applying whichever threshold is reached first. Tabulate Dice scores per condition (module configuration, domain subset, collision energy, ionization mode) rather than pooling, since the interesting signal is the per-domain difference between MSML and MSML+MSRB.

## Related tools

- **CFM-ID 4.0** (Generates the predicted ESI-MS/MS peak lists (MSML and MSML+MSRB configurations) that are scored against experimental spectra) — https://bitbucket.org/wishartlab/cfm-id-code/
- **MSRB-fragmenter** (Rule-based (SMIRKS) predictor supplying additional predicted peaks for molecules in covered chemical classes before Dice scoring) — https://bitbucket.org/wishartlab/msrb-fragmenter
- **ClassyFire** (Identifies the chemical classes of input molecules so the MSRB rule set is applied only where suitable)
- **Docker CFM-ID image** (Containerized installation of CFM-ID for running the predictions locally) — https://hub.docker.com/r/wishartlab/cfmid

## Examples

```
docker pull wishartlab/cfmid
```

## Evaluation signals

- Every matched peak pair must satisfy m/z difference < 0.01 Da OR < 10 ppm, applying whichever threshold occurs first; matches outside both windows are excluded.
- Dice scores are computed on binary peak-presence sets, so adding intensity weighting would change the metric and is incorrect for this comparison.
- In the MSML vs. MSML+MSRB benchmark, MSML+MSRB must show higher Dice scores than MSML alone on the foodomics and metabolomics subsets, and no improvement on the exposomics subset (Table 2 pattern).
- Scores should be reported separately for 10, 20, and 40 eV and for [M+H]+ and [M-H]- modes; pooling across energies or modes masks the expected configuration differences.

## Limitations

- Match acceptance is capped at 0.01 Da / 10 ppm, so small systematic calibration shifts can suppress Dice scores even when fragments are chemically correct.
- Dice over peak presence discards relative-intensity information, so spectra with identical peaks but different intensities score identically.
- CFM-ID 4.0 predictions target QTOF instruments; non-QTOF spectra are out of scope, and deeper fragmentation graphs add computational complexity with acknowledged room for runtime improvement.

## Evidence

- [other] Spectral prediction benchmark definition: [evidence span withheld — anchor status: not_found]
