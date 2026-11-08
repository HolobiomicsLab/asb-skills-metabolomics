---
name: instrument-type-spectra-annotation
description: Use when building MS/MS training corpora where spectra come from mixed
  instrument sources and you need to tag each spectrum with collision-cell vs IT instrument
  type and energy before merging, binning, or model training.
when_to_use_negative:
- The corpus already contains only spectra from a single instrument type and single
  collision energy, so tagging adds no information for model splitting.
- Spectra lack instrument-type or collision-energy metadata in their source records,
  making the annotation unreliable.
- The downstream task is direct spectral-library matching rather than fingerprint-prediction
  model training, where instrument-driven model splits are not needed.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MoNA
  role: Source of MS/MS training spectra with instrument and collision-energy metadata
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: NIST 17
  role: Source of MS/MS training spectra used to evaluate instrument-type clustering
    and model splits
- name: InChIKey
  role: Compound identifier used to merge multiple collision-energy spectra of the
    same compound before instrument annotation is propagated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:298aba23da3e74c8cd68d103b6b8e360fe702733bf2475d630268082ec41d230
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - InChIKey
  - MoNA
  - NIST 17
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# instrument-type-spectra-annotation

## Summary

Annotates each MS/MS spectrum in a training library (MoNA or NIST 17) with its instrument type — collision-cell instruments (HCD, QqQ, Q-TOF) versus ion trap (IT) — and collision energy, so that downstream fingerprint-prediction models can be trained on instrument-matched or instrument-controlled data splits.

## When to use

Use when building MS/MS training corpora where spectra come from mixed instrument sources and you need to tag each spectrum with collision-cell vs IT instrument type and energy before merging, binning, or model training.

## When NOT to use

- The corpus already contains only spectra from a single instrument type and single collision energy, so tagging adds no information for model splitting.
- Spectra lack instrument-type or collision-energy metadata in their source records, making the annotation unreliable.
- The downstream task is direct spectral-library matching rather than fingerprint-prediction model training, where instrument-driven model splits are not needed.

## Inputs

- MS/MS spectra from MoNA repository libraries (Vaniya/Fiehn Natural Products, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC)
- MS/MS spectra from the NIST 17 spectral library
- Per-spectrum instrument-type metadata (HCD, QqQ, Q-TOF, IT)
- Per-spectrum collision-energy metadata (eV)

## Outputs

- Instrument-tagged MS/MS spectra (collision-cell vs IT labels)
- Collision-energy tags partitioning spectra at the 30 eV cutoff (low ≤ 30 eV, high ≥ 30 eV)
- Merged per-compound spectra (keyed by InChIKey) with instrument annotation retained
- Per-spectrum instrument/energy assignment table suitable for model-split design

## How to apply

Download MS/MS spectra from MoNA libraries (Vaniya/Fiehn Natural Products, GNPS, RIKEN PlaSMA, MassBank, Fiehn HILIC) and NIST 17, keeping each spectrum's instrument type and collision-energy metadata. Select spectra acquired in positive mode and tag them as either collision-cell instruments (HCD, QqQ, Q-TOF) or ion trap (IT). Because the article found that 30 eV was a reasonable cutoff for high and low collision energies, tag spectra at or above 30 eV as high energy and below as low energy when splitting training sets; spectra acquired by IT clustered with collision-cell spectra, so IT can be combined with HCD for model training without loss of prediction performance. Merged spectra for the same compound (via InChIKey) inherit the instrument tag from their constituent spectra, which are merged when their m/z difference is 0.1 or less.

## Related tools

- **MoNA** (Source of MS/MS training spectra with instrument and collision-energy metadata) — https://mona.fiehnlab.ucdavis.edu/
- **NIST 17** (Source of MS/MS training spectra used to evaluate instrument-type clustering and model splits)
- **InChIKey** (Compound identifier used to merge multiple collision-energy spectra of the same compound before instrument annotation is propagated)

## Examples

```
python3 main.py
```

## Evaluation signals

- Every spectrum in the resulting table carries a non-empty instrument-type tag limited to {HCD, QqQ, Q-TOF, IT}.
- Spectra tagged 'collision-cell' exactly match the set of HCD, QqQ, and Q-TOF records; IT spectra are tagged separately.
- Collision-energy split respects the 30 eV cutoff: low-energy model tested on low-energy spectra must not include high-energy spectra in its training folds.
- Count of distinct compounds after merging by InChIKey matches the number of distinct InChIKeys present in the merged corpus.
- Training model on IT + HCD combined does not degrade prediction performance relative to separate IT-only or HCD-only models (per Table 1b).

## Limitations

- Spectra generated by the same instrument type but very different collision energies (e.g. 14 eV vs 38 eV) exhibit very different fragmentation patterns, so a single combined model trained across the 30 eV cutoff shows slightly lower accuracy than instrument- and energy-specific models (e.g. the combined model's accuracy was slightly lower than the low-collision-energy model when tested with low-energy spectra).
- Collision-cell instruments (HCD, QqQ) and IT share a clustering signature in PCA, but IT adopts a completely different fragmentation mechanism than collision-cell instruments, so treating them as interchangeable beyond model-training convenience risks misleading spectral interpretation.
- Spectra must be restricted to positive-ion mode and known instrument classes; records from unlisted instrument types cannot be reliably tagged.
- Multiple spectra per compound are merged only when their m/z difference is ≤ 0.1, so spectra differing by more than 0.1 m/z escape the merge and may carry inconsistent instrument tags.
- The 30 eV cutoff was empirically determined on NIST 17 data (916 compounds with more than 30 spectra each) and may not generalize to other libraries or collision regimes.

## Evidence

- [results] Combining IT and HCD does not hurt model performance, validating instrument grouping: [evidence span withheld — anchor status: not_found]
