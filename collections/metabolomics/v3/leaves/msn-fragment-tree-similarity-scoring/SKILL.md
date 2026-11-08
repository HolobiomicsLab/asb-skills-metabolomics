---
name: msn-fragment-tree-similarity-scoring
description: Use when you have multiple candidate SMILES structures from a de novo
  prediction step and a low-resolution MS2/MSn peak list, and need to rank candidates
  by comparing simulated fragmentation trees against the tree built from experimental
  peaks.
when_to_use_negative:
- When only a single candidate molecule exists and no ranking is needed — direct identification
  without tree comparison suffices.
- When spectra are high-resolution and high mass accuracy — the article notes TeFT
  'may not exhibit exceptional performance advantages in predicting high-quality resolution
  spectra compared to other deep models' like SIRIUS4 or MSNovelist.
- When spectra were recorded in negative ion mode — the training set was compiled
  only from spectra recorded in positive ion mode (H+), restricting applicability.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: TeFT
  role: Overall pipeline that combines the Transformer candidate generation with fragmentation
    tree similarity scoring for de novo structure prediction from low-resolution MSn
    spectra.
  repo: https://github.com/thumingo/TeFT
- name: Transformer
  role: Generates candidate complete SMILES structures from MS2 spectra whose fragment
    trees are then scored.
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:eab2f8f8526a2fc0ec195d82e13ed7aff988c36408486c1300a7ea912369280b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - GNPS
  - HMDB (5.0)
  - MoNA
  - TeFT
  - Transformer
  - Transformer enabled Fragment Tree (TeFT)
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# msn-fragment-tree-similarity-scoring

## Summary

Scores the similarity between a candidate molecule's simulated SMILES fragment tree and the fragment tree generated from an experimental low-resolution MSn spectrum, then ranks candidate molecules by that normalized score to identify the tested substance. This is the ranking stage of the TeFT de novo structure prediction pipeline.

## When to use

Use when you have multiple candidate SMILES structures from a de novo prediction step and a low-resolution MS2/MSn peak list, and need to rank candidates by comparing simulated fragmentation trees against the tree built from experimental peaks.

## When NOT to use

- When only a single candidate molecule exists and no ranking is needed — direct identification without tree comparison suffices.
- When spectra are high-resolution and high mass accuracy — the article notes TeFT 'may not exhibit exceptional performance advantages in predicting high-quality resolution spectra compared to other deep models' like SIRIUS4 or MSNovelist.
- When spectra were recorded in negative ion mode — the training set was compiled only from spectra recorded in positive ion mode (H+), restricting applicability.

## Inputs

- RECAP-based SMILES fragment trees of candidate molecules from the TeFT Transformer output
- Experimental low-resolution MSn (MS2) peak list from a miniaturized mass spectrometer, with top 5-6 fragmentation peaks and their five nearby points
- Peak list capped at 100 peaks selected by ion intensity
- Fragmentation rules from Supplementary Table S1

## Outputs

- Normalized similarity scores for each candidate molecule (similarity_scores.csv)
- Ranked list of candidate molecules by SMILES fragment tree similarity
- Top-ranked molecule identified as the tested substance
- Annotated fragmentation peaks derived from the highest-scoring SMILES fragment tree

## How to apply

First, parse the RECAP-based SMILES fragment trees of the candidate molecules produced by the TeFT Transformer decoding stage, respecting the paper's constraints: molecular weight below 500 Da, only the ten elements C, H, O, N, P, S, Cl, Br, I, F permitted, and a limit of 1500 nodes for the total count of SMILES tree nodes. Second, build the molecular fragment tree from the experimental low-resolution MSn peak list (top 5-6 fragmentation peaks with their five nearby points, capped at 100 peaks by ion intensity) using the fragmentation rules in Supplementary Table S1, with error correction applied via multiple-measurement averaging and deconvolution integration on the spectrogram data. Third, match the two fragment trees using the paper's scoring rule table: +6/+3/+0 for matched/partially matched/unmatched parent atoms, +5/+2/+0 for fragment atoms, and +4/+1.5/+0 for neutral losses, assigning each matched pair to the appropriate tier. Fourth, normalize each raw score by the perfect-match score obtained when both trees are identical. Finally, rank all candidate molecules by the normalized SMILES fragment tree similarity; the highest-scoring SMILES tree is taken as the tested substance and its tree annotates the fragmentation peaks.

## Related tools

- **TeFT** (Overall pipeline that combines the Transformer candidate generation with fragmentation tree similarity scoring for de novo structure prediction from low-resolution MSn spectra.) — https://github.com/thumingo/TeFT
- **Transformer** (Generates candidate complete SMILES structures from MS2 spectra whose fragment trees are then scored.)

## Examples

```
python rank_candidates.py --candidates smiles_trees.csv --spectrum peaks.mzML --rules supplementary_table_s1.json --output similarity_scores.csv
```

## Evaluation signals

- The output similarity_scores.csv matches the reported ranking behavior on the 16 flavonoid alcohols, where the correct candidate ranks highest (top-1 accuracy) for 8 of 16 substances.
- Each candidate's raw tree score, when divided by the perfect-match score (identical trees), yields a normalized similarity between 0 and 1.
- Scores assigned via the scoring table use only the three tiers: +6/+3/+0 for parent atoms, +5/+2/+0 for fragment atoms, and +4/+1.5/+0 for neutral losses.
- The molecular fragment tree built from the experimental spectrum respects the constraints: top 5-6 peaks with five nearby points each, and a peak list capped at 100 by intensity.
- For the miniaturized ion trap test of 23 substances, the worst-predicted result per class has similarity 0.98 (flavanols), 1 (flavones), and 0.93 (stilbenes), consistent with Table 2.

## Limitations

- TeFT was only trained on spectra recorded in positive ion mode (H+), restricting its applicability to negative ion mode data.
- The model 'may not exhibit exceptional performance advantages in predicting high-quality resolution spectra compared to other deep models' such as SIRIUS4, MSNovelist, or MassGenie.
- Constraints include molecular weight below 500 Da, only the ten elements C, H, O, N, P, S, Cl, Br, I, F permitted, and a limit of 1500 SMILES tree nodes.
- A full list of dissociation and rearrangement rules is in Supplementary Table S1; no versioning or public availability of the rule library is specified.
- No changelog or formal code availability statement beyond the GitHub repository reference was found.

## Evidence

- [discussion] simulating molecular fragmentation based on deep learning predictions and scoring the similarity with fragmentation trees: "simulating molecular fragmentation based on deep learning predictions and scoring the similarity with fragmentation trees"
- [discussion] By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked"
- [discussion] the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [discussion] a peak list, limited to a maximum of 100 peaks selected based on their ion intensities: "a peak list, limited to a maximum of 100 peaks selected based on their ion intensities"
- [discussion] A full list of dissociation and rearrangement rules adopted in our experiments can be found in Supplementary Table S1: "A full list of dissociation and rearrangement rules adopted in our experiments can be found in Supplementary Table S1"
- [discussion] constraints such as limiting the molecular weight to less than 500 Da: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability: "we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability"
