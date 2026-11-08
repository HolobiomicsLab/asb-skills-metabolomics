---
name: spectral-cosine-similarity-evaluation
description: Use when you have predicted MS/MS spectra (e.g. MGF from 3DMolMS pred_msms)
  paired with experimental test spectra (PKL/MGF) and need per-compound cosine similarity
  scores stratified by ion mode or collision energy.
when_to_use_negative:
- Comparing spectra where predicted peaks include an isotope envelope or peaks above
  the precursor m/z — 3DMolMS predictions contain neither, so a raw cosine against
  spectra with such peaks conflates modeling artifacts
- Computing instrument-level benchmark scores for non-Agilent data without first excluding
  compounds overlapping the PCDL/NIST20 training libraries — the evaluation set must
  be disjoint from training data
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: molnetpack
  role: Provides the MolNet.evaluate engine that pairs predicted MGF with experimental
    PKL spectra and computes per-compound cosine similarities
  repo: https://github.com/JosieHong/3DMolMS
- name: 3DMolMS
  role: Generates the predicted MS/MS spectra (MGF) that are scored against experimental
    spectra
  repo: https://github.com/JosieHong/3DMolMS
- name: Pandas
  role: Aggregates per-compound cosine similarities into stratified summary CSVs (ion
    mode, collision energy)
- name: matplotlib
  role: Produces similarity histograms and 20 eV vs 40 eV comparison plots from the
    summary CSV
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad354
    title: '3DMolMS: prediction of tandem mass spectra from 3D molecular conformations'
  source_content_hash: sha256:9b23ccabd04a356f7057e7a38a03492527fa5cad26d5468c293273c0ffd76f9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JosieHong/3DMolMS
  tools:
  - 3DMolMS
  - NumPy
  - Pandas
  - PyTorch
  - Python+
  - RDKit
  - matplotlib
  - molnetpack
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# spectral-cosine-similarity-evaluation

## Summary

Evaluates predicted MS/MS spectra against experimental spectra by computing per-compound cosine similarities, as used by 3DMolMS to benchmark predictions in positive/negative ion modes and across collision energies.

## When to use

Use when you have predicted MS/MS spectra (e.g. MGF from 3DMolMS pred_msms) paired with experimental test spectra (PKL/MGF) and need per-compound cosine similarity scores stratified by ion mode or collision energy.

## When NOT to use

- Comparing spectra where predicted peaks include an isotope envelope or peaks above the precursor m/z — 3DMolMS predictions contain neither, so a raw cosine against spectra with such peaks conflates modeling artifacts
- Computing instrument-level benchmark scores for non-Agilent data without first excluding compounds overlapping the PCDL/NIST20 training libraries — the evaluation set must be disjoint from training data

## Inputs

- Experimental MS/MS test spectra as PKL records with compound titles, precursor types, and collision-energy labels (e.g. qtof_all_test.pkl)
- Predicted MS/MS spectra in MGF format from 3DMolMS (output_msms.mgf from MolNet.pred_msms)

## Outputs

- Per-compound cosine similarity table (eval_results.csv)
- Similarity histogram plot (eval_similarity_hist.png)
- Stratified summary CSV (e.g. eval_pcdl_energy_summary.csv by 20 eV vs 40 eV and ion mode) and comparison plot (eval_pcdl_energy.png)

## How to apply

Load the experimental test spectra as a PKL (e.g. qtof_all_test.pkl with precursor and collision-energy labels) and the predicted MGF produced by MolNet.pred_msms. Pair each predicted MGF ion block to its experimental spectrum by compound title using the molnetpack MolNet.evaluate engine (or scripts/eval.py), which computes cosine similarity between the predicted and experimental peak lists per compound. Aggregate similarities with Pandas and stratify by positive/negative ion mode and by collision energy (20 eV vs 40 eV) to reproduce paper-style benchmark tables. Write a summary CSV and optionally a similarity histogram or matplotlib comparison plot. Correctness is judged by mean cosine similarity falling in the expected range for the model/instrument combination (e.g. 0.691 positive / 0.478 negative for 3DMolMS on PCDL/NIST20-like test data, and higher similarity at 40 eV than 20 eV in both ion modes).

## Related tools

- **molnetpack** (Provides the MolNet.evaluate engine that pairs predicted MGF with experimental PKL spectra and computes per-compound cosine similarities) — https://github.com/JosieHong/3DMolMS
- **3DMolMS** (Generates the predicted MS/MS spectra (MGF) that are scored against experimental spectra) — https://github.com/JosieHong/3DMolMS
- **Pandas** (Aggregates per-compound cosine similarities into stratified summary CSVs (ion mode, collision energy))
- **matplotlib** (Produces similarity histograms and 20 eV vs 40 eV comparison plots from the summary CSV)

## Examples

```
molnet_engine.evaluate(test_pkl="./data/qtof_all_test.pkl", pred_mgf="./output_msms.mgf", result_path="./eval_results.csv", plot_path="./eval_similarity_hist.png")
```

## Evaluation signals

- Average cosine similarity in positive mode should reach the model's benchmark range (e.g. 0.691 for 3DMolMS on the main test set; 0.506 for fine-tuned MoNA Q-TOF)
- Negative-mode mean cosine similarity should be lower than positive mode but within reported ranges (e.g. 0.478 main; 0.430 fine-tuned MoNA)
- The 40 eV collision-energy stratum should show a higher mean cosine similarity than the 20 eV stratum in both ion modes (Table 3 pattern)
- Every predicted MGF ion block must be paired to exactly one experimental spectrum by compound title, with no unpaired leftovers
- Results CSV should contain a similarity value per compound and per-ion-mode summary rows written by evaluate()

## Limitations

- Cosine similarity depends on matched precursor and acquisition conditions: predictions made with the QTOF checkpoint are trained under METLIN-SMRT-like conditions, so evaluating spectra from other instruments requires fine-tuning (e.g. on MoNA Q-TOF subsets)
- Predicted spectra never contain peaks above the precursor m/z and no isotope envelope, which caps similarity for experimental spectra containing such peaks
- Baseline comparisons can be very poor: CFM-ID achieved only 0.076 (positive) / 0.291 (negative) on MoNA Q-TOF data, so cosine similarity without a reference model can be misleading

## Evidence

- [abstract] 3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively: "3DMolMS predicted the spectra with the average cosine similarity of 0.691 and 0.478 with the experimental MS/MS spectra acquired in positive and negative ion modes, respectively"
- [readme] results_df = molnet_engine.evaluate(test_pkl, pred_mgf, result_path, plot_path): "results_df = molnet_engine.evaluate(
    test_pkl="./data/qtof_all_test.pkl",
    pred_mgf="./output_msms.mgf",
    result_path="./eval_results.csv",
    plot_path="./eval_similarity_hist.png",
)"
- [methods] Predicted spectra never contain peaks above the precursor m/z — in particular, no isotope envelope.: "Predicted spectra never contain peaks above the precursor m/z —
in particular, no isotope envelope."
