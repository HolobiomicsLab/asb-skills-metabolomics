---
name: spectra-candidate-ranking-evaluation
description: Use when you must rank candidate molecular structures for tandem mass
  spectra from a structure prediction model and compare that ranking against established
  tools (SIRIUS4, MetFrag) with statistical significance testing.
when_to_use_negative:
- When the query spectra are high-resolution from instruments like Orbitrap or FT-ICR
  — the article notes TeFT is not compared favourably there and models like MassGenie
  or MSNovelist are geared to high mass accuracy.
- When the spectra are recorded in negative ion mode — TeFT was trained only on positive-mode
  (H+) spectra, restricting its applicability.
- When candidate structures need not be ranked because the identity is already known
  — the ranking step requires an unresolved candidate list under the MW/element constraints.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TeFT
  role: Generates de novo candidate structures from low-resolution MSn spectra and
    ranks them by SMILES fragment-tree similarity to the fragmentation tree built
    from the measured spectra.
  repo: thumingo/TeFT
- name: SIRIUS4
  role: Baseline tool that predicts full structures (with CSI:FingerID) for the same
    spectra under matched candidate constraints for statistical comparison.
- name: MetFrag
  role: Baseline candidate-ranking tool applied to the identical spectrum sets with
    the same molecular-weight and element restrictions.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:c4b74275cd3f4433c6ec99eef759ca72d2018b7ba87b8d24c787322e51e4dbf8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - MetFrag
  - SIRIUS4
  - TeFT
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spectra-candidate-ranking-evaluation

## Summary

Benchmark and statistically compare spectral structure-annotation tools — e.g. TeFT against SIRIUS4 (CSI:FingerID) and MetFrag — on identical MS/MS spectrum sets under matched database constraints, reporting correct-structure and top-ranked similarity percentages plus Wilcoxon signed-rank significance.

## When to use

Use when you must rank candidate molecular structures for tandem mass spectra from a structure prediction model and compare that ranking against established tools (SIRIUS4, MetFrag) with statistical significance testing.

## When NOT to use

- When the query spectra are high-resolution from instruments like Orbitrap or FT-ICR — the article notes TeFT is not compared favourably there and models like MassGenie or MSNovelist are geared to high mass accuracy.
- When the spectra are recorded in negative ion mode — TeFT was trained only on positive-mode (H+) spectra, restricting its applicability.
- When candidate structures need not be ranked because the identity is already known — the ranking step requires an unresolved candidate list under the MW/element constraints.

## Inputs

- Tandem mass spectrometry (MS/MS or MSn) peak lists per query spectrum
- Candidate molecular structure lists (SMILES) per spectrum, restricted to MW <500 Da and the ten permitted elements
- Reference structures used to judge correct-structure hits
- Per-spectrum ranking output from each compared tool (TeFT, SIRIUS4/CSI:FingerID, MetFrag)

## Outputs

- Correct-structure percentage and top-ranked similarity percentage (>0.9, >0.8) for each tool on each spectrum set
- Wilcoxon signed-rank test p-values comparing TeFT against each baseline
- Ranked candidate list with per-candidate fragment-tree similarity scores per spectrum
- comparison_metrics.csv summarizing the percentages and p-values

## How to apply

Load the evaluation spectra (e.g. the 660-spectrum test set and the 93 CASMI 2017 spectra) together with the corresponding candidate structure lists generated under matched constraints — molecular weight <500 Da and elements limited to C, H, O, N, P, S, Cl, Br, I, F. Run each tool (TeFT, SIRIUS4 with CSI:FingerID, MetFrag) on every spectrum with the same candidate database restrictions so that rankings are directly comparable. For each tool compute the percentage of spectra whose top-ranked candidate is the correct structure, and the percentage whose top-ranked candidate has similarity >0.9 and >0.8 to the reference (TeFT reported 30% correct, 47% >0.9, 67% >0.8; SIRIUS4 reported 27.6% full structures and 39% >0.9). Apply Wilcoxon signed-rank tests to the paired per-spectrum scores of each tool against TeFT to establish significance (Table 1 statistics).

## Related tools

- **TeFT** (Generates de novo candidate structures from low-resolution MSn spectra and ranks them by SMILES fragment-tree similarity to the fragmentation tree built from the measured spectra.) — thumingo/TeFT
- **SIRIUS4** (Baseline tool that predicts full structures (with CSI:FingerID) for the same spectra under matched candidate constraints for statistical comparison.)
- **MetFrag** (Baseline candidate-ranking tool applied to the identical spectrum sets with the same molecular-weight and element restrictions.)

## Examples

```
python run_benchmark.py --spectra test_660.spectra --tool TeFT --tool SIRIUS4 --tool MetFrag --mw_max 500 --elements C,H,O,N,P,S,Cl,Br,I,F --out comparison_metrics.csv
```

## Evaluation signals

- Correct-structure percentage should match the reported values (TeFT 195/660 ≈ 30%; SIRIUS4 27.6% full structures on the 660-spectrum test set).
- Top-ranked similarity percentages should fall within the reported ranges (TeFT: 47% >0.9, 67% >0.8; SIRIUS4: 39% >0.9).
- Wilcoxon signed-rank p-values must be reproducible when comparing paired per-spectrum results of TeFT vs. SIRIUS4 and TeFT vs. MetFrag.
- Every query spectrum must yield a ranked candidate list for all tools under identical MW <500 Da and ten-element constraints — mismatches indicate inconsistent candidate sets.
- On the 16 flavonoid alcohols, the ranking should recover complete structures for 8 of 16 substances.

## Limitations

- TeFT is trained only on positive ion mode (H+) spectra, so negative-mode spectra fall outside its validated scope.
- The article does not report a quantitative head-to-head performance comparison with SIRIUS4/MSNovelist/MassGenie for high-resolution spectra, acknowledging the model may not be advantageous there.
- Candidate enumeration is capped by the study's constraints (MW <500 Da, ten elements), so rankings are only valid within that chemical search space.
- The TeFT repository README provides no usage documentation, so reproducing the exact ranking pipeline depends on the source code itself.

## Evidence

- [results] On the 660-spectrum test set, TeFT correctly identified 30% of structures (195/660) with all similarity >0.9 at 47% and >0.8 at 67%, while SIRIUS4 predicted 27.6% of full structures and 39% with similarity >0.9.: [evidence span withheld — anchor status: not_found]
- [discussion] the training set standardized the SMILES representations for all molecules and removed duplicate entries: "the training set standardized the SMILES representations for all molecules and removed duplicate entries"
- [discussion] constraints such as limiting the molecular weight to less than 500 Da: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked"
- [discussion] SIRIUS4, which includes CSI: FingerID, demands a higher level of mass resolution: "SIRIUS4, which includes CSI: FingerID, demands a higher level of mass resolution"
