---
name: predicted-vs-groundtruth-scatter-visualization
description: Use when a supervised or heuristic spectral similarity metric must be
  visually benchmarked against a structural ground truth (Daylight Tanimoto) on MS/MS
  query–reference pairs, and a scatter point cloud plus Pearson r are the reporting
  format.
when_to_use_negative:
- The input pairs lack any structural ground truth (e.g., unknown or non-molecular
  spectra), so no Tanimoto score can anchor the scatter plot
- The task is evaluating retrieval/analogue search performance or molecular networking
  rather than correlation of a similarity score with structural similarity
- Only a single method's scores exist without the baselines (Cosine, Modified Cosine,
  Spec2Vec, MS2DeepScore) needed for the comparative scatter benchmark
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TransExION
  role: Produces the predicted spectral similarity scores plotted against Tanimoto
    ground truth via ms_test_spectral_similarity.py
  repo: github.com/banhdzui/TransExION
- name: RDKit
  role: Computes Tanimoto scores on Daylight fingerprints as the ground-truth structural
    similarity for each pair
- name: Matplotlib
  role: Renders the predicted-vs-Tanimoto point-cloud scatter plots (Fig. 7)
- name: MS2DeepScore
  role: Baseline deep-learning spectral similarity whose correlation with Tanimoto
    is visualized for comparison
- name: Spec2Vec
  role: Unsupervised baseline spectral similarity whose correlation with Tanimoto
    is visualized for comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-024-00858-5
    title: 'TransExION: a transformer based explainable similarity metric for comparing
      IONS in tandem mass spectrometry'
  source_content_hash: sha256:73b1163ef47e192d0f582662e905655ac3d4a51e269f9786d30b70fc3ffec8bd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MS2DeepScore
  - Matplotlib
  - PyTorch
  - RDKit
  - Spec2Vec
  - TransExION
  - pyteomics
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# predicted-vs-groundtruth-scatter-visualization

## Summary

Generates predicted-vs-Tanimoto scatter plots for spectral similarity methods (e.g., TransExION, Cosine, Modified Cosine, Spec2Vec, MS2DeepScore) on the mergedGNPS testing pairs, using binned pair sampling and Pearson correlation to visualize how well each predicted score tracks structural similarity.

## When to use

Use when a supervised or heuristic spectral similarity metric must be visually benchmarked against a structural ground truth (Daylight Tanimoto) on MS/MS query–reference pairs, and a scatter point cloud plus Pearson r are the reporting format.

## When NOT to use

- The input pairs lack any structural ground truth (e.g., unknown or non-molecular spectra), so no Tanimoto score can anchor the scatter plot
- The task is evaluating retrieval/analogue search performance or molecular networking rather than correlation of a similarity score with structural similarity
- Only a single method's scores exist without the baselines (Cosine, Modified Cosine, Spec2Vec, MS2DeepScore) needed for the comparative scatter benchmark

## Inputs

- Predicted spectral similarity scores per query–reference pair for each method (TransExION, Cosine, Modified Cosine, Spec2Vec, MS2DeepScore), e.g., from ms_test_spectral_similarity.py CSV output
- Tanimoto structural similarity scores on Daylight fingerprints per pair, computed with RDKit via ms_compute_structural_similarity.py
- mergedGNPS query (testing) and reference (training) MS/MS spectra in MGF/h5py format
- Spectrum pair id file (h5py, optionally converted from CSV via ms_covert_csv2h5py.py) selecting the evaluated pairs

## Outputs

- Predicted-vs-Tanimoto scatter plot / point cloud per method (Fig. 7-style)
- Pearson correlation coefficient r per method over all mergedGNPS testing pairs
- similarity_scores.csv
- correlation_results.csv

## How to apply

For each method, load the per-pair predicted spectral similarity scores and the RDKit-computed Tanimoto scores on Daylight fingerprints for the same spectrum pairs. Apply the paper's binned evaluation protocol: equal-width binning into 10 bins between 0.0 and 1.0 on structural similarity, selecting at most k=3 pairs per bin per testing spectrum (yielding 11,425 mergedGNPS testing pairs). Plot the predicted similarity against the Tanimoto ground truth as a point cloud per method with Matplotlib, so tighter, more diagonal clouds indicate better agreement. Compute the Pearson correlation coefficient r between predicted and Tanimoto scores over all mergedGNPS testing pairs and annotate/report it per method. Rationale: the binned subsampling prevents redundant near-duplicate pairs from dominating the visualization, and the scatter plus r jointly show both calibration and noise of each metric. Save similarity_scores.csv and correlation_results.csv alongside the figure.

## Related tools

- **TransExION** (Produces the predicted spectral similarity scores plotted against Tanimoto ground truth via ms_test_spectral_similarity.py) — github.com/banhdzui/TransExION
- **RDKit** (Computes Tanimoto scores on Daylight fingerprints as the ground-truth structural similarity for each pair)
- **Matplotlib** (Renders the predicted-vs-Tanimoto point-cloud scatter plots (Fig. 7))
- **MS2DeepScore** (Baseline deep-learning spectral similarity whose correlation with Tanimoto is visualized for comparison)
- **Spec2Vec** (Unsupervised baseline spectral similarity whose correlation with Tanimoto is visualized for comparison)

## Examples

```
python ms_test_spectral_similarity.py --db GNPS_MassBank_train.db --query GNPS_MassBank_test.db --model GNPS_MassBank.ms.model --output GNPS_MassBank_result.csv
```

## Evaluation signals

- Pearson r on mergedGNPS should reach ~0.811 for TransExION versus 0.489 (Cosine), 0.538 (Modified Cosine), 0.557 (Spec2Vec), and 0.755 (MS2DeepScore)
- Pair set after binning must contain 11,425 mergedGNPS testing pairs (10 equal-width bins over [0.0, 1.0], at most k=3 pairs per bin per testing spectrum)
- Predicted scores and predicted-vs-Tanimoto scatter values must lie within [0.0, 1.0] and use the same pair identifiers as the Tanimoto ground-truth file
- TransExION's point cloud should be tighter and more diagonal than the baselines', with less widespread scatter, consistent with the higher r
- similarity_scores.csv and correlation_results.csv must contain one row per evaluated pair/method and one r per method, respectively

## Limitations

- Correlation depends on the binned subsampling protocol (10 bins, k=3 pairs per bin per testing spectrum); using all pairs or different binning changes the reported r and point cloud
- The evaluation covers mergedGNPS testing data with Daylight-fingerprint Tanimoto as ground truth; transfer to other test sets or fingerprint types is not established here
- The article's discussion reports the outperformance claim without quantitative metrics, so the r values used here come from the task workflow rather than the discussion text

## Evidence

- [discussion] TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model.: "TransExION outperforms simple, heuristic similarity measures, such as Cosine and Modified Cosine, and the unsupervised Spec2Vec model"
- [readme] Supervised model predicting spectral similarity correlated to structural similarity: "TransExION is a supervised learning method estimating ***spectral similarity*** between MS/MS spectra that are strongly correlated to their structural similarity."
- [readme] h5py conversion of large-scale input data: "To work with large scale dataset, the input data files are converted into h5py format. Our library supports a function to convert a MGF format file into h5py format."
- [readme] Tooling for computing spectral similarity used to build the scatter data: "To compute spectral similarity between spectra using a trained TransExION model, the script ```ms_test_spectral_similarity.py``` can be used with parameters"
