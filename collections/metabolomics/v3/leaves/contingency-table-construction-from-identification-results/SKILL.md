---
name: contingency-table-construction-from-identification-results
description: 'Decision support for judging whether DeepMASS identifications can be
  trusted for an unknown: use when you have identification results (e.g. LOOCV on
  662 KEGG compounds or external biomarker tests) and need to tabulate validity against
  presence of a structure-related metabolite in the database.'
when_to_use_negative:
- Input is raw MS/MS spectra needing identification in the first place — run the DeepMASS
  structural-similarity scoring workflow (example.py) before tabulating validity
- The unknown has no structurally related metabolite in the database and you expect
  a valid identification — the skill's premise is that validity depends on structural-neighbor
  availability
- You need to extend the identification range of an MS/MS database itself rather than
  evaluate identification validity of existing results
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: DeepMASS
  role: Produces the identification results (structural-similarity scores via deep
    neural network on MS/MS spectra) whose validity is tabulated in the contingency
    table
  repo: github.com/hcji/DeepMASS
- name: KEGG
  role: Reference database of 662 compounds whose MS/MS spectra supply the LOOCV cases
    and the known metabolites queried for structural relatedness
- name: Shimadzu LC-ESI-IT-TOF
  role: Instrument that generated external-test MS/MS spectra of male infertility
    biomarkers used as external-test identification cases
- name: Bruker Compact LC-ESI-QTOF
  role: Instrument that generated external-test MS/MS spectra of male infertility
    biomarkers used as external-test identification cases
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.8b05405
    title: Deep MS/MS-Aided Structural-Similarity Scoring for Unknown Metabolite Identification
  source_content_hash: sha256:796159b1613d4948dd8ba7aef22b59de52ffed765916a64f289f1e41739141f9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bruker Compact LC-ESI-QTOF
  - DeepMASS
  - KEGG
  - Shimadzu LC-ESI-IT-TOF
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# contingency-table-construction-from-identification-results

## Summary

Builds a validity-vs-presence contingency table from DeepMASS metabolite identification results to test whether identification of an unknown compound is valid when a structure-related metabolite exists in the reference database. It operationalizes the paper's central claim by cross-tabulating LOOCV and external-test identification outcomes against structural-neighbor database coverage.

## When to use

Decision support for judging whether DeepMASS identifications can be trusted for an unknown: use when you have identification results (e.g. LOOCV on 662 KEGG compounds or external biomarker tests) and need to tabulate validity against presence of a structure-related metabolite in the database.

## When NOT to use

- Input is raw MS/MS spectra needing identification in the first place — run the DeepMASS structural-similarity scoring workflow (example.py) before tabulating validity
- The unknown has no structurally related metabolite in the database and you expect a valid identification — the skill's premise is that validity depends on structural-neighbor availability
- You need to extend the identification range of an MS/MS database itself rather than evaluate identification validity of existing results

## Inputs

- DeepMASS LOOCV identification results for MS/MS spectra of 662 KEGG compounds
- DeepMASS external-test identification results for male infertility biomarkers (Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF spectra)
- Reference database (KEGG) with known metabolite structures

## Outputs

- Contingency table: identification validity (valid/invalid) x structure-related database metabolite presence (present/absent)
- Cell counts for 'present & valid' and 'absent & invalid' supporting the validity claim

## How to apply

Load the leave-one-out cross-validation identification results from DeepMASS's evaluation on MS/MS spectra of 662 KEGG compounds, plus the external-test results for male infertility biomarkers measured on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF. For each case (LOOCV and external test), determine whether a structure-related metabolite of the unknown is present in the reference database. Tabulate identification validity (valid/invalid) against structure-related database metabolite presence (present/absent) to build a contingency table. The table supports the paper's finding that identification of an unknown compound is valid if its structure-related metabolite is available in the database; the structure-similarity rationale comes from DeepMASS's deep-neural-network scoring of MS/MS spectra against known metabolites. Compare the 'present & valid' and 'absent & invalid' cell counts against the reported finding to confirm reproduction.

## Related tools

- **DeepMASS** (Produces the identification results (structural-similarity scores via deep neural network on MS/MS spectra) whose validity is tabulated in the contingency table) — github.com/hcji/DeepMASS
- **KEGG** (Reference database of 662 compounds whose MS/MS spectra supply the LOOCV cases and the known metabolites queried for structural relatedness)
- **Shimadzu LC-ESI-IT-TOF** (Instrument that generated external-test MS/MS spectra of male infertility biomarkers used as external-test identification cases)
- **Bruker Compact LC-ESI-QTOF** (Instrument that generated external-test MS/MS spectra of male infertility biomarkers used as external-test identification cases)

## Examples

```
python example.py  # run DeepMASS identification, then tabulate valid/invalid vs structure-related-metabolite present/absent for the 662 KEGG LOOCV and male-infertility external-test results
```

## Evaluation signals

- 'present & valid' cell dominates the contingency table, matching the paper's claim that identification is valid when a structure-related metabolite is available
- 'absent & invalid' cell dominates the complementary column, consistent with the reported finding
- LOOCV cases come from exactly 662 KEGG compounds and the external-test cases come from the male infertility biomarker study (Shimadzu IT-TOF and Bruker QTOF spectra)
- Every case in the table is assigned both a validity label and a structure-related-presence label with no missing classifications

## Limitations

- Validity of identification depends on availability of a structure-related metabolite in the database; without one, identification is not valid
- The experimental spectra (MetDNA dataset) have been removed from the repository at the dataset owner's request, so the pipeline cannot be run directly; users must train their own model with an in-house database or wait for public-database spectra to be included
- The evaluation covers only 662 KEGG compounds (LOOCV) and one external biomarker cohort, so the validity claim is bounded by that scope

## Evidence

- [abstract] validity finding: "Results show that the identification of unknown compound is valid if its structure-related metabolite is available in the database"
- [abstract] LOOCV and external test setup: "We evaluated DeepMASS with leave-one-out cross-validation on MS/MS spectra of 662 compounds in KEGG"
- [abstract] external test instrument context: "biomarkers from male infertility study measured on Shimadzu LC-ESI-IT-TOF and Bruker Compact LC-ESI-QTOF"
- [abstract] structural-similarity basis: "These structurally related metabolites often have related MS/MS spectra, which provide the possibility to identify unknown metabolites through known ones"
- [readme] dataset removal caveat: [evidence span withheld — anchor status: external_doc]
