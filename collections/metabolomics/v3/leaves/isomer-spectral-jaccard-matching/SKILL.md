---
name: isomer-spectral-jaccard-matching
description: Use when MS/MS spectral matching produces ambiguous or highest-score-tied
  identifications for isomers (e.g., leucine vs isoleucine) and multi-collision-energy
  spectra are available to compute longitudinal fragment profiles.
when_to_use_negative:
- Only a single-collision-energy (stepped NCE) spectrum is available with no multi-CE
  acquisition — longitudinal Jaccard requires LFPs across 13 distinct CEs; cross-sectional
  algorithms like entropy similarity are the fallback.
- The analytical goal is general metabolite annotation against public databases (MoNA,
  GNPS, METLIN) rather than discriminating candidate isomers with highly similar substructures.
- Input spectra have not passed spectrum filtering, invalid fragment removal, and
  fragment rationality assessment — the upstream preprocessing from 001-007.ipynb
  is a prerequisite.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Jaccard similarity algorithm
  role: The core similarity computation, optimized to treat predominant and minute
    LFP fragments independently when scoring isomer pairs
- name: Python 3.8.13
  role: Runtime environment (conda env 'longprof') for computing Jaccard scores on
    LFP inputs and writing the score CSV
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: Jupyter Notebook
  role: Notebooks 001-007.ipynb execute isomer similarity calculations on CQMU database
    entries and write lfp_similarity_scores.csv
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- name: mzbatch.py
  role: Library of mass spectrometry data processing methods (file operations, data
    calculation, data extraction) supporting LFP construction and scoring
  repo: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c01414
    title: Longitudinal Fragment Profiles Based on Multi-Collision Energy Tandem Mass
      Spectra Improve the Accuracy of Metabolite Identification in Untargeted Metabolomics
  source_content_hash: sha256:639d57df91e3167d7817a05382a834d0e55af1ce509ebf665346c37a4464283e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
  tools:
  - Jaccard similarity algorithm
  - Jupyter Notebook
  - Python
  - Python 3.8.13
  - mzbatch.py
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 24
---

# isomer-spectral-jaccard-matching

## Summary

An optimized Jaccard similarity algorithm for matching MS/MS spectra of isomers by comparing longitudinal fragment ion-intensity profiles (LFPs) across multiple collision energies, treating predominant and minute fragments independently. This differentiates isomers like leucine/isoleucine that cross-sectional spectral matching cannot separate.

## When to use

Use when MS/MS spectral matching produces ambiguous or highest-score-tied identifications for isomers (e.g., leucine vs isoleucine) and multi-collision-energy spectra are available to compute longitudinal fragment profiles.

## When NOT to use

- Only a single-collision-energy (stepped NCE) spectrum is available with no multi-CE acquisition — longitudinal Jaccard requires LFPs across 13 distinct CEs; cross-sectional algorithms like entropy similarity are the fallback.
- The analytical goal is general metabolite annotation against public databases (MoNA, GNPS, METLIN) rather than discriminating candidate isomers with highly similar substructures.
- Input spectra have not passed spectrum filtering, invalid fragment removal, and fragment rationality assessment — the upstream preprocessing from 001-007.ipynb is a prerequisite.

## Inputs

- Chained LFP inputs: fragment ion-intensity profiles of query and reference isomers across 13 collision energies plus the stepped NCE 20/40/60% combined spectrum
- Paired experimental and reference MS/MS spectra for candidate isomer pairs
- Python 3.8.13 environment with requirements.txt dependencies (Jupyter Notebook, mzbatch.py library)

## Outputs

- Per-fragment, per-algorithm similarity score table (lfp_similarity_scores.csv)
- Jaccard similarity scores distinguishing isomer pairs (≥ 0.9 for similar LFPs, < 0.1 for different LFPs)
- Cross-sectional benchmark scores from dot product, modified dot product, and entropy algorithms

## How to apply

Construct longitudinal fragment profiles (LFPs) from MS/MS spectra acquired at 13 distinct collision energies plus the stepped NCE 20/40/60% combined spectrum, extracting relative ion intensities of each fragment m/z across the energies. For each fragment, compute the optimized Jaccard similarity (equation 1 of the paper) between the query and reference LFPs at each individual collision energy, and repeat the computation on the stepped NCE 20/40/60% combined spectrum to obtain the cross-sectional comparison. The optimization treats predominant and minute fragments independently, so subtle differences carried by minute fragments are not overwhelmed by predominant peaks. Benchmark against classic dot product, modified dot product, and entropy similarity on the stepped combined spectrum to confirm cross-sectional methods fail to distinguish the isomers. Correct application is judged by whether visually similar LFPs receive Jaccard scores ≥ 0.9 and visually different LFPs receive scores < 0.1, as observed for leucine/isoleucine among 19 algorithms tested.

## Related tools

- **Jaccard similarity algorithm** (The core similarity computation, optimized to treat predominant and minute LFP fragments independently when scoring isomer pairs)
- **Python 3.8.13** (Runtime environment (conda env 'longprof') for computing Jaccard scores on LFP inputs and writing the score CSV) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **Jupyter Notebook** (Notebooks 001-007.ipynb execute isomer similarity calculations on CQMU database entries and write lfp_similarity_scores.csv) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git
- **mzbatch.py** (Library of mass spectrometry data processing methods (file operations, data calculation, data extraction) supporting LFP construction and scoring) — https://github.com/yourusername/LongitudinalProf-MSMS-Method.git

## Examples

```
conda create -n longprof python=3.8.13; conda activate longprof; pip install -r requirements.txt; jupyter notebook 001-007.ipynb  # run cells through isomer similarity calculations to produce lfp_similarity_scores.csv
```

## Evaluation signals

- Jaccard scores for visually similar LFPs (e.g., leucine/isoleucine at matching fragments) must be ≥ 0.9.
- Jaccard scores for visually different LFPs must be < 0.1, confirming discrimination where cross-sectional algorithms fail.
- Benchmark dot product, modified dot product, and entropy similarity on the stepped NCE 20/40/60% spectrum should yield near-identical scores for both isomers, demonstrating the expected failure of cross-sectional evaluation.
- The output CSV (lfp_similarity_scores.csv) must contain scores for every fragment m/z at each of the 13 individual collision energies plus the stepped combined spectrum, for all 19 algorithms evaluated.
- Scores should replicate the paper's finding that Jaccard was the best of 19 algorithms at capturing LFP differences between leucine and isoleucine.

## Limitations

- Requires MS/MS data acquired at 13 distinct collision energies plus stepped NCE 20/40/60% — not applicable to single-CE legacy spectra.
- The in-house databases (CQMD with ~180,000 MS/MS spectra of over 4500 metabolites, and CQMU) are described as in-house with no public accession stated; replication may require reconstructing the reference library.
- No changelog was found for the repository, and results referenced in supplementary artifacts (Figures 5e–h, S6, Excel S9–S13) have availability not confirmed in the discussion section.
- The method's identification criteria were established by ROC analysis on the CQMD isomer database; thresholds may not transfer to other instruments or libraries without re-calibration.

## Evidence

- [results] Jaccard best captured LFP differences: [evidence span withheld — anchor status: not_found]
