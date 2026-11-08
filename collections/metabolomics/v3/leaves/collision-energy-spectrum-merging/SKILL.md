---
name: collision-energy-spectrum-merging
description: Use when multiple MS/MS spectra of the same compound (same InChIKey)
  were acquired at different collision energies and must be consolidated into a single
  spectrum per compound for downstream fingerprint prediction or spectral feature
  table construction.
when_to_use_negative:
- Spectra are from different compounds or different InChIKeys.
- Only a single spectrum exists for a compound — no merging is needed.
- Spectra were acquired in negative mode or with adducts other than H+ or NH4+, outside
  the article's selection criteria.
- Peaks are already binned into a feature table; the merging step operates on raw
  peak lists.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MoNA
  role: Source repository from which the multiple MS/MS spectra per compound are downloaded
    before merging
  repo: https://mona.fiehnlab.ucdavis.edu/
- name: NIST 17
  role: Additional spectral library supplying spectra to be merged by InChIKey
- name: MetFID
  role: Downstream tool that consumes the merged spectra for training the CNN-based
    fingerprint predictor
  repo: https://github.com/aohongyu/MetFID
- name: InChIKey
  role: Compound identifier used to group spectra from the same compound for merging
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:ac3e6627118d97eb006bba110824184dd033cc8170f3cf955f066244f3bf6b8a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - InChIKey
  - MetFID
  - MoNA
  - NIST 17
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# collision-energy-spectrum-merging

## Summary

Merges multiple MS/MS spectra of the same compound (identified by InChIKey) acquired at different collision energies into a single training-ready spectrum, combining peaks whose m/z difference is ≤ 0.1 and averaging their intensities. This reduces duplicate spectra per compound for fingerprint prediction training while retaining fragmentation information from varying collision energies.

## When to use

Use when multiple MS/MS spectra of the same compound (same InChIKey) were acquired at different collision energies and must be consolidated into a single spectrum per compound for downstream fingerprint prediction or spectral feature table construction.

## When NOT to use

- Spectra are from different compounds or different InChIKeys.
- Only a single spectrum exists for a compound — no merging is needed.
- Spectra were acquired in negative mode or with adducts other than H+ or NH4+, outside the article's selection criteria.
- Peaks are already binned into a feature table; the merging step operates on raw peak lists.

## Inputs

- Multiple MS/MS spectra of the same compound (same InChIKey) acquired at different collision energies
- Peak lists with m/z and intensity values from MoNA or NIST 17 libraries
- Precursor ion m/z values
- Collision energy annotations

## Outputs

- One merged spectrum per compound (by InChIKey) with combined peak list (m/z, mean intensity)

## How to apply

Scale peak intensities to 0–100 and remove spectra with fewer than five peaks with relative intensity above 2% (post-scaling). Denoise by locating the highest-intensity peak whose mass is greater than the precursor mass, removing peaks less intense than it, then discarding all remaining peaks with intensity below 10. Select spectra acquired in the positive mode, from collision-cell instruments (HCD, QqQ, Q-TOF) or ion trap (IT), with H+ or NH4+ adduct only, and within the 100–1010 Da mass range. Group remaining spectra by compound InChIKey, then merge peaks from different spectra if their m/z difference is ≤ 0.1, assigning the merged peak the mean intensity. The rationale is that spectra acquired at different collision energies share overlapping fragments, and combining them produces one representative spectrum per compound that avoids duplicating the same compound's fragmentation information in the training set.

## Related tools

- **MoNA** (Source repository from which the multiple MS/MS spectra per compound are downloaded before merging) — https://mona.fiehnlab.ucdavis.edu/
- **NIST 17** (Additional spectral library supplying spectra to be merged by InChIKey)
- **MetFID** (Downstream tool that consumes the merged spectra for training the CNN-based fingerprint predictor) — https://github.com/aohongyu/MetFID
- **InChIKey** (Compound identifier used to group spectra from the same compound for merging)

## Examples

```
python3 main.py  # with testing_compound.txt in _files/; merged spectra are prepared upstream by the MetFID preprocessing pipeline
```

## Evaluation signals

- Merged peaks must have m/z difference ≤ 0.1 from the original peaks being combined.
- Intensity of each merged peak equals the mean of the constituent peaks' intensities.
- Number of output spectra equals the number of unique InChIKeys (not the raw spectrum count).
- All input spectra must have already been scaled to 0–100 intensity and denoised before merging.
- Verify no spectra from negative-mode or non-H+/NH4+ adduct acquisitions entered the merge step.

## Limitations

- Merging combines peaks across different collision energies into a single representative spectrum, which can obscure energy-specific fragmentation patterns (the article reports spectra at 14 eV vs 38 eV have very different fragmentation patterns).
- The 30 eV cutoff for high/low collision energy was determined by cluster analysis but the merging step does not preserve this distinction.
- The 0.1 m/z tolerance and mean-intensity rule are fixed; no adaptive tolerance or weighted intensity is applied.
- Applicable only to positive-mode spectra from collision-cell (HCD, QqQ, Q-TOF) or ion-trap instruments with H+ or NH4+ adducts, within the 100–1010 Da mass range.
- Spectra with fewer than five peaks of relative intensity above 2% are removed before merging, so low-quality spectra are excluded regardless of compound identity.

## Evidence

- [methods] Mean intensity of merged peaks: [evidence span withheld — anchor status: not_found]
