---
name: in-silico-metabolite-annotation
description: Use when you have .mgf MS/MS spectra (e.g. [M+H]+ adducts, parent masses
  100–1500 Da, ≥6 fragments) and need candidate molecular formula/structure annotations
  for dereplication.
when_to_use_negative:
- All benchmarking compounds are present in the DNP, which by definition favors ISDB-DNP
  over Sirius/MS-Finder — do not use these absolute baseline numbers to claim tool
  superiority.
- The GNPS spectral libraries were part of Sirius's training set, so cross-tool comparison
  is confounded.
- 197 benchmarking entries have large parent-mass differences (>0.01 Da) between experimental
  and calculated exact mass — annotation tools cannot flag these erroneous entries.
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0605
tools:
- name: ISDB-DNP
  role: In-silico spectral library (backed by Dictionary of Natural Products v 27.1)
    queried with filtered .mgf spectra to produce structure candidates
  repo: http://oolonek.github.io/ISDB/
- name: Sirius
  role: Predicts fragmentation trees to generate molecular formula and structure candidate
    annotations for the same spectra
- name: MS-Finder
  role: Integrates spectral/structural parameters to produce formula and structure
    candidate annotations
- name: RDKit
  role: Converts library structures to InChIKeys used to build the in-silico database
    and deduplicate structures/stereoisomers
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:dbdf60948306a40e4b9679154e443c1269e20ae80d60adb7911ea70b1ea0843f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ISDB-DNP
  - MS-Finder
  - RDKit
  - Sirius
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# In-silico metabolite annotation

## Summary

Annotates MS/MS spectra of specialized metabolites by matching them against in-silico and experimental spectral libraries (ISDB-DNP) and combinatorial/ML annotation tools (Sirius, MS-Finder), then benchmarking rank-1 correctness on the 2,107-spectrum DNP benchmarking dataset.

## When to use

Use when you have .mgf MS/MS spectra (e.g. [M+H]+ adducts, parent masses 100–1500 Da, ≥6 fragments) and need candidate molecular formula/structure annotations for dereplication.

## When NOT to use

- All benchmarking compounds are present in the DNP, which by definition favors ISDB-DNP over Sirius/MS-Finder — do not use these absolute baseline numbers to claim tool superiority.
- The GNPS spectral libraries were part of Sirius's training set, so cross-tool comparison is confounded.
- 197 benchmarking entries have large parent-mass differences (>0.01 Da) between experimental and calculated exact mass — annotation tools cannot flag these erroneous entries.

## Inputs

- .mgf spectral files of MS/MS spectra
- Filtered MS/MS spectra ([M+H]+ adducts, parent masses 100–1,500 Da, 500 most intense peaks, ≥6 fragments)
- Reference library structures (e.g. Dictionary of Natural Products v 27.1) converted to InChIKeys
- Benchmarking reference identifications for the 2,107-spectrum dataset

## Outputs

- Per-tool annotation tables with candidate structures/formulas and scores
- Rank-1 annotation counts and rates per tool (ISDB-DNP, Sirius, MS-Finder)
- ISDB-DNP in-silico database backed by DNP v 27.1

## How to apply

Convert the structures of your reference library (e.g. Dictionary of Natural Products v 27.1) to InChIKeys with RDKit and build an ISDB in-silico database; query filtered .mgf spectra (parent mass 100–1500 Da, [M+H]+ adducts, 500 most intense peaks, ≥6 fragments) against it via spectral matching. In parallel, run Sirius and MS-Finder on the same spectra to obtain formula and structure candidates. Evaluate rank-1 correctness of each tool's top annotation against reference identifications; on the 2,107-spectrum benchmarking dataset expect ISDB-DNP 214 (10.2%), Sirius 975 (46.3%) and MS-Finder 180 (8.5%) correct rank-1 annotations before taxonomically informed re-ranking. Treat these baselines as a pre-condition for any downstream scoring improvement (e.g. F1 gains of 1.5–7× from taxonomic re-ranking).

## Related tools

- **ISDB-DNP** (In-silico spectral library (backed by Dictionary of Natural Products v 27.1) queried with filtered .mgf spectra to produce structure candidates) — http://oolonek.github.io/ISDB/
- **Sirius** (Predicts fragmentation trees to generate molecular formula and structure candidate annotations for the same spectra)
- **MS-Finder** (Integrates spectral/structural parameters to produce formula and structure candidate annotations)
- **RDKit** (Converts library structures to InChIKeys used to build the in-silico database and deduplicate structures/stereoisomers)

## Examples

```
docker pull adafede/tima-r
```

## Evaluation signals

- Verify that filtering retained only spectra with [M+H]+ adducts, parent masses between 100 and 1,500 Da, at least 6 fragments, and the 500 most intense peaks per spectrum.
- Check that reproduced rank-1 correct counts match the reported baselines: ISDB-DNP 214 (10.2%), Sirius 975 (46.3%), MS-Finder 180 (8.5%).
- Confirm duplicate structures and stereoisomers were removed by keeping distinct InChIKeys according to the first layer (first 14 characters).
- Benchmarking dataset size must equal 2,107 distinct entries after filtering.
- F1 score improvements of 1.5–7× should only appear after taxonomically informed re-ranking, not in the raw tool output.

## Limitations

- GNPS spectral libraries were part of Sirius's training set, confounding fair tool comparison.
- 197 benchmarking entries have large mass differences (>0.01 Da) between experimental parent ion mass and calculated exact mass, and annotation tools cannot identify these erroneous entries.
- The optimized score combination is heavily dependent on the training sets and should not be taken as absolute optimal values.
- Taxonomically informed scoring will mostly benefit annotation of specialized metabolites, not ubiquitous molecules.

## Evidence

- [discussion] Baseline rank-1 counts for the three tools: [evidence span withheld — anchor status: not_found]
- [discussion] Spectral filtering criteria: "filtered to retain only entries having at least 6 fragments. For spectra containing more than 500 fragments, only the 500 most intense were kept"
- [discussion] Parent mass range filter: "filtered to keep entries which parent masses were comprised between 100 and 1,500 Da"
- [discussion] Structural deduplication rule: "Duplicate structures and stereoisomers were removed by keeping distinct InChIKey according to the first layer (first 14 characters)"
- [discussion] Benchmarking bias caveat: "all compounds of the benchmarking dataset are present in the DNP, and the ISDB-DNP tool, which is by definition backed by the same DB is thus favored"
