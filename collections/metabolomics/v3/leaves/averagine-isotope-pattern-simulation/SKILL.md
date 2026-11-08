---
name: averagine-isotope-pattern-simulation
description: Use when you need synthetic peptide isotope patterns spanning defined
  monoisotopic mass and charge-state ranges — e.g., to create labeled signal+noise
  TIMS windows or to build an LSH reference database of isotope patterns for timsTOF
  data.
when_to_use_negative:
- When a gold-standard peak-level annotation of real MS data exists — the paper notes
  synthetic data cannot fully reflect real data behavior and was used only because
  no such ground truth was available
- When realistic chemical (self-similar contaminant) noise must be modeled — the article
  notes such noise would be misclassified as true signal and is not captured by this
  synthetic recipe
- When the goal is de novo feature finding from real TDF/TIMS raw data rather than
  benchmarking or reference-based matching
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: createSynthetics (proteolizard framework)
  role: Python module used to generate averagine-modeled synthetic isotope patterns
    and labeled signal/noise windows
  repo: https://github.com/hildebrandtlab/mzBucket
- name: mzBucket
  role: Repository hosting the createSynthetics module and output folders for the
    synthetic datasets
  repo: https://github.com/hildebrandtlab/mzBucket
- name: pyproteolizard-data
  role: Provides data-access classes (PyTimsDataHandle, TimsFrame, MzSpectrum) and
    binned mass-axis representation for timsTOF frames into which synthetic patterns
    are placed
  repo: https://github.com/theGreatHerrLebert/proteolizard-data
- name: pyproteolizard-algorithm
  role: Supplies the LSH implementation that hashes the generated averagine patterns
    into the reference database and classifies windows by hash collision
  repo: https://github.com/theGreatHerrLebert/proteolizard-algorithm
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1186/s12859-022-04833-5
    title: Locality-sensitive hashing enables efficient and scalable signal classification
      in high-throughput mass spectrometry raw data
  source_content_hash: sha256:2985e7eec7b798985767aa5b24e7ce9c0f7897dd3fc7226ef421a0115d14d942
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hildebrandtlab/mzBucket
  tools:
  - OpenTIMS
  - Python
  - TensorFlow 2
  - createSynthetics (proteolizard framework)
  - mzBucket
  - pyproteolizard-algorithm
  - pyproteolizard-data
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# averagine-isotope-pattern-simulation

## Summary

Generate synthetic peptide isotope patterns from the averagine model to serve as ground-truth signal in synthetic mass spectrometry benchmarks and as a hashed reference database for isotope-pattern detection via locality-sensitive hashing (LSH).

## When to use

Use when you need synthetic peptide isotope patterns spanning defined monoisotopic mass and charge-state ranges — e.g., to create labeled signal+noise TIMS windows or to build an LSH reference database of isotope patterns for timsTOF data.

## When NOT to use

- When a gold-standard peak-level annotation of real MS data exists — the paper notes synthetic data cannot fully reflect real data behavior and was used only because no such ground truth was available
- When realistic chemical (self-similar contaminant) noise must be modeled — the article notes such noise would be misclassified as true signal and is not captured by this synthetic recipe
- When the goal is de novo feature finding from real TDF/TIMS raw data rather than benchmarking or reference-based matching

## Inputs

- Averagine model parameters: monoisotopic mass range 150–5000 u (150–1800 Da for the reference database) and charge states 1–5
- Sampling and binning settings: 1e-5 sampling resolution, 1e-2 Da binning
- Synthetic noise recipe: Poisson(4)+1 peak counts, exponential intensities with parameter 15
- Window definition: 10 m/z windows with 0.1 m/z equally spaced bins

## Outputs

- Synthetic signal+noise peak windows with per-peak ground-truth labels (signal, noise_1, noise_2), written to data/output_create_synthetics at six intensity scales (1000, 500, 250, 125, 64, 32)
- Hashed reference database of synthetic averagine isotope patterns for LSH-based isotope-pattern detection

