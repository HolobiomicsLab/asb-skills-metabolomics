---
name: flavonoid-fragmentation-tree-similarity-scoring
description: Use when you have a low-resolution MS2 spectrum (e.g. from a miniaturized
  mass spectrometer) of an unknown flavonoid or natural product and must rank candidate
  structures by fragment tree similarity rather than library matching.
when_to_use_negative:
- High-resolution, high-accuracy spectra from large instruments — the article notes
  SIRIUS4/CSI:FingerID, MSNovelist, and MassGenie are geared to such data and may
  outperform TeFT there
- Spectra recorded in negative ion mode — TeFT was only trained on positive ion mode
  (H+) spectra
- Molecular formula identification for molecules >500 Da or containing elements outside
  C, H, O, N, P, S, Cl, Br, I, F
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: TeFT
  role: Runs the fragmentation tree simulation that generates candidate molecular
    structures and the SMILES fragment trees compared for similarity ranking
  repo: https://github.com/thumingo/TeFT
- name: miniaturized mass spectrometer
  role: Acquires the low-resolution MS2 spectrum of the marker peak (m/z 269) from
    capsule extract that serves as the experimental fragment tree source
- name: MetFrag
  role: Prior fragment-prediction approach combining database search with fragment
    prediction that TeFT's fragmentation tree similarity scoring extends to low-resolution
    spectra
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42004-024-01189-0
    title: Structural annotation of unknown molecules in a miniaturized mass spectrometer
      based on a transformer enabled fragment tree method
  source_content_hash: sha256:365424af9d05f30dd7fad137f08ce017758233b4f314fba40ac77cff736ca97a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/thumingo/TeFT
  tools:
  - MetFrag
  - TeFT
  - Transformer enabled Fragment Tree (TeFT)
  - miniaturized mass spectrometer
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# flavonoid-fragmentation-tree-similarity-scoring

## Summary

Rank unknown-molecule candidates by comparing SMILES fragment trees predicted by TeFT's fragmentation tree simulation against the fragment tree built from an experimental low-resolution MS2 spectrum, as applied to the m/z 269 marker peak from Anweiyang capsule extract where formononetin emerged as the top-scoring candidate.

## When to use

Use when you have a low-resolution MS2 spectrum (e.g. from a miniaturized mass spectrometer) of an unknown flavonoid or natural product and must rank candidate structures by fragment tree similarity rather than library matching.

## When NOT to use

- High-resolution, high-accuracy spectra from large instruments — the article notes SIRIUS4/CSI:FingerID, MSNovelist, and MassGenie are geared to such data and may outperform TeFT there
- Spectra recorded in negative ion mode — TeFT was only trained on positive ion mode (H+) spectra
- Molecular formula identification for molecules >500 Da or containing elements outside C, H, O, N, P, S, Cl, Br, I, F

## Inputs

- MS2 spectrum of a marker peak from an unknown flavonoid (e.g. m/z 269 from Anweiyang capsule extract, low-resolution miniaturized mass spectrometer data)
- Peak list limited to a maximum of 100 peaks selected by ion intensity
- Top 5-6 fragmentation peaks with their five nearby peak points as experimental fragment data

## Outputs

- Ranked list of candidate molecular structures by SMILES fragment tree similarity
- Fragment annotations mapping predicted fragments to the three experimental fragmentation peaks
- Identified compound structure (e.g. formononetin for the Anweiyang m/z 269 spectrum)

## How to apply

Extract the target MS2 spectrum (e.g. the m/z 269 marker peak from Anweiyang capsule extract acquired by ultrasound-assisted extraction at 180 W for 45 min), select the top 5-6 fragmentation peaks with their five nearby peak points, and cap the peak list at 100 peaks by ion intensity. Run TeFT's fragmentation tree simulation on the spectrum to generate candidate molecular structures, applying the filters: molecular weight <500 Da, only the ten elements C, H, O, N, P, S, Cl, Br, I, F permitted, and SMILES trees limited to 1500 nodes. For each candidate, build its SMILES fragment tree and compare similarity to the experimental molecular fragment tree generated from the tandem MS spectrum, then rank substances by this similarity score. Verify the top candidate (formononetin in the Anweiyang case) by confirming that its predicted fragments completely annotate the three experimental fragmentation peaks.

## Related tools

- **TeFT** (Runs the fragmentation tree simulation that generates candidate molecular structures and the SMILES fragment trees compared for similarity ranking) — https://github.com/thumingo/TeFT
- **miniaturized mass spectrometer** (Acquires the low-resolution MS2 spectrum of the marker peak (m/z 269) from capsule extract that serves as the experimental fragment tree source)
- **MetFrag** (Prior fragment-prediction approach combining database search with fragment prediction that TeFT's fragmentation tree similarity scoring extends to low-resolution spectra)

## Examples

```
git clone https://github.com/thumingo/TeFT && python run_teft.py --ms2 mz269_anweiyang.txt --max-peaks 100 --max-nodes 1500 --mw-cutoff 500
```

## Evaluation signals

- The top-scoring candidate matches the reported result: formononetin was highest-scoring on the Anweiyang m/z 269 spectrum
- Complete annotation: all three experimental fragmentation peaks of the top candidate are fully annotated
- Constraint compliance: all surviving candidates have molecular weight <500 Da, contain only the ten permitted elements, and have SMILES trees ≤1500 nodes
- Peak-list sanity: experimental input contains at most 100 peaks by intensity, with the top 5-6 fragmentation peaks plus five nearby peak points selected

## Limitations

- TeFT was trained only on positive ion mode (H+) spectra, restricting applicability to negative ion mode data
- The model may not outperform SIRIUS4, MSNovelist, or MassGenie on high-quality high-resolution spectra
- Requires fragmentation peaks with clear fragmentation rules; ultrasound-assisted extraction and peak selection from low-resolution spectra can omit informative peaks beyond the 100-peak intensity cap

## Evidence

- [discussion] Rank by similarity: "By comparing the similarity between the SMILES fragment trees of each predicted molecule and the molecular fragment trees generated from tandem mass spectrometry, substance similarity was ranked"
- [discussion] Peak selection filters: "the top five to six fragmentation peaks and their five nearby peak points were selected as experimental data"
- [discussion] Constraint filters: "constraints such as limiting the molecular weight to less than 500 Da"
- [discussion] UAE extraction: "We employ ultrasound-assisted extraction (UAE) as a crucial technique to extract flavonoids from the drugs"
- [abstract] Anweiyang finding: "TeFT confirmed the structure of the compound contained in a Chinese medicine substance called the Anweiyang capsule"
- [discussion] Positive-ion-only limitation: "we only trained TeFT on spectra recorded in positive ion mode (H+), restricting its applicability"
