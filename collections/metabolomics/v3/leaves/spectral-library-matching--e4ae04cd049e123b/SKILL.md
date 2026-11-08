---
name: spectral-library-matching
description: Use when raw or centroided MS/MS spectra must be identified by searching
  against a spectral library, and score-threshold/FDR behavior of the similarity metric
  matters.
when_to_use_negative:
- Input is already an annotated feature/compound table rather than MS/MS peak lists
  — no spectral matching is needed
- Comparing uncleaned spectra without precursor m/z available — the clean_spectrum
  precursor-ions-removal step (precursor_mz - 1.6 Da) and identity constraint cannot
  be applied
- Spectra were pre-processed with power transforms at w < 1 on ion-rich spectra with
  high entropy values — the article reports decreased similarity scoring performance
  in that regime
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ms_entropy
  role: Python package providing clean_spectrum, calculate_entropy_similarity, read_one_spectrum,
    and FlashEntropySearch for spectral cleaning and similarity scoring
  repo: https://github.com/YuanyueLi/MSEntropy
- name: MSEntropy
  role: Multi-language (Python/R/C/C++/JavaScript) package with Flash entropy search
    for accelerated entropy-similarity library search
  repo: https://github.com/YuanyueLi/MSEntropy
- name: SpectralEntropy
  role: Original source code for the 43 similarity algorithms benchmarked in the paper
  repo: https://github.com/YuanyueLi/SpectralEntropy
- name: NumPy
  role: Array representation of peaks as 2D [m/z, intensity] arrays passed to ms_entropy
    functions
- name: DynamicEntropySearch
  role: Indexed library construction/update for unlimited library spectra with fast
    entropy-similarity search
  repo: https://github.com/2bereal-me/DynamicEntropySearch
- name: SciPy
  role: 'Alternative: the entropy function from scipy.stats can compute unweighted
    entropy similarity directly'
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s41592-021-01331-z
    title: Spectral entropy outperforms MS/MS dot product similarity for small-molecule
      compound identification
  source_content_hash: sha256:0c74165ee4c115801964bb164a8d4753756ea9864054363baf0305cf3775e0c0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/YuanyueLi/MSEntropy
  tools:
  - DynamicEntropySearch
  - MSEntropy
  - NumPy
  - Python
  - Python 3.7.9
  - R 3.6.3
  - SciPy
  - SpectralEntropy
  - ms_entropy
  - multiprocessing
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5909090909090909
n_claims: 22
---

# spectral-library-matching

## Summary

Matching MS/MS query spectra against spectral libraries (e.g., NIST20, MoNA/Massbank.us, GNPS) using entropy similarity instead of dot product cosine, with precursor-mz identity constraints and score-threshold/FDR evaluation for small-molecule identification.

## When to use

Use when raw or centroided MS/MS spectra must be identified by searching against a spectral library, and score-threshold/FDR behavior of the similarity metric matters.

## When NOT to use

- Input is already an annotated feature/compound table rather than MS/MS peak lists — no spectral matching is needed
- Comparing uncleaned spectra without precursor m/z available — the clean_spectrum precursor-ions-removal step (precursor_mz - 1.6 Da) and identity constraint cannot be applied
- Spectra were pre-processed with power transforms at w < 1 on ion-rich spectra with high entropy values — the article reports decreased similarity scoring performance in that regime

## Inputs

- MS/MS query spectra (.mgf, .msp, .mzML) from NIST20, MassBank of North America (MoNA/Massbank.us), GNPS, or experimental metabolomics runs
- Spectral library reference spectra as dicts with 'precursor_mz' (float) and 'peaks' (2D np.ndarray of [m/z, intensity], dtype float32)
- Precursor m/z values per spectrum for identity constraint (ms1_tolerance_in_da ~ 0.01)

## Outputs

- Per-query top-N matches with entropy similarity scores (e.g., topn=3, min_similarity=0.01)
- Similarity-score threshold vs. FDR/TPR tables (e.g., similarity_benchmark_fdr_tpr.csv, fdr_threshold_analysis.csv)
- Compound annotations at entropy similarity score 0.75 plus flags for poor-quality, noisy spectra