## How to apply

Use the createSynthetics module of the proteolizard framework (Python) to draw isotope patterns from the averagine model for monoisotopic masses in the range 150–5000 u with charge states 1–5, sampling at 1e-5 resolution and binning to 1e-2 Da for reference-database use. Combine these signal patterns with stochastically drawn noise peaks (Poisson(4)+1 peak counts, exponential intensities with parameter 15) inside 10 m/z windows with 0.1 m/z equally spaced binning, and label each synthetic peak as signal, noise_1, or noise_2. Repeat generation across maximum-intensity scales (1000, 500, 250, 125, 64, 32) to benchmark how signal-to-noise level affects downstream LSH classification. For the reference-based matching workflow, hash the averagine-generated patterns into the LSH database so that unidentified signal windows hashed later can be matched by hash collision, followed by explicit cosine similarity scoring (retain matches at or above the 0.6 threshold).

## Related tools

- **createSynthetics (proteolizard framework)** (Python module used to generate averagine-modeled synthetic isotope patterns and labeled signal/noise windows) — https://github.com/hildebrandtlab/mzBucket
- **mzBucket** (Repository hosting the createSynthetics module and output folders for the synthetic datasets) — https://github.com/hildebrandtlab/mzBucket
- **pyproteolizard-data** (Provides data-access classes (PyTimsDataHandle, TimsFrame, MzSpectrum) and binned mass-axis representation for timsTOF frames into which synthetic patterns are placed) — https://github.com/theGreatHerrLebert/proteolizard-data
- **pyproteolizard-algorithm** (Supplies the LSH implementation that hashes the generated averagine patterns into the reference database and classifies windows by hash collision) — https://github.com/theGreatHerrLebert/proteolizard-algorithm

## Examples

```
from proteolizardSynthetics.createSynthetics import create_synthetics; create_synthetics(mass_range=(150, 5000), charges=range(1, 6), window=10.0, bin_width=0.1, intensity_scales=[1000, 500, 250, 125, 64, 32])
```

## Evaluation signals

- Output files for all six intensity scales (1000, 500, 250, 125, 64, 32) exist under data/output_create_synthetics
- Generated peaks conform to the specified recipe: 10 m/z windows with 0.1 m/z equally spaced bins, Poisson(4)+1 noise peak counts, exponential(15) noise intensities, averagine masses 150–5000 u, charges 1–5
- Every synthetic peak carries one of the three ground-truth labels: signal, noise_1, or noise_2
- For reference-database use: hashing unidentified windows with the LSH collision table yields ≥ 0.6 cosine-similarity confirmed matches, e.g., 6200 of N=6902 vendor-selected DDA-PASEF precursors matched with Manhattan distance ≤ 10 in (m/z, ion-mobility apex, charge)

## Limitations

- Synthetic data cannot fully reflect the behavior of real data
- Chemical noise (repeated contaminants) is self-similar and would be misclassified as a true signal by the LSH approach trained/evaluated on these synthetics
- Signals with low relative intensity to noise peaks could be lost, as the similarity measure on which the LSH is based drops
- The paper reports no numeric threshold for generation fidelity, so validation of the synthetic output is qualitative only

## Evidence

- [results] Averagine mass/charge range: [evidence span withheld — anchor status: not_found]
- [methods] Noise sampling recipe: [evidence span withheld — anchor status: not_found]
- [results] Reference database construction: "These synthetic isotope patterns are hashed to form a reference database, which then can be used to detect isotope patterns by collision of hashes in sets of spectrum windows."
- [readme] README on generated data availability: "Generated data and code are available at https://github.com/hildebrandtlab/mzBucket"
- [discussion] Cosine similarity confirmation threshold: "When an unidentified window is now hashed, its key set can be used to select a set of candidate isotope patterns, for which cosine similarity is calculated explicitly."
