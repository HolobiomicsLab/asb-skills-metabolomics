---
name: dataset-count-verification
description: Use when reproducing or auditing a claim that depends on exact dataset
  frequencies, such as the CyProduct paper's finding that η+η-forming reactions occur
  in only 4 of 828 η-η BoM records and all 4 are Cyclization.
when_to_use_negative:
- When the goal is to train or score a CypBoM classifier on the BoM records — use
  the model-training skill instead; counting is only a consistency check.
- When the input SDF lacks annotated BoM fields (e.g. a raw Zaretzki SoM dataset without
  the literature curation that produced BoMs) — the tally cannot be computed from
  unconverted SoM data.
- When auditing counts of a different dataset (e.g. EBoMD2 hold-out, 60Reactants.sdf,
  30NonReactants.sdf) whose published totals are not the 828/1348/86/·+η·4 figures.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: EBoMD
  role: Source dataset whose annotated BoM records are counted to verify the paper's
    totals
  repo: https://bitbucket.org/wishartlab/cyproduct/src/master/
- name: EBoMD.sdf
  role: Concrete file containing the 679 Zaretzki-derived compounds with BoM annotations
    to parse and tally
- name: Zaretzki dataset
  role: Upstream source of the 679 substrate compounds from which the BoM annotations
    were derived
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jcim.1c00144
    title: 'CyProduct: A Software Tool for Accurately Predicting the Byproducts of
      Human Cytochrome P450 Metabolism'
  source_content_hash: sha256:fad4f0022f1f9d45aa7c051a69167f3154cbd6236b17309a046fe2d470455e2c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EBoMD
  - EBoMD.sdf
  - Zaretzki dataset
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# dataset-count-verification

## Summary

Tallies annotated bond-of-metabolism (BoM) records in the EBoMD/EBoMD2 SDF datasets and cross-checks the counts (e.g. 828 η-η BoMs, 1348 η-H, 86 η-SPN, 4 η+η Cyclization cases) against values reported in the CyProduct paper, flagging any mismatch as a reproduction failure.

## When to use

Use when reproducing or auditing a claim that depends on exact dataset frequencies, such as the CyProduct paper's finding that η+η-forming reactions occur in only 4 of 828 η-η BoM records and all 4 are Cyclization.

## When NOT to use

- When the goal is to train or score a CypBoM classifier on the BoM records — use the model-training skill instead; counting is only a consistency check.
- When the input SDF lacks annotated BoM fields (e.g. a raw Zaretzki SoM dataset without the literature curation that produced BoMs) — the tally cannot be computed from unconverted SoM data.
- When auditing counts of a different dataset (e.g. EBoMD2 hold-out, 60Reactants.sdf, 30NonReactants.sdf) whose published totals are not the 828/1348/86/·+η·4 figures.

## Inputs

- EBoMD.sdf — SDF file with annotated BoM records for the 679 Zaretzki CYP450 substrate compounds (in the repository's Datasets folder)
- Expected counts from the paper: 828 η-η, 1348 η-H, 86 η-SPN BoM records; 4 η+η-forming Cyclization records

## Outputs

- CSV table (e.g. ebomd_eta_eta_cyclization_counts.csv) with per-class BoM tallies, the η+η Cyclization count, and a boolean verification result for the '4 of 828, all Cyclization' claim
- Log of any count mismatches or unexpected ReactionType labels

## How to apply

Parse the annotated SDF file (EBoMD.sdf) to extract every BoM record for the 679 Zaretzki-derived compounds, recording the bond class (η-η, η-H, η-SPN) and whether the bond is modified or formed. Tally records per class to confirm the paper's totals: 828 η-η, 1348 η-H, and 86 η-SPN BoMs. Within the η-η subset, separate the formed bonds (η+η) from modified ones, and verify that exactly 4 η+η records exist and that all 4 carry the ReactionType 'Cyclization' — the paper states that η+η-forming reactions are very rare (only 4 of 828). Finally, write the tallies and a boolean verification flag to a structured output (e.g. ebomd_eta_eta_cyclization_counts.csv) so the reproduction can be diffed against the published numbers; any discrepancy indicates a parsing or labeling error rather than a genuine dataset property.

## Related tools

- **EBoMD** (Source dataset whose annotated BoM records are counted to verify the paper's totals) — https://bitbucket.org/wishartlab/cyproduct/src/master/
- **EBoMD.sdf** (Concrete file containing the 679 Zaretzki-derived compounds with BoM annotations to parse and tally)
- **Zaretzki dataset** (Upstream source of the 679 substrate compounds from which the BoM annotations were derived)

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('ebomd_eta_eta_cyclization_counts.csv'); assert (df.eta_eta_total==828).all() and (df.eta_plus_eta_cyclization==4).all()"
```

## Evaluation signals

- The counted η-η BoM total must equal 828.
- The counted η-H BoM total must equal 1348 and the η-SPN total must equal 86.
- The η+η-forming subset must contain exactly 4 records, each labeled ReactionType = Cyclization.
- A boolean 'verified' flag in the output CSV must be True; any False indicates a parsing or annotation-classification error.
- Class separation must correctly distinguish formed bonds (η+η) from modified bonds within the η-η group — misclassification here would inflate the η+η count.

## Limitations

- Counts depend on the annotation conventions used in the SDF; the paper notes that the BoM dataset was produced by converting the Zaretzki SoM dataset via literature curation, so any deviation in that curation will alter the tallies.
- The paper does not state software versions or provide an explicit repository URL in the text, so tooling for the parse may need to be inferred from the README's file descriptions.
- The claim '4 of 828, all Cyclization' is specific to EBoMD; it must not be extrapolated to EBoMD2 or the BioTransformer hold-out dataset.

## Evidence

- [methods] Total counts of η-η, η-H and η-SPN BoMs in EBoMD are 828, 1348 and 86: [evidence span withheld — anchor status: not_found]
- [methods] η+η-forming reactions are very rare -- occurring in only 4 of the 828 η-η reactions: [evidence span withheld — anchor status: not_found]
