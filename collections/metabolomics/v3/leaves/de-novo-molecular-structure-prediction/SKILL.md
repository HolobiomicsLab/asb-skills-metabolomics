---
name: de-novo-molecular-structure-prediction
description: Use when you have a low-resolution MS2 spectrum (e.g., from a miniaturized
  mass spectrometer) for an unknown small molecule and need de novo structure prediction
  without a database match.
when_to_use_negative:
- Spectra are high-resolution, high-accuracy MS/MS — the article notes models like
  SIRIUS4 (CSI:FingerID), MSNovelist, and MassGenie are geared toward high-quality
  resolution spectra and quantitative comparisons were not provided.
- Spectra are recorded in negative ion mode — TeFT was trained only on positive ion
  mode (H+) spectra, restricting applicability there.
- The compound's mass spectrum is already identified by library search and no de novo
  interpretation is needed.
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: TeFT
  role: 'Core model: combines fragmentation tree models with a Transformer module
    to generate de novo molecular structure (SMILES) from MSn spectra and rank candidates
    by SMILES fragment tree similarity'
  repo: https://github.com/thumingo/TeFT
- name: miniaturized mass spectrometer
  role: On-site instrument producing the low-resolution MS2 spectra (e.g., the m/z
    269 marker peak from Anweiyang capsule extract) that serve as the method's input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:a4ff6eec757efd662c8d2fa54b827e2e8aafd06c563ee33acbcc4f7a0e9cffc5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - TeFT
  - miniaturized mass spectrometer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# de-novo-molecular-structure-prediction

## Summary

De novo prediction of a molecule's complete structure from low-resolution tandem mass spectrometry (MSn) spectra using the Transformer enabled Fragment Tree (TeFT) method, designed for on-site miniaturized mass spectrometers where library search and high-resolution deep models fail.

## When to use

Use when you have a low-resolution MS2 spectrum (e.g., from a miniaturized mass spectrometer) for an unknown small molecule and need de novo structure prediction without a database match.

## When NOT to use

- Spectra are high-resolution, high-accuracy MS/MS — the article notes models like SIRIUS4 (CSI:FingerID), MSNovelist, and MassGenie are geared toward high-quality resolution spectra and quantitative comparisons were not provided.
- Spectra are recorded in negative ion mode — TeFT was trained only on positive ion mode (H+) spectra, restricting applicability there.
- The compound's mass spectrum is already identified by library search and no de novo interpretation is needed.

## Inputs

- Low-resolution MS2 (MSn) tandem mass spectrum acquired on a miniaturized mass spectrometer, recorded in positive ion mode (H+)
- Peak list of m/z and ion intensities (maximum 100 peaks by intensity; top 5-6 fragmentation peaks plus 5 nearby peak points used as experimental data)

## Outputs

- Ranked list of candidate molecular structures (SMILES) with SMILES fragment-tree similarity scores
- Predicted complete molecular structure for the top candidate
- Fragmentation tree annotations mapping predicted fragments to the experimental fragmentation peaks

## How to apply

From the MS2 spectrum, select the top five to six fragmentation peaks plus their five nearby peak points, with the peak list limited to a maximum of 100 peaks by ion intensity. Apply error correction via multiple measurement averaging and deconvolution integration on the spectrogram data, then run TeFT's fragmentation tree simulation, which combines fragmentation tree models with a deep learning Transformer module that generates the complete SMILES structure sequence. Constrain the search space by limiting molecular weight to less than 500 Da, permitting only the ten elements C, H, O, N, P, S, Cl, Br, I, and F, and imposing a limit of 1500 nodes for the total count of SMILES tree nodes. Rank candidate substances by comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from the tandem mass spectrum. Verify the top candidate by checking that its predicted fragment tree completely annotates the experimental fragmentation peaks (e.g., formononetin fully annotated three peaks of the Anweiyang capsule m/z 269 spectrum).

## Related tools

- **TeFT** (Core model: combines fragmentation tree models with a Transformer module to generate de novo molecular structure (SMILES) from MSn spectra and rank candidates by SMILES fragment tree similarity) — https://github.com/thumingo/TeFT
- **miniaturized mass spectrometer** (On-site instrument producing the low-resolution MS2 spectra (e.g., the m/z 269 marker peak from Anweiyang capsule extract) that serve as the method's input)

## Examples

```
git clone https://github.com/thumingo/TeFT
```

## Evaluation signals

- The highest-scoring candidate's predicted fragment tree provides complete annotation of the selected top 5-6 fragmentation peaks (validation case: formononetin fully annotated three peaks of the Anweiyang capsule m/z 269 spectrum).
- All candidate structures satisfy the constraints: molecular weight < 500 Da, elements limited to C, H, O, N, P, S, Cl, Br, I, F, and SMILES tree node count limited to 1500.
- The experimental peak list respects the selection rules: maximum 100 peaks by ion intensity, top 5-6 fragmentation peaks plus five nearby peak points.
- On the flavonoid benchmark (16 flavonoid alcohols on the miniaturized mass spectrometer), the pipeline reproduces complete structure prediction for 8 substances.
- Error-corrected spectra (multiple measurement averaging and deconvolution integration) show reduced spurious peaks compared to raw spectrogram data.

## Limitations

- TeFT was only trained on spectra recorded in positive ion mode (H+), restricting its applicability to negative ion mode data.
- The model may not exhibit exceptional performance advantages for high-quality/high-resolution spectra compared to other deep models (e.g., SIRIUS4 with CSI:FingerID, MSNovelist, MassGenie).
- It is best suited to annotating fragmentation peaks with clear fragmentation rules; compounds with unclear fragmentation behavior may not be fully annotated.
- Prior deep models like MassGenie require computing resources unavailable for on-site miniaturized mass spectrometry platforms — a constraint TeFT addresses but which limits retraining scale.

## Evidence

- [abstract] TeFT method description: "we propose the Transformer enabled Fragment Tree (TeFT) method, which combines various types of fragmentation tree models and a deep learning Transformer module"
- [discussion] Peak selection rules: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [discussion] Candidate constraints: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] Candidate ranking mechanism: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry"
- [readme] TeFT definition from repository: "TeFT, a predict model combines various types of fragmentation tree models and the deep learning Transformer module."