## How to apply

Format each spectrum as a dict with 'precursor_mz' (float) and 'peaks' (2D array of [m/z, intensity]), e.g., loaded via ms_entropy.read_one_spectrum from .mgf, .msp, or .mzML. Clean every spectrum with ms_entropy.clean_spectrum: remove empty peaks, remove ions above precursor_mz - 1.6 Da (shown to improve identification on NIST20 and Massbank.us), centroid peaks within +/- min_ms2_difference_in_da (0.05 Da), drop peaks below noise_threshold * max intensity (0.01, i.e., <1% base peak), retain only the top max_peak_num peaks, and normalize intensities to sum to 1. Compute entropy similarity with ms_entropy.calculate_entropy_similarity (or FlashEntropySearch/DynamicEntropySearch for large libraries) at ms2 tolerance 0.05 Da, and optionally compare against dot product and other algorithms (43 total were benchmarked in Python 3.7.9 with fragment matching at 0.05 Da tolerance). Restrict matches by precursor identity (ms1_tolerance_in_da ~ 0.01), retrieve top-N matches (e.g., topn=3, min_similarity=0.01), and annotate at an entropy similarity score threshold of 0.75. Evaluate by sweeping score thresholds, classifying matches as true/false identifications, and computing FDR/TPR curves; FDR below 10% is expected at score 0.75 on experimental natural-product and gut metabolome spectra.

## Related tools

- **ms_entropy** (Python package providing clean_spectrum, calculate_entropy_similarity, read_one_spectrum, and FlashEntropySearch for spectral cleaning and similarity scoring) — https://github.com/YuanyueLi/MSEntropy
- **MSEntropy** (Multi-language (Python/R/C/C++/JavaScript) package with Flash entropy search for accelerated entropy-similarity library search) — https://github.com/YuanyueLi/MSEntropy
- **SpectralEntropy** (Original source code for the 43 similarity algorithms benchmarked in the paper) — https://github.com/YuanyueLi/SpectralEntropy
- **NumPy** (Array representation of peaks as 2D [m/z, intensity] arrays passed to ms_entropy functions)
- **DynamicEntropySearch** (Indexed library construction/update for unlimited library spectra with fast entropy-similarity search) — https://github.com/2bereal-me/DynamicEntropySearch
- **SciPy** (Alternative: the entropy function from scipy.stats can compute unweighted entropy similarity directly)

## Examples

```
import ms_entropy as me; clean = me.clean_spectrum({'precursor_mz': 150.0, 'peaks': peaks}, precursor_ions_removal_da=1.6, noise_threshold=0.01, min_ms2_difference_in_da=0.05); score = me.calculate_entropy_similarity(clean_query, clean_ref, ms2_tolerance_in_da=0.05)
```

## Evaluation signals

- Cleaned spectra have intensities normalized to sum = 1 and no peaks above precursor_mz - 1.6
- In the NIST20 benchmark (434,287 spectra), entropy similarity should outperform all 42 alternative similarity algorithms including dot product similarity on FDR/TPR curves
- On 37,299 experimental natural-product spectra, FDR must fall below 10% at entropy similarity score 0.75
- All computed FDR values lie between 0 and 1; the baseline entropy-similarity curve reproduces the 0.75-score acceptance criterion
- Top-N matches respect precursor identity constraint (ms1_tolerance_in_da = 0.01) and min_similarity filter of 0.01

## Limitations

- Even with entropy similarity scores, no single similarity threshold can be defined that would prevent false-positive discoveries
- Dot product similarity is more affected by noise ions, whereas entropy similarity is robust — but benchmark results depend on the 0.05 Da fragment-matching tolerance and cleaning parameters
- ENTACT study data for overall compound identification schemes were not yet publicly available for benchmarking
- Cleaning is mandatory before search: lack of cleaning in DynamicEntropySearch workflows can lead to errors

## Evidence

- [readme] README required spectrum keys: [evidence span withheld — anchor status: external_doc]
