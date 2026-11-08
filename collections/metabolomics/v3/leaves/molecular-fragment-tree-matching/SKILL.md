---
name: molecular-fragment-tree-matching
description: Use when candidate molecules must be ranked against low-resolution MSn
  spectra (e.g. miniaturized ion-trap, positive-ion H+ mode) by comparing simulated
  RECAP SMILES fragment trees with a fragmentation tree built from the experimental
  peak list.
when_to_use_negative:
- Input spectra are high-resolution and high-accuracy, where deep models like MassGenie
  or SIRIUS4-based approaches are more appropriate
- Only a molecular fingerprint (as in MSNovelist) or substructure identity is needed
  rather than a full fragment-tree comparison
- Spectra were recorded in negative ion mode or with adducts other than H+, for which
  TeFT was not trained
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: TeFT
  role: Generates the candidate SMILES fragment trees that are matched against the
    MSn-derived molecular fragment tree; the matching stage ranks candidates and annotates
    peaks.
  repo: https://github.com/thumingo/TeFT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:7f920dc7d79540fe918f92db5b100c7d7cde248c22e18fbb8a8ef13c2da839ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - TeFT
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# molecular-fragment-tree-matching

## Summary

Scores the similarity between the SMILES fragment trees of TeFT-predicted candidate molecules and the molecular fragment tree derived from low-resolution tandem MSn spectra, ranking candidates to identify the tested substance.

## When to use

Use when candidate molecules must be ranked against low-resolution MSn spectra (e.g. miniaturized ion-trap, positive-ion H+ mode) by comparing simulated RECAP SMILES fragment trees with a fragmentation tree built from the experimental peak list.

## When NOT to use

- Input spectra are high-resolution and high-accuracy, where deep models like MassGenie or SIRIUS4-based approaches are more appropriate
- Only a molecular fingerprint (as in MSNovelist) or substructure identity is needed rather than a full fragment-tree comparison
- Spectra were recorded in negative ion mode or with adducts other than H+, for which TeFT was not trained

## Inputs

- SMILES fragment trees of TeFT-predicted candidate molecules
- Low-resolution tandem MSn peak list from the miniaturized mass spectrometer (up to 100 peaks by intensity)
- Fragmentation rule table (Supplementary Table S1 dissociation and rearrangement rules)

## Outputs

- Ranked list of candidate molecules by normalized SMILES fragment tree similarity (e.g. similarity_scores.csv)
- Annotated fragmentation peaks for the highest-scoring molecule

## How to apply

Parse the SMILES fragment trees of candidate molecules (max 1500 nodes, MW <500 Da, only C,H,O,N,P,S,Cl,Br,I,F) produced by the TeFT Transformer decoding stage. Build the molecular fragment tree from the low-resolution MSn peak list (top 5-6 fragmentation peaks with five nearby points, max 100 peaks by intensity) using the dissociation/rearrangement rules in Supplementary Table S1. Match the two trees node-by-node with the scoring rule table: +6/+3/+0 for parent atoms, +5/+2/+0 for fragment atoms, +4/+1.5/+0 for neutral losses. Normalize each raw score by the perfect-match score of an identical-tree comparison and rank candidates by normalized SMILES fragment tree similarity; the highest-scoring tree identifies the tested substance and annotates fragmentation peaks.

## Related tools

- **TeFT** (Generates the candidate SMILES fragment trees that are matched against the MSn-derived molecular fragment tree; the matching stage ranks candidates and annotates peaks.) — https://github.com/thumingo/TeFT

## Examples

```
python teft_match.py --peaks msn_peaks.csv --candidates smiles_trees.json --rules supplementary_table_s1.csv --out similarity_scores.csv
```

## Evaluation signals

- On the 16-flavonoid-alcohol validation set, the correct candidate for each spectrum should rank highest (top-1) in at least 8 of 16 cases, matching the reported complete-structure-prediction count.
- Each raw tree score divided by the perfect-match score (identical-tree comparison) should yield a normalized value ≤ 1.0.
- The matched node pair count should not exceed the smaller tree's node count and must respect the 1500-node SMILES-tree limit.
- All matched node pairs must use only the three tier tables: parent atoms (+6/+3/+0), fragment atoms (+5/+2/+0), neutral losses (+4/+1.5/+0).
- The top-ranked tree's neutral losses should be assignable to the experimental peak list using the Supplementary Table S1 rules, producing peak annotations consistent with the spectrum.

## Limitations

- TeFT was only trained on positive-ion-mode (H+) spectra, so this matching is unvalidated for other ionization modes.
- The fragmentation rule library (Supplementary Table S1) has no stated versioning or public release.
- Molecular weight is capped below 500 Da and only the ten elements C, H, O, N, P, S, Cl, Br, I, F are permitted.
- No quantitative performance comparison against SIRIUS4, MSNovelist, or MassGenie is provided for this matching stage.

## Evidence

- [discussion] By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked"
- [discussion] the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [discussion] we have imposed a limitation of 1500 nodes for the total count of SMILES tree nodes: "we have imposed a limitation of 1500 nodes for the total count of SMILES tree nodes"
- [discussion] constraints such as limiting the molecular weight to less than 500 Da: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability: "we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability"
- [abstract] the TeFT has been validated on a miniaturized mass spectrometer with low-resolution spectra for 16 flavonoid alcohols, achieving complete structure prediction for 8 substances: "the TeFT has been validated on a miniaturized mass spectrometer with low-resolution spectra for 16 flavonoid alcohols, achieving complete structure prediction for 8 substances"
- [readme] TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module. It is aimed to generate the specific structure of molecules de novo solely from mass spectrometry spectra: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module. It is aimed to generate the specific structure of molecules de novo solely from"
