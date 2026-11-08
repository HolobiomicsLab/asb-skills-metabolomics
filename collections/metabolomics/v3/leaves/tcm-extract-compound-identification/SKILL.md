---
name: tcm-extract-compound-identification
description: Use when a low-resolution MS2 spectrum of a marker peak from a TCM/herbal
  extract was acquired on a miniaturized mass spectrometer and the goal is de novo
  structural annotation of the contained compound without a library match.
when_to_use_negative:
- Input spectra are high-resolution/high-accuracy MS2 — models like MassGenie, SIRIUS4,
  or MSNovelist are geared toward that regime and TeFT may not show performance advantages
  there.
- Spectra were recorded in negative ion mode — TeFT was only trained on positive ion
  mode (H+) spectra.
- The compound's fragmentation behavior lacks clear fragmentation rules, or prior
  mass spectrometry library search already yields a confident identification.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0154
tools:
- name: TeFT
  role: 'De novo structure generation from low-resolution MSn spectra: simulates the
    fragmentation tree, generates candidate molecules, and ranks them by SMILES fragment-tree
    similarity'
  repo: thumingo/TeFT
- name: miniaturized mass spectrometer
  role: On-site acquisition of the low-resolution MS2 spectrum of the TCM extract
    marker peak that is fed to TeFT
- name: ultrasound-assisted extraction (UAE)
  role: Prepares the TCM (e.g., Anweiyang capsule) extract containing the flavonoid
    marker ion prior to MS2 acquisition
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:f168e7bdb9ab376835e92f93653dbcb84f703e39c06a3a688b1f25983f91e63d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - TeFT
  - miniaturized mass spectrometer
  - ultrasound-assisted extraction (UAE)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# tcm-extract-compound-identification

## Summary

Identifies the unknown compound in a traditional Chinese medicine extract (e.g., the m/z 269 marker peak of Anweiyang capsules) by running TeFT fragmentation-tree simulation on low-resolution MS2 spectra from a miniaturized mass spectrometer and ranking candidates by SMILES fragment-tree similarity.

## When to use

Use when a low-resolution MS2 spectrum of a marker peak from a TCM/herbal extract was acquired on a miniaturized mass spectrometer and the goal is de novo structural annotation of the contained compound without a library match.

## When NOT to use

- Input spectra are high-resolution/high-accuracy MS2 — models like MassGenie, SIRIUS4, or MSNovelist are geared toward that regime and TeFT may not show performance advantages there.
- Spectra were recorded in negative ion mode — TeFT was only trained on positive ion mode (H+) spectra.
- The compound's fragmentation behavior lacks clear fragmentation rules, or prior mass spectrometry library search already yields a confident identification.

## Inputs

- MS2 spectrum of a selected marker peak (e.g., m/z 269) from a TCM extract, acquired on a miniaturized (low-resolution) mass spectrometer
- Peak list capped at 100 peaks by ion intensity (top 5-6 fragmentation peaks plus five nearby peak points each)
- SMILES structure database or candidate generation space (elements C, H, O, N, P, S, Cl, Br, I, F; MW <500 Da)

## Outputs

- Ranked candidate list of molecular structures scored by SMILES fragment-tree similarity
- Predicted structure (SMILES) of the top-ranked candidate, e.g., formononetin
- Fragmentation peak annotations mapping the experimental MS2 peaks to the top candidate's fragment tree

## How to apply

Prepare the marker-peak MS2 spectrum (e.g., m/z 269 from Anweiyang capsule extract obtained via ultrasound-assisted extraction at 180 W for 45 min) and select the top 5-6 fragmentation peaks with their five nearby peak points, capped at 100 peaks by ion intensity. Run TeFT's fragmentation tree simulation (thumingo/TeFT, combining fragmentation tree models with a Transformer module) to generate candidate structures, constraining molecular weight to <500 Da and permitting only the ten elements C, H, O, N, P, S, Cl, Br, I, F. Rank candidates by comparing SMILES fragment-tree similarity between each predicted molecule and the experimental fragment tree, limiting SMILES trees to 1500 nodes. Validate the top-ranked candidate by confirming it fully annotates the observed fragmentation peaks; for the Anweiyang capsule the top hit was formononetin, which annotated all three fragmentation peaks.

## Related tools

- **TeFT** (De novo structure generation from low-resolution MSn spectra: simulates the fragmentation tree, generates candidate molecules, and ranks them by SMILES fragment-tree similarity) — thumingo/TeFT
- **miniaturized mass spectrometer** (On-site acquisition of the low-resolution MS2 spectrum of the TCM extract marker peak that is fed to TeFT)
- **ultrasound-assisted extraction (UAE)** (Prepares the TCM (e.g., Anweiyang capsule) extract containing the flavonoid marker ion prior to MS2 acquisition)

## Evaluation signals

- The top-ranked candidate matches the expected compound (e.g., formononetin for the Anweiyang m/z 269 spectrum) and has the highest fragment-tree similarity score.
- All observed fragmentation peaks (three for the Anweiyang case) are fully annotated by the top candidate's fragment tree.
- All returned candidates satisfy the constraints: molecular weight <500 Da and composition limited to C, H, O, N, P, S, Cl, Br, I, F.
- The experimental peak list respects the curation rules: top 5-6 fragmentation peaks plus nearby points, maximum 100 peaks by intensity.
- SMILES trees of candidates stay within the 1500-node limit during similarity ranking.

## Limitations

- TeFT was trained only on spectra recorded in positive ion mode (H+), restricting applicability to negative ion mode data.
- TeFT is best suited to annotating fragmentation peaks with clear fragmentation rules; the article does not claim performance advantages over SIRIUS4, MSNovelist, or MassGenie on high-quality-resolution spectra.
- Training relied on compiled open-source databases (GNPS, HMDB 5.0, MoNA), so candidates outside that chemical space may be missed.
- The article does not provide a quantitative head-to-head comparison of TeFT with SIRIUS4/MSNovelist/MassGenie.

## Evidence

- [abstract] Finally, TeFT confirmed the structure of the compound contained in a Chinese medicine substance called the Anweiyang capsule: "Finally, TeFT confirmed the structure of the compound contained in a Chinese medicine substance called the Anweiyang capsule"
- [abstract] the TeFT has been validated on a miniaturized mass spectrometer with low-resolution spectra for 16 flavonoid alcohols: "the TeFT has been validated on a miniaturized mass spectrometer with low-resolution spectra for 16 flavonoid alcohols, achieving complete structure prediction for 8 substances"
- [discussion] By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry"
- [readme] TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module.: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module."
- [discussion] we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability: "we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability"
