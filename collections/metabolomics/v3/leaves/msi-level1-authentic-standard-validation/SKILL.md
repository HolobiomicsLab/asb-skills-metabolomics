---
name: msi-level1-authentic-standard-validation
description: Use when you have MSI level 1 authentic-standard MS/MS identifications
  and need to quantify how accurately an in silico annotation tool (e.g., CSI:FingerID)
  ranks the true structure among its candidates.
when_to_use_negative:
- Spectra are not backed by authentic standards (e.g., MSI level 2 or level 3 annotations)
  — there is no ground-truth identity to validate against
- You need a confidence estimate for individual unknown spectra rather than a global
  accuracy benchmark of the annotation tool
- The acquisition platform, collision energies, or precursor mass tolerance differ
  substantially from the 5 ppm HILIC-Q Exactive HF conditions the benchmark was computed
  under
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CSI:FingerID
  role: In silico fragmentation tool whose top-ranked structure predictions are benchmarked
    against MSI level 1 authentic-standard identifications
  repo: https://github.com/sirius-ms/sirius
- name: Sirius/CSI:Finger ID
  role: SIRIUS pipeline (GUI or CLI) used to compute fragmentation trees and submit
    spectra to the CSI:FingerID web service to retrieve ranked candidate structures
  repo: https://github.com/sirius-ms/sirius
- name: HILIC-Q Exactive HF mass spectrometry
  role: Instrument platform that acquired the MSI level 1 MS/MS spectra of authentic
    standards used as the validation ground truth
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.8b04698
    title: Structure Annotation of All Mass Spectra in Untargeted Metabolomics
  source_content_hash: sha256:9730db8d06c92b82aa5b9b775c9e1a896802d2d60a212451c98a66d147b57a7a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sirius-ms/sirius
  tools:
  - CSI:FingerID
  - HILIC-Q Exactive HF mass spectrometry
  - Sirius/CSI:Finger ID
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# msi-level1-authentic-standard-validation

## Summary

Validates in silico structure annotations (e.g., CSI:FingerID top-ranked hits) by benchmarking them against MSI confidence level 1 identifications established with authentic standards using precursor mass, retention time, and MS/MS matching. This yields a concrete top-hit accuracy figure for untargeted metabolomics annotation workflows.

## When to use

Use when you have MSI level 1 authentic-standard MS/MS identifications and need to quantify how accurately an in silico annotation tool (e.g., CSI:FingerID) ranks the true structure among its candidates.

## When NOT to use

- Spectra are not backed by authentic standards (e.g., MSI level 2 or level 3 annotations) — there is no ground-truth identity to validate against
- You need a confidence estimate for individual unknown spectra rather than a global accuracy benchmark of the annotation tool
- The acquisition platform, collision energies, or precursor mass tolerance differ substantially from the 5 ppm HILIC-Q Exactive HF conditions the benchmark was computed under

## Inputs

- 103 unambiguously assigned MSI level 1 MS/MS spectra from the HILIC-Q Exactive HF urinary assay (authentic-standard identifications with precursor mass, retention time, and MS/MS matching)
- SIRIUS-compatible MS/MS input files (mzML/MGF/.ms) for the level 1 compounds
- Structure search space (e.g., biodatabase) for CSI:FingerID candidate retrieval

## Outputs

- Per-compound agreement table: predicted top-ranked structure vs. authentic-standard identity, plus rank position of the correct structure among CSI:FingerID candidates
- Summary accuracy metrics: top-hit accuracy (40%, 41/103) and top-3 accuracy (52%) for the MSI level 1 compound set

## How to apply

Load the set of unambiguously assigned MSI level 1 MS/MS spectra — in this study, 103 such spectra acquired on the HILIC-Q Exactive HF platform. Submit each spectrum to CSI:FingerID via the SIRIUS pipeline (CLI or web service) using the biodatabase as the structure search space and a 5 ppm precursor mass tolerance. Retrieve the top-ranked candidate structures for each compound, then compare each predicted structure against the authentic-standard (MSI level 1) assignment for the same compound, recording agreement and the rank position of the correct structure among candidates. Summarize agreement rates across all compounds: in this dataset 41 of 103 compounds (40%) were correctly annotated as the top hit, and top-3 accuracy was 52%. This validation is required because Sirius/CSI:FingerID and NIST17 hybrid search assignments must be confirmed with authentic chemicals before being trusted.

## Related tools

- **CSI:FingerID** (In silico fragmentation tool whose top-ranked structure predictions are benchmarked against MSI level 1 authentic-standard identifications) — https://github.com/sirius-ms/sirius
- **Sirius/CSI:Finger ID** (SIRIUS pipeline (GUI or CLI) used to compute fragmentation trees and submit spectra to the CSI:FingerID web service to retrieve ranked candidate structures) — https://github.com/sirius-ms/sirius
- **HILIC-Q Exactive HF mass spectrometry** (Instrument platform that acquired the MSI level 1 MS/MS spectra of authentic standards used as the validation ground truth)

## Examples

```
sirius -i spectra_ms1.ms --output results --csi-fingerid-id --masstrace-tolerance-ppm 5
```

## Evaluation signals

- Computed top-hit accuracy over the 103 MSI level 1 spectra reproduces the published 40% (41/103) when querying against the biodatabase at 5 ppm mass accuracy
- Computed top-3 accuracy reproduces the published 52% for the same compound set
- Every compound in the validation set has an MSI level 1 assignment traceable to an authentic standard with precursor mass, retention time, and MS/MS matching (175 level 1 compounds were identified in the full study; the 103 unambiguous subset is used here)
- For each compound, the correct authentic-standard structure appears among the retrieved CSI:FingerID candidate list (or is recorded as absent) with an explicit rank position

## Limitations

- Only ~40% of MSI level 1 spectra are correctly annotated as the top hit — CSI:FingerID top-ranked predictions are not sufficient on their own and require validation with authentic chemicals
- Accuracy was measured on a specific platform (HILIC-Q Exactive HF) and search space (biodatabase, 5 ppm); other instruments or search spaces may shift the benchmark
- The field lacks generally accepted and validated automated calculations of compound identification confidence levels with false-discovery rate assessments in metabolomics MS/MS, so this benchmark does not generalize as a per-spectrum FDR

## Evidence

- [readme] SIRIUS/CSI:FingerID tool integration: [evidence span withheld — anchor status: external_doc]
