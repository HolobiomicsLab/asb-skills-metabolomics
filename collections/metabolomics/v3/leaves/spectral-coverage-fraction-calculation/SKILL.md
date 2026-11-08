---
name: spectral-coverage-fraction-calculation
description: Use when you have counts of merged MS/MS spectra and of MSI level 1/2
  compound annotations for an untargeted metabolomics assay (e.g. urinary HILIC-Q
  Exactive HF data) and need to report the fraction of spectra covered and the unannotated
  remainder before applying in silico annotation.
when_to_use_negative:
- Your input is already per-spectrum annotation results from CSI:FingerID or NIST
  hybrid search at MSI level 3 — this skill only quantifies coverage of the stricter
  MSI level 1/2 annotations, not level 3 outcomes.
- You need false-discovery-rate assessment or automated confidence scoring of compound
  identifications; the article notes the field lacks generally accepted validated
  automated calculations of identification confidence levels with FDR.
- You want to annotate individual spectra rather than summarize dataset-level annotation
  coverage.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIRIUS
  role: Framework whose integrated CSI:FingerID / NIST hybrid search produce the MSI
    level 2 and 3 annotations whose spectral coverage is being quantified; also the
    downstream step applied to the unannotated fraction this skill exposes.
  repo: https://github.com/sirius-ms/sirius
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:2c77f98f6bf317b8c0697dc425ca0452fbc3f64c6cb830f224ce9a67a455335e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - SIRIUS
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# spectral-coverage-fraction-calculation

## Summary

Quantifies how much of the acquired MS/MS spectral space in an untargeted metabolomics dataset is annotated by compounds at Metabolomics Standards Initiative (MSI) levels 1 and 2, reporting the covered fraction and the residual unannotated fraction that motivates MSI level 3 annotation strategies such as CSI:FingerID and NIST hybrid search.

## When to use

Use when you have counts of merged MS/MS spectra and of MSI level 1/2 compound annotations for an untargeted metabolomics assay (e.g. urinary HILIC-Q Exactive HF data) and need to report the fraction of spectra covered and the unannotated remainder before applying in silico annotation.

## When NOT to use

- Your input is already per-spectrum annotation results from CSI:FingerID or NIST hybrid search at MSI level 3 — this skill only quantifies coverage of the stricter MSI level 1/2 annotations, not level 3 outcomes.
- You need false-discovery-rate assessment or automated confidence scoring of compound identifications; the article notes the field lacks generally accepted validated automated calculations of identification confidence levels with FDR.
- You want to annotate individual spectra rather than summarize dataset-level annotation coverage.

## Inputs

- Merged-spectrum annotation table (e.g. Table S2) with counts of MSI level 1 and level 2 annotated spectra out of total merged MS/MS spectra (numerator 753, denominator 3894 in the source dataset)
- Optionally, per-compound annotation lists at MSI level 1 and level 2 to re-derive the annotated spectrum count

## Outputs

- coverage_summary.csv containing the annotated-spectrum count, total merged-spectrum count, coverage fraction (≈0.193), and unannotated fraction (≈0.807)

## How to apply

Load a merged-spectrum annotation summary (e.g. Table S2) that records the total number of merged urinary MS/MS spectra and the number of those spectra carrying an MSI level 1 or level 2 compound annotation (753 of 3,894 in the source study). Compute the coverage fraction as the annotated count divided by the total merged-spectrum count using simple Python (pandas) arithmetic, and compute the residual unannotated fraction as (total − annotated) / total. Write the numerator, denominator, and both fractions to a small summary CSV (e.g. coverage_summary.csv). Rationale: the article reports that MSI level 1 (175 compounds) plus level 2 (578 compounds) annotations cover only 19.3% of acquired MS/MS spectra, leaving >80% unannotated — this quantification is the explicit motivation for extending annotation to MSI level 3 via CSI:FingerID and the NIST hybrid search.

## Related tools

- **SIRIUS** (Framework whose integrated CSI:FingerID / NIST hybrid search produce the MSI level 2 and 3 annotations whose spectral coverage is being quantified; also the downstream step applied to the unannotated fraction this skill exposes.) — https://github.com/sirius-ms/sirius

## Examples

```
import pandas as pd; ann = 753; tot = 3894; cov = ann/tot; un = (tot-ann)/tot; pd.DataFrame({'annotated':[ann],'total':[tot],'coverage':[cov],'unannotated':[un]}).to_csv('coverage_summary.csv', index=False)
```

## Evaluation signals

- Computed coverage fraction must reproduce the paper's reported 19.3% (753/3894 ≈ 0.193) within rounding tolerance.
- Computed unannotated fraction must equal 1 − coverage (≈0.807), confirming that more than 80% of spectra remain without MSI level 1/2 annotation.
- Numerator and denominator in coverage_summary.csv must match the merged-spectrum annotation source (753 annotated of 3894 total).
- Summary table must contain both fractions plus their numerator and denominator so the calculation is auditable.
- Cross-check that level 1 (175 compounds) + level 2 (578 compounds) = 753 annotations, consistent with the reported MSI level 1/2 totals.

## Limitations

- Coverage is defined only against MSI level 1 and level 2 annotations; it does not reflect the additional coverage that CSI:FingerID or NIST hybrid search MSI level 3 annotations would provide.
- Requires a precomputed merged-spectrum count and annotation counts (e.g. from Table S2); it cannot be derived from raw vendor mzML alone without first performing annotation.
- The article's overall analysis found the majority of acquired MS/MS spectra cannot be annotated with certainty to specific chemicals, so a low coverage fraction may simply reflect the current limits of authentic-standard-based identification rather than assay failure.
- MSI level 3 classification outcomes (ClassyFire superclasses/classes) that would be needed to characterize the unannotated remainder are outside the scope of this arithmetic skill.

## Evidence
