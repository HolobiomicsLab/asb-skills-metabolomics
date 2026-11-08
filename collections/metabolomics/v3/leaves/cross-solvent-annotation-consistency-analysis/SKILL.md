---
name: cross-solvent-annotation-consistency-analysis
description: Use when evaluating DeepSAT annotation robustness on paired HSQC spectra
  of identical compounds recorded in different NMR solvents, or when choosing whether
  solvent changes threaten structure-identification reliability.
when_to_use_negative:
- When spectra are measured in a single solvent only — no cross-solvent retention
  can be computed and standard single-condition DeepSAT evaluation applies instead.
- When the goal is interpretability of which substructures drive predictions (occlusion
  sensitivity) rather than solvent-robustness quantification.
- When comparing against SMART 2.0 or other NMR search tools on general performance
  — this skill is specifically the solvent-change robustness evaluation.
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: DeepSAT
  role: CNN multi-task model that ingests 1H-13C HSQC spectra and predicts fingerprint
    bit strings, molecular weights, and structure classes used for cross-solvent annotation
    comparison
  repo: github.com/mwang87/DeepSAT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-023-00738-4
    title: 'DeepSAT: Learning Molecular Structures from Nuclear Magnetic Resonance
      Data'
  source_content_hash: sha256:9f6707c183187cb30eacc827c164128fe631c90d8560451dad6711228e793c85
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepSAT
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-solvent-annotation-consistency-analysis

## Summary

Assess how stable DeepSAT's 1H-13C HSQC structure identification is when the same natural products are re-measured in a different deuterated solvent (methanol-d4 vs chloroform-d), quantifying the fraction of correct annotations retained across solvent conditions.

## When to use

Use when evaluating DeepSAT annotation robustness on paired HSQC spectra of identical compounds recorded in different NMR solvents, or when choosing whether solvent changes threaten structure-identification reliability.

## When NOT to use

- When spectra are measured in a single solvent only — no cross-solvent retention can be computed and standard single-condition DeepSAT evaluation applies instead.
- When the goal is interpretability of which substructures drive predictions (occlusion sensitivity) rather than solvent-robustness quantification.
- When comparing against SMART 2.0 or other NMR search tools on general performance — this skill is specifically the solvent-change robustness evaluation.

## Inputs

- Paired 1H-13C HSQC spectra of the same natural products measured in two deuterated solvents (e.g. methanol-d4 and chloroform-d)
- Known molecular structures / reference small molecules for Morgan fingerprint generation
- Pretrained DeepSAT CNN multi-task model (normal or multiplicity-edited HSQC inputs supported)

## Outputs

- Per-compound, per-solvent identification/annotation results with cosine scores (e.g. solvent_comparison.csv)
- Solvent-wise top-1 identification rates (e.g. 74.4% chloroform-d, 66.7% methanol-d4)
- Cross-solvent retention fraction of correctly identified structures (must exceed 65%)

## How to apply

Assemble paired 1H-13C HSQC spectra (here 18 natural products in methanol-d4 and the same 18 in chloroform-d, 36 spectra total) and run each through the pretrained DeepSAT CNN multi-task model, which predicts fingerprint bit strings, molecular weights, and structure classes. For each prediction, generate Morgan fingerprints from known small molecules and compute cosine similarity, applying the cosine score threshold of 0.8 used for annotation rate measurement; compute the identification/annotation rate separately per solvent condition. Then match results across solvents for each natural product and calculate the fraction still correctly identified when the solvent changes — the article reports DeepSAT stays above 65% retention, with 74.4% top-1 correct in chloroform-d and 66.7% in methanol-d4. Write per-compound and per-solvent outcomes to a table (e.g. solvent_comparison.csv) so per-compound solvent sensitivity is auditable.

## Related tools

- **DeepSAT** (CNN multi-task model that ingests 1H-13C HSQC spectra and predicts fingerprint bit strings, molecular weights, and structure classes used for cross-solvent annotation comparison) — github.com/mwang87/DeepSAT

## Examples

```
curl -X POST http://localhost:8000/api/smart3/search -d '[{"1H": 3.2, "13C": 55.1}, {"1H": 6.9, "13C": 128.4}]'
```

## Evaluation signals

- Top-1 correct identification rate must remain above 65% in each solvent condition (74.4% chloroform-d and 66.7% methanol-d4 in the reference evaluation).
- Annotation rate must be measured with the same cosine score threshold of 0.8 applied to predicted vs known Morgan fingerprints.
- Per-compound results must be matched across solvents so the fraction still correctly identified under solvent change is directly computable.
- Average cosine score between predicted and ground-truth fingerprints should fall in the ~0.84–0.86 range reported for normal (0.8450) and multiplicity-edited (0.8574) HSQC models.
- Output must include per-compound and per-solvent columns (solvent_comparison.csv schema) rather than only aggregate rates.

## Limitations

- Not all results are correctly identified across solvents — over 65% but less than 100% of annotations transfer when the solvent changes.
- Only two solvents (methanol-d4 vs chloroform-d) are covered in the reference evaluation; extrapolation to other deuterated solvents is untested in the provided context.
- Cosine score threshold of 0.8 is a fixed decision point; annotations scoring below it are not counted, which can inflate apparent solvent sensitivity.

## Evidence

- [results] DeepSAT's top-1 correct identification rate was 74.4% for spectra measured in chloroform-d and 66.7% for methanol-d4, remaining above 65% under both solvent conditions.: [evidence span withheld — anchor status: not_found]
- [results] Threshold definition for annotation: "For the measurement of annotation rate, cosine score of 0.8 was set as the threshold"
- [intro] Model trained on paired-solvent HSQC inputs: "DeepSAT was trained using a convolutional neural network (CNN)-based multi-task supervised learning architecture with 143,467 1H-13C HSQC spectra"
- [discussion] Solvent-change robustness criterion: "over 65% of the results were still correctly identified even though the data were recorded in different solvents"
- [readme] Programmatic classification API for HSQC peaks: "You can put in your peaks as a json list of dicts, with 1H,13C as headers."
