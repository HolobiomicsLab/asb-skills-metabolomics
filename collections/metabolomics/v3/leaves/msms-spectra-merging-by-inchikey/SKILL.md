---
name: msms-spectra-merging-by-inchikey
description: Use when building a training set from MS/MS spectral libraries (e.g.
  NIST 17, MoNA) where the same compound appears in multiple spectra at different
  collision energies or instrument types, and a single unified input vector per compound
  is needed.
when_to_use_negative:
- 'When the goal is collision-energy-specific prediction: the article showed a combined
  model has slightly lower accuracy and F1 than a low-energy-only model on low-energy
  test spectra, so use separate models instead of merging'
- When the spectra are already deduplicated per compound (e.g. a single reference
  spectrum per InChIKey) and no peak-consolidation step is needed
- When the unknown analyte's candidate pool includes compounds whose spectra were
  acquired under drastically different conditions that the ANN was never trained to
  merge across
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: MetFID
  role: The downstream ANN that consumes the merged spectra; the merging step prepares
    the training inputs it requires
  repo: https://github.com/aohongyu/MetFID
- name: NIST 17
  role: Source spectral library whose spectra are grouped and merged by compound InChIKey
- name: MoNA
  role: Source repository from which MS/MS spectra are downloaded before merging by
    InChIKey
  repo: https://mona.fiehnlab.ucdavis.edu/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-020-01726-7
    title: 'MetFID: artificial neural network-based compound fingerprint prediction
      for metabolite annotation'
  source_content_hash: sha256:7170b5e38c01e6c8cd9e09026ceedce2a130f8a27167be3f09a31f3c8ca06b6c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/aohongyu/MetFID
  tools:
  - InChIKey
  - MetFID
  - MoNA
  - NIST 17
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# MS/MS spectra merging by InChIKey

## Summary

Aggregates multiple MS/MS spectra of the same compound — acquired at different collision energies (low ≤30 eV vs. high ≥30 eV) or on different instruments (HCD, QqQ, Q-TOF, IT) — into a single merged training spectrum keyed on the compound's first InChIKey block, to build collision-energy- and instrument-agnostic ANN inputs.

## When to use

Use when building a training set from MS/MS spectral libraries (e.g. NIST 17, MoNA) where the same compound appears in multiple spectra at different collision energies or instrument types, and a single unified input vector per compound is needed.

## When NOT to use

- When the goal is collision-energy-specific prediction: the article showed a combined model has slightly lower accuracy and F1 than a low-energy-only model on low-energy test spectra, so use separate models instead of merging
- When the spectra are already deduplicated per compound (e.g. a single reference spectrum per InChIKey) and no peak-consolidation step is needed
- When the unknown analyte's candidate pool includes compounds whose spectra were acquired under drastically different conditions that the ANN was never trained to merge across

## Inputs

- Multiple MS/MS spectra (peak lists with precursor mass, ionization mode, m/z–intensity pairs) of the same compound from NIST 17 or MoNA
- InChIKey identifiers for each spectrum's compound (first block used to group compounds)
- Instrument and collision-energy metadata for each spectrum (e.g. HCD 14 eV, HCD 38 eV, QqQ, IT)

## Outputs

- One merged spectrum per compound: a consolidated peak list with precursor mass, ionization mode, and summed m/z–intensity peaks
- Merged, binned m/z feature vectors used as ANN input (11,784 merged MoNA spectra of 5,667 compounds; 122,481 merged NIST 17 spectra of 10,730 compounds)

## How to apply

First select spectra that are positive-mode and acquired via collision-cell instruments (HCD, QqQ, Q-TOF) or ion trap (IT). Group spectra of the same compound using the InChIKey as the compound identifier. Scale peak intensities to 0–100, remove spectra with fewer than five peaks above 2% relative intensity, denoise by removing peaks with intensity lower than 10, then merge peaks from multiple spectra of the same compound into one peak when their m/z difference is ≤0.1. Before merging, run cluster analysis on spectra grouped by collision energy and instrument type to determine the cutoff: the article found 30 eV a reasonable cutoff separating low from high collision-energy spectra, and observed that IT spectra nonetheless cluster with collision-cell spectra despite the different fragmentation mechanism. The merged, compound-level spectra then feed the downstream binning (e.g. 0.01 m/z bins) that produces the ANN's input vectors; a 528-bit MACCS/FP3/FP4 fingerprint (via OpenBabel) serves as the multi-label target. Validation is structure-disjoint: no shared first InChIKey block between training and test compounds, so merging must be done consistently on both sides.

## Related tools

- **MetFID** (The downstream ANN that consumes the merged spectra; the merging step prepares the training inputs it requires) — https://github.com/aohongyu/MetFID
- **NIST 17** (Source spectral library whose spectra are grouped and merged by compound InChIKey)
- **MoNA** (Source repository from which MS/MS spectra are downloaded before merging by InChIKey) — https://mona.fiehnlab.ucdavis.edu/

## Examples

```
python3 main.py
```

## Evaluation signals

- Each merged spectrum contains exactly one precursor mass and one ionization mode per compound (one merged record per unique first InChIKey block)
- No two peaks in a merged spectrum remain within 0.1 m/z of each other (merge rule invariant)
- The total merged compound count matches the expected library coverage (e.g. 5,667 compounds from 11,784 MoNA spectra; 10,730 compounds from 122,481 NIST 17 spectra)
- Training and test sets are structure-disjoint: no shared first InChIKey block across the split
- Cluster analysis of pre-merge spectra confirms that spectra with similar collision energies cluster together and that the 30 eV cutoff separates low/high energy groups

## Limitations

- The combined merged model has slightly lower accuracy and F1 than separate low/high collision-energy models on low-energy test spectra, so merging sacrifices some performance for model simplicity
- Collision-energy merging can be confounded: spectra generated by the same instrument but at very different collision energies (e.g. 14 eV vs 38 eV) show very different fragmentation patterns, with higher energies yielding more peaks including more low-m/z peaks
- Ion trap spectra cluster with collision-cell spectra despite a completely different fragmentation mechanism, so merging across instrument types may be unjustified in principle even if empirically it did not hurt performance

## Evidence
