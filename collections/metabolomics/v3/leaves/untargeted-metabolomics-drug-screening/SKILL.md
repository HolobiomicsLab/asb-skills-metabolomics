---
name: untargeted-metabolomics-drug-screening
description: Use when untargeted metabolomics MS/MS runs (mzML or converted vendor
  files) must be screened for drug exposures, especially when only metabolized versions
  of a drug may be present rather than the parent compound.
when_to_use_negative:
- The input is already a feature table or annotation table rather than raw/MS/MS spectra
  needing library search
- The goal is clinical decision-making — the article states the GNPS Drug Library
  is 'not intended for immediate clinical decision-making'
- Targeted quantification of a drug with authentic standards is needed; this skill
  provides qualitative exposure screening via untargeted library matching
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Hosts the GNPS Drug Library of MS/MS reference spectra for drugs and their
    metabolites/analogs used to annotate drug exposures from untargeted metabolomics
    spectra
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- name: microbeMASST / domainMASSTs batch search (jobs.py)
  role: Batch-searches MS/MS spectra via the GNPS Fast Search API against indexed
    public data (GNPS/MassIVE, Metabolomics Workbench, Metabolights, NORMAN) to propagate
    drug-spectrum matches
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:aded5c4fe11b87b3e0b7c622ccd73a263c916f9a7f9852fc0dd37fd7aef5b363
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - microbeMASST / domainMASSTs batch search (jobs.py)
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# untargeted-metabolomics-drug-screening

## Summary

Screens untargeted metabolomics MS/MS spectra against the GNPS Drug Library — which contains reference spectra of parent drugs and their metabolized analogs with machine-readable pharmacologic metadata — to empirically and retroactively establish drug exposure records from biological or environmental samples.

## When to use

Use when untargeted metabolomics MS/MS runs (mzML or converted vendor files) must be screened for drug exposures, especially when only metabolized versions of a drug may be present rather than the parent compound.

## When NOT to use

- The input is already a feature table or annotation table rather than raw/MS/MS spectra needing library search
- The goal is clinical decision-making — the article states the GNPS Drug Library is 'not intended for immediate clinical decision-making'
- Targeted quantification of a drug with authentic standards is needed; this skill provides qualitative exposure screening via untargeted library matching

## Inputs

- Raw vendor or mzML untargeted metabolomics files from MassIVE (e.g., MSV000085944, MSV000084008, MSV000082493)
- Molecular feature extraction output (.mgf MS/MS spectra from MZmine or GNPS molecular networking)
- USI lists of spectra (.csv or .tsv) for batch Fast Search API queries

## Outputs

- Per-sample drug/metabolite annotation hit lists (drug_detection_frequencies.csv)
- Drug detection frequencies per drug and biofluid matrix (fraction of positive samples)
- Flags for drug-matrix frequency cells deviating from reported values

## How to apply

Download the untargeted metabolomics datasets (e.g., MassIVE accessions MSV000085944, MSV000084008, MSV000082493) and run molecular feature extraction so each sample yields MS/MS spectra of chromatographic peaks. Search those spectra against the GNPS Drug Library reference spectra using the library's precursor mass tolerance and MS/MS peak matching tolerances to annotate parent drugs and their metabolized analogs per sample. For each drug of interest (e.g., diphenhydramine, caffeine, midazolam, omeprazole), tally the samples in which the drug or one of its metabolites is detected per matrix (plasma, feces, etc.) and compute detection frequencies as the fraction of positive samples. Compare computed frequencies against expected values (e.g., midazolam 46% vs 69% in plasma when metabolites are included, omeprazole 0% vs 21.4% in feces), writing per-sample hits to drug_detection_frequencies.csv and flagging any drug-matrix cell that deviates. Optionally use the GNPS Fast Search API via the domainMASST jobs.py batch script (Python 3.10) to propagate spectral matches across indexed public datasets.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Hosts the GNPS Drug Library of MS/MS reference spectra for drugs and their metabolites/analogs used to annotate drug exposures from untargeted metabolomics spectra) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- **microbeMASST / domainMASSTs batch search (jobs.py)** (Batch-searches MS/MS spectra via the GNPS Fast Search API against indexed public data (GNPS/MassIVE, Metabolomics Workbench, Metabolights, NORMAN) to propagate drug-spectrum matches) — https://github.com/robinschmid/microbe_masst

## Examples

```
python code/jobs.py  # batch-search .mgf/USI spectra via GNPS Fast Search API (Python 3.10; re-run with skip_existing=True until no new output)
```

## Evaluation signals

- Computed drug detection frequencies match reported values within the paper's reporting resolution (midazolam 46% and 69% in plasma; omeprazole 0% and 21.4% in feces)
- Per-sample hit list (drug_detection_frequencies.csv) reconciles with the tallied fraction of positive samples per drug-matrix cell
- Metabolite/analog-inclusive detection frequency is >= parent-drug-only frequency for every drug-matrix cell (inclusion can only add hits)
- Library search parameters (precursor mass tolerance, MS/MS peak tolerance) match those used by the GNPS Drug Library

## Limitations

- Only metabolized versions of a drug may be present in a sample, so parent-drug-only libraries miss exposures — hence the need for metabolite/analog references
- The GNPS Drug Library is not intended for immediate clinical decision-making; it supports exploratory, data-driven exposure readouts
- When using the Fast Search API batch script, some entries may fail and jobs.py must be re-run (skip_existing=True) until no new output is generated; Python 3.10 is required

## Evidence

- [abstract] the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics"
- [intro] Metabolized-only detection gap: "it is common that only metabolized versions of a drug are present in the sample, leading to missed drug exposure if only the parent drug is considered"
- [intro] Parent-drug-only library gap: "existing MS/MS reference libraries capture only the parent drug due to challenges in sourcing drug metabolite reference standards"
- [readme] README on accepted batch-search inputs: [evidence span withheld — anchor status: not_found]
