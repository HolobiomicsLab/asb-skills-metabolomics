---
name: fragment-tree-similarity-ranking
description: Use when candidate structure lists exist for low-resolution MS2 spectra
  and predicted structures must be ranked/evaluated by fragment-tree or fingerprint
  (Dice/Tanimoto) similarity against ground-truth structures.
when_to_use_negative:
- 'High-resolution, high-mass-accuracy spectra where fingerprint/foundation models
  like SIRIUS4 (with CSI: FingerID), MSNovelist, or MassGenie may have performance
  advantages'
- Spectra recorded in negative ion mode — TeFT was only trained on positive-mode (H+)
  spectra, so fragment trees derived from such spectra fall outside the model's applicability
- No candidate structure lists or no fragment tree prediction available for the spectra
  — the ranking step requires predicted structures to compare against
tools:
- name: TeFT
  role: De novo generator of predicted molecular structures and SMILES fragment trees
    from MS2 spectra; provides the predicted structures that are ranked via fragment-tree
    similarity
  repo: https://github.com/thumingo/TeFT
- name: SIRIUS4
  role: 'Baseline fragmentation-tree/CSI: FingerID method run on the identical spectra
    for Wilcoxon comparison of similarity distributions'
- name: MetFrag
  role: Baseline database-search/fragment-prediction method run on the same spectra
    set under matching candidate constraints
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:853f59373d6b7746cccfde1afb5b6bd2ad71f5c0dc726acf172e3177fee2fb09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - MetFrag
  - SIRIUS
  - SIRIUS4
  - TeFT
  - Transformer enabled Fragment Tree (TeFT)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# fragment-tree-similarity-ranking

## Summary

Ranks candidate molecular structures for an unknown compound by comparing SMILES fragment trees of predicted/annotated molecules against molecular fragment trees generated from tandem MS(n) spectra, using Tanimoto/Dice similarity. This is the ranking step that turns TeFT predictions into a concrete structure assignment.

## When to use

Use when candidate structure lists exist for low-resolution MS2 spectra and predicted structures must be ranked/evaluated by fragment-tree or fingerprint (Dice/Tanimoto) similarity against ground-truth structures.

## When NOT to use

- High-resolution, high-mass-accuracy spectra where fingerprint/foundation models like SIRIUS4 (with CSI: FingerID), MSNovelist, or MassGenie may have performance advantages
- Spectra recorded in negative ion mode — TeFT was only trained on positive-mode (H+) spectra, so fragment trees derived from such spectra fall outside the model's applicability
- No candidate structure lists or no fragment tree prediction available for the spectra — the ranking step requires predicted structures to compare against

## Inputs

- Positive-mode MS2 spectra of unknown compounds (low-resolution spectra accepted, e.g., from a miniaturized linear ion trap mass spectrometer)
- Candidate structure lists for each spectrum (with SMILES, constrained to MW <500 Da and elements C,H,O,N,P,S,Cl,Br,I,F)
- Predicted molecular structures / SMILES from a de novo model such as TeFT
- Ground-truth structures for benchmark spectra (e.g., the 93 CASMI 2017 challenge spectra, 660-spectrum test set)

## Outputs

- Per-spectrum ranked candidate list ordered by SMILES fragment-tree similarity
- Per-spectrum Tanimoto/Dice similarity scores between predicted and ground-truth structures
- Aggregated metrics table (e.g., comparison_metrics.csv): % top-1 correct, % mostly correct, % similarity >0.8, % similarity >0.9
- Wilcoxon signed-rank p-values comparing TeFT vs SIRIUS4 and vs MetFrag

## How to apply

Generate predicted structures de novo from each MS2 spectrum (e.g., with TeFT), then for each candidate compare the SMILES fragment tree of the predicted molecule with the molecular fragment tree derived from the tandem mass spectrum and rank candidates by similarity. Respect the established constraints: molecular weight <500 Da, only the ten elements C/H/O/N/P/S/Cl/Br/I/F, a maximum of 100 peaks selected by ion intensity, and the top 5–6 fragmentation peaks (with five nearby peak points) as experimental data. Compute Tanimoto similarity (Dice coefficient) between each predicted structure and the ground-truth structure per spectrum, then aggregate into percentage metrics: correct structures ranked first, most-structures-correct, similarity >0.8, and similarity >0.9. For benchmarking, run SIRIUS4 (with CSI: FingerID) and MetFrag over the identical spectra under the same candidate constraints and apply Wilcoxon signed-rank tests on the paired per-spectrum similarity distributions to test for significance (e.g., TeFT vs SIRIUS4 p=4.4×10⁻⁶, vs MetFrag p=2.7×10⁻¹¹ on CASMI 2017).

## Related tools

- **TeFT** (De novo generator of predicted molecular structures and SMILES fragment trees from MS2 spectra; provides the predicted structures that are ranked via fragment-tree similarity) — https://github.com/thumingo/TeFT
- **SIRIUS4** (Baseline fragmentation-tree/CSI: FingerID method run on the identical spectra for Wilcoxon comparison of similarity distributions)
- **MetFrag** (Baseline database-search/fragment-prediction method run on the same spectra set under matching candidate constraints)

## Examples

```
from rdkit import DataStructs; sim = DataStructs.DiceSimilarity(fp_predicted, fp_ground_truth)  # rank candidates by Dice/Tanimoto similarity
```

## Evaluation signals

- Top-1 correct structure rate falls in the reported benchmark range (12% on 93 CASMI 2017 spectra; 30%, 195/660, on the 660-spectrum test set)
- Similarity thresholds met at reported rates: >0.9 for 39% (CASMI) / 47% (test set) and >0.8 for 78% (CASMI) / 67% (test set) of spectra
- Wilcoxon signed-rank tests on paired per-spectrum similarity distributions yield significant p-values vs baselines (4.4×10⁻⁶ vs SIRIUS4, 2.7×10⁻¹¹ vs MetFrag)
- Pre-filtering constraints respected: MW <500 Da, only the ten permitted elements, peak list capped at 100 peaks by intensity, top 5–6 fragmentation peaks used
- SMILES fragment trees stay within the imposed limit of 1500 nodes

## Limitations

- TeFT was trained only on positive ion mode (H+) spectra, restricting applicability to negative-mode data
- The model may not outperform other deep models (SIRIUS4, MSNovelist, MassGenie) on high-quality, high-resolution spectra
- Fragmentation rule library (Supplementary Table S1) versioning/public availability is not specified
- No formal code availability statement for the TeFT implementation beyond the repository reference

## Evidence

- [discussion] By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked"
- [readme] TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module"
- [discussion] constraints such as limiting the molecular weight to less than 500 Da: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
