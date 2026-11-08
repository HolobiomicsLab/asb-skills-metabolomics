---
name: hresims-mass-error-validation
description: Use when structural annotations retrieved from the ENPKG knowledge graph
  (e.g., Sachem Tanimoto ≥ 0.8 similarity hits among confident annotations) must be
  confirmed against the accurate-mass HRESIMS data of an isolated compound before
  targeted isolation.
when_to_use_negative:
- The candidate annotation has no assigned molecular formula or only low-resolution
  MS data, so no meaningful ppm error can be computed.
- The goal is de novo structure elucidation rather than confirming an already proposed
  analog among KG annotations.
- Input is a raw feature table (e.g., MZmine feature list) without structural annotation
  — annotation confidence must be established first (e.g., ISDB + SIRIUS consensus).
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Sachem
  role: Structural-similarity/substructure queries over the KG's 2D structures that
    surface the candidate analogs subsequently mass-validated
  repo: https://enpkg.commons-lab.org/graphdb/
- name: RDKit
  role: Computation of theoretical exact monoisotopic mass from the candidate structure/formula
    for ppm-error calculation
- name: ChEMBL web resource client
  role: Provides the reported anti-L. donovani compounds (CHEMBL367) whose structures
    anchor the similarity query and activity metadata linked to validated hits
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acscentsci.3c00800
    title: A Sample-Centric and Knowledge-Driven Computational Framework for Natural
      Products Drug Discovery
  source_content_hash: sha256:9254a4dcfbfe403268b750f5c26f80d17f7fc17234828fa9bad854171f85df61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChEMBL web resource client
  - ENPKG
  - RDF
  - RDKit
  - Sachem
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# HRESIMS mass-error validation of candidate structural annotations

## Summary

Validates a KG-retrieved candidate structure (e.g., 11-β-hydroxypristimerin as an isoiguesterin analog) by comparing its theoretical monoisotopic mass against the isolated compound's HRESIMS accurate-mass measurement, accepting hits within a sub-ppm error tolerance (0.33 ppm in the reported case).

## When to use

Use when structural annotations retrieved from the ENPKG knowledge graph (e.g., Sachem Tanimoto ≥ 0.8 similarity hits among confident annotations) must be confirmed against the accurate-mass HRESIMS data of an isolated compound before targeted isolation.

## When NOT to use

- The candidate annotation has no assigned molecular formula or only low-resolution MS data, so no meaningful ppm error can be computed.
- The goal is de novo structure elucidation rather than confirming an already proposed analog among KG annotations.
- Input is a raw feature table (e.g., MZmine feature list) without structural annotation — annotation confidence must be established first (e.g., ISDB + SIRIUS consensus).

## Inputs

- Candidate structural annotations from the ENPKG KG (molecular formula, InChI/SMILES, e.g., Sachem similarity hits against CHEMBL367 anti-L. donovani compounds)
- HRESIMS accurate-mass measurement of the isolated compound (observed m/z, e.g., 481.2950)

## Outputs

- Per-candidate ppm mass error (observed vs. calculated monoisotopic mass)
- Validated analog list with structures, similarity relationships, activity metadata, and sample provenance (e.g., sachem_analog_query_results.csv)

## How to apply

Take the candidate formula returned by the knowledge-graph query (e.g., C30H41O5 for the pristimerin-type triterpene analog) and compute its theoretical monoisotopic mass, e.g., with RDKit's exact-mass calculation. Compare it against the observed HRESIMS peak of the isolated compound (here m/z 481.2950) and convert the difference to a ppm error: (observed − calculated)/calculated × 1e6. Accept the candidate only if the error falls within the reported sub-ppm tolerance (0.33 ppm for the validated analog), accounting for the ion species/adduct. In the ENPKG workflow this validation is applied to the top-ranked Sachem structural-similarity hit (Tanimoto ≥ 0.8 against isoiguesterin among 99 confident annotations) and combined with expert review, since mass accuracy alone does not confirm activity or identity.

## Related tools

- **Sachem** (Structural-similarity/substructure queries over the KG's 2D structures that surface the candidate analogs subsequently mass-validated) — https://enpkg.commons-lab.org/graphdb/
- **RDKit** (Computation of theoretical exact monoisotopic mass from the candidate structure/formula for ppm-error calculation)
- **ChEMBL web resource client** (Provides the reported anti-L. donovani compounds (CHEMBL367) whose structures anchor the similarity query and activity metadata linked to validated hits)

## Examples

```
from rdkit import Chem; from rdkit.Chem import Descriptors; calc = Descriptors.ExactMolWt(Chem.MolFromSmiles('C30H41O5')); ppm = abs(481.2950 - calc) / calc * 1e6
```

## Evaluation signals

- Computed ppm error for the top analog is within the reported tolerance (0.33 ppm for m/z 481.2950, C30H41O5).
- The validated candidate is among the 10 structures with Tanimoto similarity ≥ 0.8 against isoiguesterin (out of 99 confident annotations).
- Ion species/adduct is consistent between the observed HRESIMS peak and the calculated formula mass.
- Validation output agrees with expert review identifying the compound as an active analog of reported anti-L. donovani compounds.

## Limitations

- Mass accuracy alone does not confirm biological activity or unique identity — the paper notes no validation metric or ground-truth comparison was given for the 'active analog of reported anti-L. donovani compounds' identification, and final acceptance relied on expert review.
- Merging experimental data with public KG resources can introduce errors; no false-positive/false-negative rate for the semantic merging was reported.
- PPM tolerance assumes correct assignment of ion species; isotope-grouping and m/z tolerances elsewhere in the pipeline (e.g., 12 ppm isotope detection in MZmine) are far looser and must not be confused with the sub-ppm HRESIMS validation threshold.

## Evidence

- [other] Verify the top candidate analog against the isolated compound's accurate-mass data (HRESIMS m/z 481.2950, calculated for C30H41O5, error 0.33 ppm).: [evidence span withheld — anchor status: not_found]
- [results] Tanimoto similarity threshold for analogs: "Tanimoto similarity superior or equal to 0.8 against isoiguesterin"
- [results] Combining annotation approaches strengthens confidence: [evidence span withheld — anchor status: not_found]
- [readme] SPARQL research interface for the ENPKG graph: "The SPARQL research interface can be reached at https://enpkg.commons-lab.org/graphdb/sparql."
