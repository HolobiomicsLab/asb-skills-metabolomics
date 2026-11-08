---
name: tandem-ms-annotation-score-extraction
description: Use when a feature-level annotation table contains top candidate structures
  and scores from multiple MS/MS annotation tools (GNPS FBMN, Sirius 6, ISDB-LOTUS)
  and per-ion agreement/match indicators are needed for prioritization.
when_to_use_negative:
- When only one annotation tool is used — pairwise Tanimoto indicators between tool
  annotations are undefined without at least two structural candidates
- When the input is already the aggregated K-ranked output table rather than raw multi-tool
  annotation results
- When candidate molecules lack SMILES/InChI structures, since Morgan fingerprints
  and Tanimoto similarities cannot be computed
edam_operation: http://edamontology.org/operation_2945
tools:
- name: MS2DECIDE
  role: Python library whose K_estimation pipeline compiles GNPS, ISDB-LOTUS, and
    Sirius annotations into the unified filtered/ranked dataframe
  repo: https://github.com/MejriY/MS2DECIDE
- name: GNPS
  role: Source of FBMN library-search match scores (c_g) and one of the three structural
    annotations entering pairwise Tanimoto computation
- name: Sirius
  role: Source of Sirius 6 structure identifications (c_s) via structure_identifications.tsv;
    missing annotations defaulted to a 0.5 matching score
- name: ISDB-LOTUS
  role: Source of spectral-lib_matcher-based annotations (c_i) against ISDB-LOTUS
    spectra; zero answers treated as novelty information
- name: matchms
  role: MatchMS/spectral_lib_matcher scoring used to compute match scores between
    experimental spectra and ISDB-LOTUS library spectra
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cmtd.202400088
    title: 'MS2DECIDE: Aggregating Multiannotated Tandem Mass Spectrometry Data with
      Decision Theory Enhances Natural Products Prioritization'
  source_content_hash: sha256:810a1f2bd17890d97e859299d80b9480cdb298bf15e237bdd3439a1aa4fd379d
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tandem-ms-annotation-score-extraction

## Summary

Extracts the six MS2DECIDE indicators per MS/MS feature: the closest-molecule match scores from GNPS, Sirius, and ISDB-LOTUS (c_g, c_s, c_i) plus the three pairwise Morgan/Tanimoto similarities between the tools' structural annotations (t_gs, t_gi, t_si), which feed the decision-theory aggregation and K-value prioritization.

## When to use

Use when a feature-level annotation table contains top candidate structures and scores from multiple MS/MS annotation tools (GNPS FBMN, Sirius 6, ISDB-LOTUS) and per-ion agreement/match indicators are needed for prioritization.

## When NOT to use

- When only one annotation tool is used — pairwise Tanimoto indicators between tool annotations are undefined without at least two structural candidates
- When the input is already the aggregated K-ranked output table rather than raw multi-tool annotation results
- When candidate molecules lack SMILES/InChI structures, since Morgan fingerprints and Tanimoto similarities cannot be computed

## Inputs

- MS/MS feature/quantitative table (.csv, MZmine FBMN export format)
- MGF file of tandem mass spectra
- Sirius 6 annotation file (structure_identifications.tsv) with confidence score type exact or approximate
- GNPS FBMN library-search results (strict or iterative weighted analog search, cosine threshold e.g. 0.1)
- ISDB-LOTUS annotations from get_cfm_annotation (mass tolerance < 0.5, default 0.02 Da)

## Outputs

- indicators table (one row per MS/MS feature) with exactly the six columns c_g, c_s, c_i, t_gs, t_gi, t_si
- indicators_table.csv / unified .tsv of processed, K-ranked annotations

## How to apply

