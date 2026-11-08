---
name: msms-cosine-similarity-evaluation
description: Use when a set of predicted MS/MS spectra (MGF/PKL from 3DMolMS or baselines)
  must be quantitatively scored against experimental Agilent Q-TOF PCDL/NIST20 spectra,
  per compound and per ion mode ([M+H]+, [M-H]-).
when_to_use_negative:
- Input is raw LC-MS/MS vendor data or mzML that still needs spectral library conversion
  and preprocessing — the skill assumes spectra already exist as MGF/paired test pickles
- Predictions are needed for unsupported adducts or large molecules (> 300 atoms or
  atom types outside C, O, N, H, P, S, F, Cl, B, Br, I) — 3DMolMS input support excludes
  them
- 'The question requires isotope-envelope or precursor-above peaks: predicted spectra
  contain none, so evaluation beyond fragment peaks below the precursor m/z is inappropriate'
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0091
tools:
- name: molnetpack
  role: Provides the MolNet engine used to generate predictions and to compute cosine-similarity
    evaluation (molnet_engine.evaluate) with per-compound CSV and histogram outputs
  repo: https://github.com/JosieHong/3DMolMS
- name: 3DMolMS
  role: Source of the predicted MS/MS spectra being evaluated against experimental
    Q-TOF ground truth
  repo: https://github.com/JosieHong/3DMolMS
- name: RDKit
  role: Generates fingerprints used to compute maximum Bulk Tanimoto similarity between
    each test compound and the training set
- name: SciPy
  role: Runs the Mann-Whitney U test comparing 3DMolMS's cosine-similarity distribution
    against baseline methods
- name: Pandas
  role: Assembles the per-compound table of maximum Tanimoto similarity and cosine
    similarity and saves it as CSV
- name: matplotlib
  role: Plots cosine similarity versus maximum training-set Tanimoto similarity and
    the similarity histograms
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:fc464f9217e2421a6340564f8dbec0e26d234017589f16bd135c3365e3161b25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JosieHong/3DMolMS
  tools:
  - 3DMolMS
  - CFM-ID 4.0
  - DGCNN
  - MassFormer
  - NEIMS
  - NumPy
  - Pandas
  - PointNet
  - PyTorch
  - RDKit
  - SchNet
  - SciPy
  - matplotlib
  - molnetpack
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# msms-cosine-similarity-evaluation

## Summary

Scores predicted tandem mass spectra (e.g., 3DMolMS output MGF) against experimental reference spectra using spectral cosine similarity per ion mode, and optionally correlates per-compound accuracy with maximum Bulk Tanimoto similarity to the training set.

## When to use

Use when a set of predicted MS/MS spectra (MGF/PKL from 3DMolMS or baselines) must be quantitatively scored against experimental Agilent Q-TOF PCDL/NIST20 spectra, per compound and per ion mode ([M+H]+, [M-H]-).

## When NOT to use

- Input is raw LC-MS/MS vendor data or mzML that still needs spectral library conversion and preprocessing — the skill assumes spectra already exist as MGF/paired test pickles
- Predictions are needed for unsupported adducts or large molecules (> 300 atoms or atom types outside C, O, N, H, P, S, F, Cl, B, Br, I) — 3DMolMS input support excludes them
- The question requires isotope-envelope or precursor-above peaks: predicted spectra contain none, so evaluation beyond fragment peaks below the precursor m/z is inappropriate

## Inputs

- Held-out test pickle with experimental spectra (e.g., data/qtof_all_test.pkl produced by build_msms_dataset.py --group qtof)
- Predicted spectra in MGF format (e.g., result/pred_qtof_test.mgf or ./output_msms.mgf)
- Training pickle (e.g., data/qtof_all_train.pkl) when the Tanimoto-similarity correlation analysis is included

## Outputs

- Per-compound CSV of cosine similarity values (e.g., ./eval_results.csv), optionally joined with maximum training-set Bulk Tanimoto similarity
- Cosine-similarity histogram plot (e.g., ./eval_similarity_hist.png) and cosine-vs-Tanimoto scatter plot with fitted correlation
- Aggregated mean ± standard deviation per model and ion mode, plus Mann-Whitney U test statistics

