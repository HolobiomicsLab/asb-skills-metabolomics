---
name: casmi-2017-benchmark-evaluation
description: Use when a de novo structure-prediction model must be benchmarked on
  CASMI 2017 challenge spectra and compared with SIRIUS4/MetFrag baselines using correct-structure
  rate and molecular similarity metrics.
when_to_use_negative:
- Spectra were recorded in negative ion mode or in non-H+ adduct form, since TeFT
  was trained only on positive ion mode (H+) spectra
- High-resolution spectra are the evaluation target — the authors note TeFT may not
  outperform deep models like SIRIUS4, MSNovelist, or MassGenie on high-quality resolution
  spectra
- The input is a structure library search task with known reference spectra rather
  than de novo structure generation from MS2 spectra
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: TeFT
  role: De novo structure generation model evaluated on the 93 CASMI 2017 spectra;
    generates predicted SMILES structures from MS2 spectra
  repo: github.com/thumingo/TeFT
- name: SIRIUS4
  role: 'Baseline structure-prediction method (includes CSI: FingerID) run on the
    same 93 spectra for statistical comparison'
- name: MetFrag
  role: Baseline in silico fragmentation candidate-ranking method run on the same
    93 spectra for statistical comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:8956347ff8375e20e4af9d04f043a6f9262ecc30f14497420b693b9d5a4b26cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag
  - SIRIUS
  - SIRIUS4
  - TeFT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# casmi-2017-benchmark-evaluation

## Summary

A benchmarking protocol that evaluates the TeFT de novo structure-generation model on the 93 CASMI 2017 positive-mode MS2 spectra and statistically contrasts its performance against SIRIUS4 and MetFrag using Wilcoxon signed-rank tests.

## When to use

Use when a de novo structure-prediction model must be benchmarked on CASMI 2017 challenge spectra and compared with SIRIUS4/MetFrag baselines using correct-structure rate and molecular similarity metrics.

## When NOT to use

- Spectra were recorded in negative ion mode or in non-H+ adduct form, since TeFT was trained only on positive ion mode (H+) spectra
- High-resolution spectra are the evaluation target — the authors note TeFT may not outperform deep models like SIRIUS4, MSNovelist, or MassGenie on high-quality resolution spectra
- The input is a structure library search task with known reference spectra rather than de novo structure generation from MS2 spectra

## Inputs

- 93 CASMI 2017 challenge positive-mode MS2 spectra
- Candidate structure lists for each CASMI 2017 spectrum (SMILES)
- Ground-truth structures for the 93 challenge compounds

## Outputs

- Per-spectrum predicted structures ranked by SMILES fragment tree similarity
- Aggregated metrics: % correct (ranked first), % mostly-correct, % Dice/Tanimoto similarity >0.8, % >0.9
- Wilcoxon signed-rank p-values comparing TeFT vs SIRIUS4 and TeFT vs MetFrag similarity distributions

## How to apply

Download the 93 CASMI 2017 challenge positive-mode MS2 spectra with their candidate structure lists and run TeFT on each spectrum, applying the paper's constraints: molecular weight <500 Da, only the ten elements C/H/O/N/P/S/Cl/Br/I/F, a peak list of at most 100 peaks selected by ion intensity, and the top 5–6 fragmentation peaks (with five nearby peak points) as experimental data. Rank candidate substances by comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from the tandem mass spectra. Compute Tanimoto (Dice) similarity between each predicted structure and the ground-truth structure, then aggregate the percentage of structures ranked first (correct), mostly-correct, and with similarity >0.8 and >0.9. Run SIRIUS4 and MetFrag on the same 93 spectra and apply Wilcoxon signed-rank tests on the similarity distributions to test whether TeFT differs significantly from each baseline; in the published benchmark TeFT reached 12% correct / 22% mostly correct, 78% >0.8 and 39% >0.9 Dice similarity, with p-values of 4.4×10⁻⁶ vs SIRIUS4 and 2.7×10⁻¹¹ vs MetFrag.

## Related tools

- **TeFT** (De novo structure generation model evaluated on the 93 CASMI 2017 spectra; generates predicted SMILES structures from MS2 spectra) — github.com/thumingo/TeFT
- **SIRIUS4** (Baseline structure-prediction method (includes CSI: FingerID) run on the same 93 spectra for statistical comparison)
- **MetFrag** (Baseline in silico fragmentation candidate-ranking method run on the same 93 spectra for statistical comparison)

## Evaluation signals

- All 93 CASMI 2017 spectra are processed with TeFT and each baseline under identical constraints (MW <500 Da, ten permitted elements, max 100 peaks by intensity, top 5–6 fragmentation peaks)
- Correct-structure rate (ranked first) should fall near the reported 12%, mostly-correct near 22%, Dice >0.8 near 78%, and >0.9 near 39% when reproducing the benchmark
- Wilcoxon signed-rank p-values for TeFT vs SIRIUS4 and TeFT vs MetFrag should be in the significant range reported (4.4×10⁻⁶ and 2.7×10⁻¹¹)
- Each predicted structure is compared to its ground truth via Dice/Tanimoto similarity computed on molecular fingerprints
- Candidate ranking is driven by SMILES fragment tree similarity against fragment trees derived from the tandem mass spectra

## Limitations

- TeFT was trained only on positive ion mode (H+) spectra, restricting applicability to negative-mode CASMI spectra
- The model may not show performance advantages on high-resolution spectra compared to SIRIUS4, MSNovelist, or MassGenie
- The benchmark covers only 93 spectra with molecular-weight and element constraints (<500 Da, ten elements), so results may not generalize to larger or element-diverse molecules

## Evidence

- [discussion] By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked"
- [discussion] Constraints <500 Da: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] SIRIUS4 higher mass resolution: "compared to other mass spectrometry prediction models, for instance, SIRIUS4, which includes CSI: FingerID, demands a higher level of mass resolution"
- [discussion] Positive ion mode only: "we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability"
- [readme] TeFT model description from README: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module. It is aimed to generate the specific structure of molecules de novo solely from"