Load the MS/MS feature table with the top candidate annotations and scores from GNPS, Sirius, and ISDB-LOTUS, and extract the closest-molecule match score for each tool (c_g, c_s, c_i) using the matchms/spectral_lib_matcher scoring parameters. Compute a Morgan fingerprint for the top candidate molecule of each tool annotation and derive the three pairwise Tanimoto similarities (t_gs for GNPS-Sirius, t_gi for GNPS-ISDB-LOTUS, t_si for Sirius-ISDB-LOTUS) via the Tanimoto/Jaccard formula. Handle missing annotations explicitly: if GNPS or Sirius returns no match, assign the default value 0.7 to T_gs and T_gi since the Tanimoto cannot be computed; if Sirius fails to propose an annotation, assign 0.5 to the Sirius matching score; for ISDB-LOTUS (strict 0.02 Da library search via spectral_lib_matcher/MatchMS), a zero answer is retained as meaningful information for novelty. Assemble one row per MS/MS feature into an indicators table with exactly the six columns c_g, c_s, c_i, t_gs, t_gi, t_si for downstream aggregation.

## Related tools

- **MS2DECIDE** (Python library whose K_estimation pipeline compiles GNPS, ISDB-LOTUS, and Sirius annotations into the unified filtered/ranked dataframe) — https://github.com/MejriY/MS2DECIDE
- **GNPS** (Source of FBMN library-search match scores (c_g) and one of the three structural annotations entering pairwise Tanimoto computation)
- **Sirius** (Source of Sirius 6 structure identifications (c_s) via structure_identifications.tsv; missing annotations defaulted to a 0.5 matching score)
- **ISDB-LOTUS** (Source of spectral-lib_matcher-based annotations (c_i) against ISDB-LOTUS spectra; zero answers treated as novelty information)
- **matchms** (MatchMS/spectral_lib_matcher scoring used to compute match scores between experimental spectra and ISDB-LOTUS library spectra)

## Examples

```
from ms2decide.K_estimation import K_estimation; K_estimation()
```

## Evaluation signals

- indicators_table.csv exists and has exactly one row per MS/MS feature
- The table contains exactly the six indicator columns c_g, c_s, c_i, t_gs, t_gi, t_si
- All similarity values fall within [0, 1]
- Missing GNPS/Sirius matches carry the documented defaults (0.7 for T_gs and T_gi; 0.5 for the Sirius matching score)
- Sirius structure_identifications.tsv was checked for redundant features in the mappingFeatureId column to avoid the 'DataFrame index must be unique' error

## Limitations

- When there is no GNPS or no Sirius match, the Tanimoto between GNPS and Sirius cannot be calculated and a default of 0.7 is assigned to T_gs and T_gi
- Sirius sometimes fails to propose an annotation; a value of 0.5 is then associated with the Sirius matching score
- The GNPS workflows use a library score threshold of 0.001 which can cause FBMN failures; increasing the threshold (e.g., 0.1) is recommended
- The iterative weighted analog search can take up to three hours and launches 27 FBMN jobs
- Redundant features in Sirius structure_identifications.tsv cause a 'ValueError: DataFrame index must be unique for orient='index'' error

## Evidence

- [methods] The pairwise Tanimoto similarities between the molecules returned by each pair of tools (GNPS, Sirius, ISDB-LOTUS) are computed using Morgan fingerprints: [evidence span withheld — anchor status: not_found]
- [readme] In scenarios where there is no match with GNPS or no match with Sirius, the tanimoto between GNPS and Sirius cannot be calculated. Hence, a default value of 0.7 was assigned to $T_{gs}$ and $T_{gi}$ in these instances.: [evidence span withheld — anchor status: external_doc]
- [readme] in some cases, Sirius was not able to propose an annotation. To remedy, we associated a value of 0.5 to Sirius matching score.: [evidence span withheld — anchor status: external_doc]
- [readme] This function calculates annotations by matching mass spectrometry data against ISDB-LOTUS spectral data: [evidence span withheld — anchor status: external_doc]
- [abstract] aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS): "aggregate the outputs of three widely used annotation tools (GNPS, Sirius, and ISDB‐LOTUS)"