## How to apply

Load the held-out test pickle (e.g., data/qtof_all_test.pkl from build_msms_dataset.py --group qtof, split at the non-stereochemical InChIKey skeleton level) and the predicted spectra MGF (e.g., result/pred_qtof_test.mgf), then compute the cosine similarity between each predicted spectrum and its experimental ground truth using molnet_engine.evaluate() or scripts/eval.py, writing per-compound results with --result_path and a similarity histogram with --plot_path. Aggregate mean and standard deviation of cosine similarity per model and per ion mode; reference expectations for 3DMolMS on the Q-TOF test set are 0.691 (±0.361) for [M+H]+ and 0.478 (±0.379) for [M-H]-. To assess whether accuracy depends on training-set similarity, convert every training and test molecule to an RDKit fingerprint, compute each test compound's maximum Bulk Tanimoto similarity against all training compounds, and fit/plot cosine similarity versus that maximum (Supplementary Fig. S3). Use SciPy's Mann-Whitney U test to check whether one model's cosine distribution significantly exceeds another's before claiming superiority.

## Related tools

- **molnetpack** (Provides the MolNet engine used to generate predictions and to compute cosine-similarity evaluation (molnet_engine.evaluate) with per-compound CSV and histogram outputs) — https://github.com/JosieHong/3DMolMS
- **3DMolMS** (Source of the predicted MS/MS spectra being evaluated against experimental Q-TOF ground truth) — https://github.com/JosieHong/3DMolMS
- **RDKit** (Generates fingerprints used to compute maximum Bulk Tanimoto similarity between each test compound and the training set)
- **SciPy** (Runs the Mann-Whitney U test comparing 3DMolMS's cosine-similarity distribution against baseline methods)
- **Pandas** (Assembles the per-compound table of maximum Tanimoto similarity and cosine similarity and saves it as CSV)
- **matplotlib** (Plots cosine similarity versus maximum training-set Tanimoto similarity and the similarity histograms)

## Examples

```
molnet_engine.evaluate(test_pkl="./data/qtof_all_test.pkl", pred_mgf="./output_msms.mgf", result_path="./eval_results.csv", plot_path="./eval_similarity_hist.png")
```

## Evaluation signals

- Mean cosine similarity for 3DMolMS on the Q-TOF test split should be near 0.691 (±0.361) in [M+H]+ mode and 0.478 (±0.379) in [M-H]- mode
- All individual cosine similarity values must lie within [0, 1]; NaNs should be absent for every test compound
- Row count of the per-compound results CSV must equal the number of entries in the test pickle (no dropped compounds)
- Cosine similarity should trend upward with maximum Bulk Tanimoto similarity to the training set (positive correlation, per Supplementary Fig. S3)
- Mann-Whitney U test p-values must be reported for each baseline comparison; 3DMolMS was significant against all methods in positive mode and against NEIMS and CFM-ID in negative mode

## Limitations

- Predicted spectra never contain peaks above the precursor m/z — no isotope envelope is modeled, capping achievable cosine similarity for compounds with strong isotope patterns
- An E(3) encoder cannot model enantiomer differences; spectra-based cosine evaluation cannot distinguish enantiomers (SE(3) chirality mode is required for that)
- Prediction accuracy degrades for test compounds with low structural similarity to the training set, so aggregate means can mask compound-level failures
- Input constraints: atom number ≤ 300, limited atom types, and specific precursor types ([M+H]+, [M-H]-, [M+H-H2O]+, [M+Na]+, [M+2H]2+)

## Evidence

- [abstract] abstract cosine benchmark: "3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes"
- [readme] README evaluate call: "results_df = molnet_engine.evaluate(
    test_pkl="./data/qtof_all_test.pkl",
    pred_mgf="./output_msms.mgf",
    result_path="./eval_results.csv","
- [supplementary] Tanimoto correlation finding: [evidence span withheld — anchor status: not_found]
- [methods] no peaks above precursor: "Predicted spectra never contain peaks above the precursor m/z —
in particular, no isotope envelope."
- [results] statistics significance: [evidence span withheld — anchor status: not_found]
