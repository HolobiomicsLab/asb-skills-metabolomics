---
name: low-resolution-peak-list-handling
description: Use when the input is a low-resolution tandem MSn spectrum from a miniaturized
  mass spectrometer (peak list by ion intensity) and you need clean, size-limited
  peak lists for de novo structure prediction with TeFT.
when_to_use_negative:
- The input is high-resolution, high-accuracy mass spectrometry data — the article
  notes other models (SIRIUS4/CSI:FingerID, MSNovelist, MassGenie) suit those data
  and TeFT's advantage lies in low-resolution spectra.
- 'The spectra were recorded in negative ion mode: TeFT was only trained on positive
  ion mode (H+) spectra, so its applicability is restricted.'
- The goal is identifying known compounds by library search rather than de novo annotation
  of unknown natural products or drug metabolites from low-resolution on-site data.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: TeFT
  role: the de novo structure-generation pipeline (Transformer decoding plus fragment-tree
    similarity scoring) that consumes the cleaned low-resolution peak lists
- name: miniaturized mass spectrometer
  role: the custom small linear ion trap instrument producing the low-resolution MSn
    spectra that this preprocessing handles
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:6c2271c3f195503f9774ffaa7dd4ec46a52e1e76afbf1c4225c2ec60afbaec9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TeFT
  - miniaturized mass spectrometer
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# low-resolution-peak-list-handling

## Summary

Preprocess low-resolution MSn peak lists from miniaturized mass spectrometers (average replicates, deconvolute, cap at 100 peaks by intensity, keep top 5–6 fragmentation peaks with five nearby points) so they are suitable for TeFT fragment-tree construction and SMILES similarity scoring.

## When to use

Use when the input is a low-resolution tandem MSn spectrum from a miniaturized mass spectrometer (peak list by ion intensity) and you need clean, size-limited peak lists for de novo structure prediction with TeFT.

## When NOT to use

- The input is high-resolution, high-accuracy mass spectrometry data — the article notes other models (SIRIUS4/CSI:FingerID, MSNovelist, MassGenie) suit those data and TeFT's advantage lies in low-resolution spectra.
- The spectra were recorded in negative ion mode: TeFT was only trained on positive ion mode (H+) spectra, so its applicability is restricted.
- The goal is identifying known compounds by library search rather than de novo annotation of unknown natural products or drug metabolites from low-resolution on-site data.

## Inputs

- low-resolution tandem MSn spectra (peak lists with m/z and ion intensity) from a custom-made small linear ion trap mass spectrometer with a continuous atmospheric pressure interface
- multiple replicate measurements per substance (for averaging)
- candidate SMILES fragment trees from the TeFT Transformer (≤1500 nodes, MW <500 Da, elements C,H,O,N,P,S,Cl,Br,I,F)

## Outputs

- deconvoluted, replicate-averaged peak list capped at 100 peaks by ion intensity
- experimental fragmentation-peak subset: top 5–6 peaks with their five nearby points, used to build the molecular fragment tree
- similarity_scores.csv (ranked candidate molecules by normalized SMILES fragment tree similarity)

## How to apply

For each measured substance, average multiple measurement replicates to reduce random noise, then apply deconvolution integration to correct errors on the spectrogram data. From the corrected spectrum, build a peak list limited to a maximum of 100 peaks selected by their ion intensities. For the experimental fragment tree, select the top five to six fragmentation peaks and, for each, retain the five nearby peak points as the experimental data fed into the fragmentation rules (Supplementary Table S1). Because these spectra come from a small linear ion trap mass spectrometer with low mass resolution, TeFT was specifically designed for such spectra; high-resolution inputs are better served by tools like SIRIUS4, MassGenie, or MSNovelist. Write the cleaned peak list out as the input for the candidate-similarity ranking stage (similarity_scores.csv).

## Related tools

- **TeFT** (the de novo structure-generation pipeline (Transformer decoding plus fragment-tree similarity scoring) that consumes the cleaned low-resolution peak lists)
- **miniaturized mass spectrometer** (the custom small linear ion trap instrument producing the low-resolution MSn spectra that this preprocessing handles)

## Examples

```
python teft_pipeline.py --peaks spectra.txt --replicates 5 --max-peaks 100 --top-frag 6 --nearby 5 --mode positive --output similarity_scores.csv
```

## Evaluation signals

- The processed peak list contains at most 100 peaks, ordered by ion intensity.
- The experimental fragment tree is built from exactly the top 5–6 fragmentation peaks, each with its five nearby peak points retained.
- After replicate averaging and deconvolution, replicate-to-replicate peak m/z/intensity discrepancies are removed (deconvolution integration applied across multiple measurements).
- Downstream validation: on the 16 flavonoid alcohols, complete structure prediction is achieved for 8 substances — the correct candidate for each spectrum should rank highest in similarity_scores.csv (top-1 accuracy).
- All peaks derive from spectra recorded in positive ion mode (H+); any negative-mode peaks indicate a misuse of the pipeline.

## Limitations

- TeFT was only trained on spectra recorded in positive ion mode (H+), restricting its applicability to other ionization modes.
- The fragmentation rule library (Supplementary Table S1) is only described, not versioned or publicly distributed in the README.
- The article notes TeFT 'may not exhibit exceptional performance advantages in predicting high-quality resolution spectra compared to other deep models', so this skill should not be expected to outperform high-resolution specialists on such data.

## Evidence

- [discussion] we implement error correction by employing multiple measurement averaging and deconvolution integration techniques on the spectrogram data: "we implement error correction by employing multiple measurement averaging and deconvolution integration techniques on the spectrogram data"
- [discussion] a peak list, limited to a maximum of 100 peaks selected based on their ion intensities: "a peak list, limited to a maximum of 100 peaks selected based on their ion intensities"
- [discussion] the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [abstract] the TeFT has been validated on a miniaturized mass spectrometer with low-resolution spectra for 16 flavonoid alcohols, achieving complete structure prediction for 8 substances: "the TeFT has been validated on a miniaturized mass spectrometer with low-resolution spectra for 16 flavonoid alcohols, achieving complete structure prediction for 8 substances"
- [discussion] we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability: "we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability"
