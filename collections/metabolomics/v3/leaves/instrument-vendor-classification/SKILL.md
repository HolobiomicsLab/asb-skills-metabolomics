---
name: instrument-vendor-classification
description: Use when you need to stratify repository studies by acquisition instrument
  vendor (e.g., Bruker vs Agilent/Varian) or by sample type from raw NMR data available
  in Metabolomics Workbench.
when_to_use_negative:
- The study exposes only processed spectra, not raw vendor acquisition files — vendor
  identity cannot be reliably read and the raw-data count (41) cannot be reproduced
- You are analyzing LC–MS or GC–MS studies rather than the NMR-based metabolomics
  data CloMet targets
- You need class metadata for downstream classification re-analysis — the article
  notes that when classes are absent from repositories, such re-analysis is infeasible
  regardless of vendor classification
edam_operation: http://edamontology.org/operation_3222
tools:
- name: CloMet
  role: Harmonizes the non-standardized file systems of public repository data sets
    so vendor acquisition files can be consistently located and read; deployed via
    Docker
  repo: https://github.com/rmallol/clomet
- name: Metabolomics Workbench
  role: Source repository queried via its REST API to obtain the study catalog (1592
    studies) and identify NMR, human, and raw-data subsets
  repo: https://www.metabolomicsworkbench.org/rest
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.2c00602
    title: 'CloMet: A Novel Open-Source and Modular Software Platform That Connects
      Established Metabolomics Repositories and Data Analysis Resources'
  source_content_hash: sha256:189fb5647de186306f67eab0d78e05fc3b28b27ec7a4dfc0e4b8d9fc97e59411
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/rmallol/clomet
  tools:
  - CloMet
  - Metabolomics Workbench
  edam_operation: http://edamontology.org/operation_3222
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# instrument-vendor-classification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Classifies public NMR-based metabolomics studies by instrument vendor (Bruker vs Agilent/Varian) by inspecting raw vendor acquisition files retrieved from the Metabolomics Workbench REST API, enabling repository survey breakdowns like those reported in the CloMet paper.

## When to use

Use when you need to stratify repository studies by acquisition instrument vendor (e.g., Bruker vs Agilent/Varian) or by sample type from raw NMR data available in Metabolomics Workbench.

## When NOT to use

- The study exposes only processed spectra, not raw vendor acquisition files — vendor identity cannot be reliably read and the raw-data count (41) cannot be reproduced
- You are analyzing LC–MS or GC–MS studies rather than the NMR-based metabolomics data CloMet targets
- You need class metadata for downstream classification re-analysis — the article notes that when classes are absent from repositories, such re-analysis is infeasible regardless of vendor classification

## Inputs

- Metabolomics Workbench REST API study catalog (JSON) from https://www.metabolomicsworkbench.org/rest
- Study metadata fields: analysis type (NMR), organism (Homo sapiens), sample type (blood plasma, blood serum)
- Raw vendor acquisition files attached to human NMR studies

## Outputs

- JSON of reproduced survey counts (workbench_survey_counts.json) with values 1592, 166, 61, 41, 9, 7, 33, 8
- Per-study instrument vendor labels (Bruker, Agilent/Varian) for the human NMR raw-data subset

## How to apply

Query the Metabolomics Workbench REST API (https://www.metabolomicsworkbench.org/rest) study endpoints to retrieve the full study catalog in JSON form, then count total studies (expected 1592). Filter by analysis type 'NMR' (expected 166) and by organism 'Homo sapiens' (expected 61). Within the human NMR subset, restrict to studies that provide raw vendor acquisition files (not just processed spectra) and read vendor identity from those files to split them into Bruker (33) and Agilent/Varian (8), stratifying simultaneously by sample type (blood plasma 9, blood serum 7). The rationale is that CloMet harmonizes the non-standardized file systems of public repositories, so vendor classification must be done on the original acquisition files before format conversion.

## Related tools

- **CloMet** (Harmonizes the non-standardized file systems of public repository data sets so vendor acquisition files can be consistently located and read; deployed via Docker) — https://github.com/rmallol/clomet
- **Metabolomics Workbench** (Source repository queried via its REST API to obtain the study catalog (1592 studies) and identify NMR, human, and raw-data subsets) — https://www.metabolomicsworkbench.org/rest

## Examples

```
python -c "import urllib.request, json; d=json.load(urllib.request.urlopen('https://www.metabolomicsworkbench.org/rest/study/study_id/summary/')); print(len(d))"
```

## Evaluation signals

- Total study count equals 1592, matching the paper's survey
- NMR-filtered count equals 166; Homo sapiens subset within NMR equals 61
- Raw-data subset equals 41, stratified exactly as 9 blood plasma, 7 blood serum
- Vendor split of selected experiments equals 33 Bruker and 8 Agilent/Varian
- Any mismatched count requires re-checking the REST API query filters before reporting

## Limitations

- Survey counts are a snapshot: the repository grows over time, so re-running later may no longer reproduce 1592/166/61/41
- Vendor classification depends entirely on raw acquisition files being deposited; processed-only studies are unverifiable
- If class/sample metadata is missing alongside the data in public repositories, downstream re-analysis beyond vendor counting is not feasible
- CloMet does not itself implement data processing algorithms; extension relies on the open-source developer community

## Evidence
