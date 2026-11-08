---
name: gnps-spectral-library-curation
description: Use when a downloaded GNPS spectral library MGF must be cleaned into
  a uniform, filtered format (metadata normalization, peak/intensity filtering) prior
  to cross-validation splits, model training, or spectral-matching benchmarks.
when_to_use_negative:
- The input is already a curated/pickled matchms library or a pretrained MS2Query
  sqlite library folder — no re-cleaning is needed before running MS2Query
- The task is raw LC-MS/MS feature preprocessing (peak picking, clustering of MS2
  spectra per feature) — MS2Query does no peak picking or clustering; use a tool like
  MZMine first
- The goal is exact-match-only benchmarking with a 0.25 Da mass tolerance and no analogue
  search, where analogue-set construction is unnecessary
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: matchms
  role: Loads the GNPS MGF library, cleans metadata to a uniform format, and applies
    the peak/intensity filters (1000 Da cutoff, 0.1% relative intensity cutoff, 500-peak
    trim, <3-peak removal)
  repo: https://github.com/iomega/ms2query
- name: MS2Query
  role: Downstream consumer of the curated library; its benchmark (version 0.3.2)
    evaluates recall-versus-Tanimoto on the cleaned analogues test set
  repo: https://github.com/iomega/ms2query
- name: PubChem
  role: Source for filling missing InChIKeys by compound name and molecular formula
    lookup during library annotation cleanup
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-37446-4
    title: 'MS2Query: reliable and scalable MS2 mass spectra-based analogue search'
  source_content_hash: sha256:122537afe8d959b7b00ae8fd12720c09d3f262deb2961fb41e886bca42626605
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iomega/ms2query
  tools:
  - GNPS
  - MS2Query
  - PubChem
  - matchms
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# gnps-spectral-library-curation

## Summary

Clean and standardize GNPS MGF mass spectral libraries (e.g. ALL_GNPS.mgf) with matchms so spectra have uniform metadata and normalized peaks before analogue-search benchmarking or model training. Without this curation step, downstream MS2Deepscore/Spec2Vec training and modified-cosine baselines operate on inconsistent annotations and noisy peaks.

## When to use

Use when a downloaded GNPS spectral library MGF must be cleaned into a uniform, filtered format (metadata normalization, peak/intensity filtering) prior to cross-validation splits, model training, or spectral-matching benchmarks.

## When NOT to use

- The input is already a curated/pickled matchms library or a pretrained MS2Query sqlite library folder — no re-cleaning is needed before running MS2Query
- The task is raw LC-MS/MS feature preprocessing (peak picking, clustering of MS2 spectra per feature) — MS2Query does no peak picking or clustering; use a tool like MZMine first
- The goal is exact-match-only benchmarking with a 0.25 Da mass tolerance and no analogue search, where analogue-set construction is unnecessary

## Inputs

- GNPS MGF spectral library file (e.g. ALL_GNPS.mgf, downloaded 15 Nov 2021; or ALL_GNPS_NO_PROPOGATED.mgf, downloaded 1 Nov 2022)
- MS2 peak lists with precursor m/z, metadata, and intensities per spectrum

## Outputs

- Cleaned, normalized matchms spectrum objects (uniform metadata, InChIKeys completed via PubChem)
- Curated library split by ion mode and into 20 cross-validation folds
- recall_tanimoto_benchmark.csv (recall-versus-Tanimoto table) and recall_tanimoto_benchmark.png from the downstream benchmark

## How to apply

Load the GNPS library MGF (e.g. ALL_GNPS.mgf) with matchms and run the metadata cleaner to get a uniform annotation format, removing or correcting misplaced metadata; add missing InChIKeys by searching compound name and molecular formula on PubChem via a matchms extras method. Then apply the peak-level filters in order: remove peaks above 1000 Da, remove peaks with intensity below 0.1% of the highest peak in the spectrum, and for spectra with more than 500 peaks remove the lowest-intensity peaks until 500 remain. Remove spectra with fewer than 3 peaks entirely from the library and normalize intensities. Finally split the curated library into ion-mode subsets (positive/negative) and construct the analogues test set with 20 cross-validation folds, ensuring MS2Deepscore and Spec2Vec models are retrained per fold so no test-fold spectra leak into training. The rationale is that the modified-cosine and learned-similarity methods are only comparable if they search the same cleaned library.

## Related tools

- **matchms** (Loads the GNPS MGF library, cleans metadata to a uniform format, and applies the peak/intensity filters (1000 Da cutoff, 0.1% relative intensity cutoff, 500-peak trim, <3-peak removal)) — https://github.com/iomega/ms2query
- **MS2Query** (Downstream consumer of the curated library; its benchmark (version 0.3.2) evaluates recall-versus-Tanimoto on the cleaned analogues test set) — https://github.com/iomega/ms2query
- **PubChem** (Source for filling missing InChIKeys by compound name and molecular formula lookup during library annotation cleanup)

## Evaluation signals

- No spectrum in the curated library has fewer than 3 peaks, and no spectrum retains more than 500 peaks
- No peak above 1000 Da remains, and all remaining peaks have intensity ≥ 0.1% of the spectrum's highest peak
- Metadata fields are in a uniform format after matchms cleaning, and InChIKeys are present (completed via PubChem lookup where missing)
- Cross-validation folds contain no test spectra in the retrained MS2Deepscore/Spec2Vec training sets (leakage check per fold)
- Reproduced average Tanimoto at 35% recall matches the reported values (MS2Query 0.63, modified cosine 0.45) within reasonable tolerance, with MS2Query outperforming the baseline

## Limitations

- Software versions for rdkit, matchms, and scikit-learn used in the original benchmark were not stated, so exact reproducibility of cleaning behavior may vary
- It is hard to objectively judge whether performance on newly generated data matches the benchmarking test set
- Several supplementary notes (e.g. other tested features, GNPS analogue-search comparison) were not available, limiting full reproduction of the benchmark context

## Evidence

- [discussion] The dataset was first cleaned using matchms33. The metadata was cleaned to get a uniform format and to remove or correct misplaced metadata.: "The dataset was first cleaned using matchms33. The metadata was cleaned to get a uniform format and to remove or correct misplaced metadata."
- [discussion] Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed.: "Peaks above 1000 Da were removed and peaks with an intensity of less than 0,1 % of the highest peak were removed."
- [discussion] Spectra with less than 3 peaks were completely removed from the library.: "Spectra with less than 3 peaks were completely removed from the library."
- [discussion] A method from matchms extras was used to add missing InChIKeys by searching the compound name and molecular formula on PubChem.: "A method from matchms extras was used to add missing InChIKeys by searching the compound name and molecular formula on PubChem."
- [readme] A pretrained library for MS2Query is available based on the GNPS library.: [evidence span withheld — anchor status: external_doc]
- [discussion] for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models: "for all benchmarking new models for MS2Deepscore and Spec2Vec were trained to ensure that none of the test spectra were used for training these models"
