---
name: tanimoto-similarity-calculation
description: Use when the same MS/MS feature has structural annotations from multiple
  tools (GNPS, Sirius, ISDB-LOTUS) and you need pairwise chemical-similarity concordance
  indicators to feed MS2DECIDE aggregation.
when_to_use_negative:
- Only one tool produced a structural annotation for a feature — no pairwise Tanimoto
  can be computed (defaults, not real similarities, would be reported)
- You need spectral similarity (cosine/greedy scores) rather than structural similarity
  between candidate molecules
- Annotations lack SMILES/InChI structures (e.g., empty GNPS entries), so Morgan fingerprints
  cannot be generated
tools:
- name: MS2DECIDE
  role: Python library whose K_estimation workflow compiles the three tool annotations
    and consumes the Tanimoto indicators for decision-theory aggregation
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Source of one of the three structural annotations (top FBMN library-match
    molecule) used in the pairwise Tanimoto computations
- name: Sirius
  role: Source of one of the three structural annotations (structure_identifications.tsv
    from Sirius 6 batch mode)
- name: ISDB-LOTUS
  role: Source of one of the three structural annotations, produced via get_cfm_annotation
    matching against ISDB-LOTUS spectral data
- name: matchms
  role: Spectral matching backend (spectral_lib_matcher) used to obtain the ISDB-LOTUS
    annotation whose structure enters the Tanimoto computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:8d697aa9dba4cd60b4c3893ea87b764d6ed2cb932d92eabd5ebd0f71c3025625
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MejriY/MS2DECIDE
  tools:
  - GNPS
  - ISDB-LOTUS
  - MS2DECIDE
  - Sirius
  - matchms
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# tanimoto-similarity-calculation

## Summary

Computes the three pairwise Tanimoto similarity indicators (t_g,s, t_g,i, t_s,i) between the top structural annotations returned by GNPS, Sirius, and ISDB-LOTUS for each MS/MS feature, using Morgan fingerprints. These structural-concordance indicators are core inputs to the MS2DECIDE decision-theory aggregation.

## When to use

Use when the same MS/MS feature has structural annotations from multiple tools (GNPS, Sirius, ISDB-LOTUS) and you need pairwise chemical-similarity concordance indicators to feed MS2DECIDE aggregation.

## When NOT to use

- Only one tool produced a structural annotation for a feature — no pairwise Tanimoto can be computed (defaults, not real similarities, would be reported)
- You need spectral similarity (cosine/greedy scores) rather than structural similarity between candidate molecules
- Annotations lack SMILES/InChI structures (e.g., empty GNPS entries), so Morgan fingerprints cannot be generated

## Inputs

- GNPS FBMN annotation results (top library hit per feature, from spectral matching or iterative weighted analog search)
- Sirius 6 annotation file (structure_identifications.tsv, Confidence Approximate score)
- ISDB-LOTUS annotations obtained via get_cfm_annotation(mgf, ISDBtol) against ISDB-LOTUS spectral data (Zenodo record 8287341) with spectral_lib_matcher/MatchMS
- Quantitative feature table (.csv from MZmine >2.53 FBMN export) and MGF file with tandem mass spectra

## Outputs

- indicators_table.csv with one row per MS/MS feature and six indicator columns: c_g, c_s, c_i, t_g,s, t_g,i, t_s,i
- Tanimoto similarity values in [0, 1] for each of the three tool pairs per feature

## How to apply

For each MS/MS feature, extract the top candidate molecule (SMILES/InChI) from each tool's annotation: GNPS from the FBMN spectral-matching result, Sirius from structure_identifications.tsv, and ISDB-LOTUS via get_cfm_annotation(mgf, ISDBtol) using spectral_lib_matcher (strict 0.02 Da tolerance by default). Compute a Morgan fingerprint for each of the three candidates and derive the Tanimoto coefficient (Jaccard form) for each tool pair: t_g,s (GNPS-Sirius), t_g,i (GNPS-ISDB-LOTUS), t_s,i (Sirius-ISDB-LOTUS). When one tool returns no match, the pairwise Tanimoto cannot be computed, so a default value of 0.7 is assigned to T_gs and T_gi; when Sirius cannot propose an annotation, its matching score is set to 0.5. Assemble one row per feature with the six indicator columns c_g, c_s, c_i, t_g,s, t_g,i, t_s,i matching Table 3 of the article.

## Related tools

- **MS2DECIDE** (Python library whose K_estimation workflow compiles the three tool annotations and consumes the Tanimoto indicators for decision-theory aggregation) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Source of one of the three structural annotations (top FBMN library-match molecule) used in the pairwise Tanimoto computations)
- **Sirius** (Source of one of the three structural annotations (structure_identifications.tsv from Sirius 6 batch mode))
- **ISDB-LOTUS** (Source of one of the three structural annotations, produced via get_cfm_annotation matching against ISDB-LOTUS spectral data)
- **matchms** (Spectral matching backend (spectral_lib_matcher) used to obtain the ISDB-LOTUS annotation whose structure enters the Tanimoto computation)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- indicators_table.csv exists, has exactly one row per MS/MS feature, and contains exactly the six columns c_g, c_s, c_i, t_g,s, t_g,i, t_s,i
- All Tanimoto values fall within [0, 1]
- Rows where GNPS or Sirius returned no match show the documented default of 0.7 for T_gs/T_gi rather than a computed similarity
- Manually recompute the Tanimoto coefficient from RDKit Morgan fingerprints of the reported top-candidate SMILES for a sample of features and confirm it matches t_g,s/t_g,i/t_s,i
- The table merges cleanly with the MS2DECIDE K-value output (K.tsv) so each feature's ranks can be traced back to its six indicators

## Limitations

- When there is no GNPS or Sirius match, the pairwise Tanimoto cannot be calculated and a default value of 0.7 is assigned to T_gs and T_gi, so those entries are imputed rather than measured
- In some cases Sirius cannot propose an annotation; a value of 0.5 is associated with the Sirius matching score in those instances
- Sirius structure_identifications.tsv may contain redundant features in the mappingFeatureId column, raising 'ValueError: DataFrame index must be unique for orient=index' unless cleaned
- The iterative weighted GNPS analog search can leave empty annotations, which prevent fingerprint computation for the GNPS side of a pair

## Evidence

- [results] The pairwise Tanimoto similarities between the molecules returned by each pair of tools (GNPS, Sirius, ISDB-LOTUS) are computed using Morgan fingerprints: [evidence span withheld — anchor status: not_found]
- [readme] In scenarios where there is no match with GNPS or no match with Sirius, the tanimoto between GNPS and Sirius cannot be calculated. Hence, a default value of 0.7 was assigned to $T_{gs}$ and $T_{gi}$: [evidence span withheld — anchor status: external_doc]
- [readme] Unfortunately, in some cases, Sirius was not able to propose an annotation. To remedy, we associated a value of 0.5 to Sirius matching score.: [evidence span withheld — anchor status: external_doc]
- [readme] This function calculates annotations by matching mass spectrometry data against ISDB-LOTUS spectral data accessible: [evidence span withheld — anchor status: external_doc]
