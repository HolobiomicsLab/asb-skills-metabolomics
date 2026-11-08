---
name: msms-spectral-library-search
description: Use when untargeted LC-MS/MS feature spectra (mzML/mgf from raw vendor
  files) must be annotated against a drug reference library to detect parent drugs,
  metabolized analogs, or drug exposures in biological or environmental samples.
when_to_use_negative:
- The input is already a curated annotation or feature table with drug identities
  — re-searching the library adds nothing
- Immediate clinical decision-making is the goal — the GNPS Drug Library is explicitly
  not intended for that
- Annotating general (non-drug) metabolites against taxonomy- or food-specific references
  — use the domain-specific MASSTs (microbeMASST, plantMASST, foodMASST, etc.) instead
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS Drug Library
  role: Reference MS/MS spectral library (99,122 spectra, 4723 drugs, metadata for
    4560 drugs) used as the search target for drug annotation
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- name: MZmine
  role: Feature detection, deconvolution, alignment, and MS/MS spectral extraction
    from raw LC-MS/MS files to produce .mgf spectra for library search
  repo: https://github.com/mzmine/mzmine
- name: microbe_masst (jobs.py batch search)
  role: Batch multi-spectrum search against GNPS/MassIVE, Metabolomics Workbench,
    Metabolights, and NORMAN via the Fast Search API, with cosine score, m/z tolerance,
    and minimum matching peaks as parameters
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_004
  - task_008
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:837025fcaf2411d120f18a8464e7b9867b713525ae724bdaddfa0e1fef7d2dbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
  tools:
  - GNPS Drug Library
  - MZmine
  - microbe_masst (jobs.py batch search)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# msms-spectral-library-search

## Summary

Matches experimental MS/MS spectra from untargeted metabolomics runs against the GNPS Drug Library (99,122 reference spectra covering 4723 unique drugs and their metabolites/analogs) to empirically read out drug exposures per sample, with standardized pharmacologic metadata attached to each hit.

## When to use

Use when untargeted LC-MS/MS feature spectra (mzML/mgf from raw vendor files) must be annotated against a drug reference library to detect parent drugs, metabolized analogs, or drug exposures in biological or environmental samples.

## When NOT to use

- The input is already a curated annotation or feature table with drug identities — re-searching the library adds nothing
- Immediate clinical decision-making is the goal — the GNPS Drug Library is explicitly not intended for that
- Annotating general (non-drug) metabolites against taxonomy- or food-specific references — use the domain-specific MASSTs (microbeMASST, plantMASST, foodMASST, etc.) instead

## Inputs

- Raw untargeted LC-MS/MS metabolomics runs from MassIVE (e.g., MSV000085944, MSV000084008, MSV000082493, MSV000097575, MSV000080673) as vendor files or mzML/mzXML
- Feature-extracted MS/MS spectra (e.g., .mgf from MZmine) or a list of USIs in .csv/.tsv
- GNPS Drug Library reference spectra (drugs plus metabolites/analogs) with machine-readable pharmacologic metadata

## Outputs

- Per-sample drug annotation table (drug-by-sample hits with pharmacologic class and exposure source, e.g., drug_annotations.csv)
- Per-drug, per-matrix detection frequencies (e.g., drug_detection_frequencies.csv) with flags for cells deviating from reported values
- Domain-MASST search outputs: _matches.tsv, _library.tsv, _datasets.tsv, _count_domain.tsv, and interactive HTML/JSON tree files

## How to apply

Convert or load raw vendor/mzML files and run molecular feature extraction (e.g., MZmine feature detection, deconvolution, alignment, MS/MS spectral extraction) so each sample yields MS/MS spectra of chromatographic peaks suitable for library search. Match each spectrum against the GNPS Drug Library reference spectra via spectral similarity search using cosine score with precursor and MS/MS peak mass tolerances per GNPS library-search conventions; for batch multi-spectrum search the microbe_masst jobs.py wrapper exposes minimum cosine score, m/z tolerance, and minimum matching peaks as tunable parameters. For each sample, record the annotated drug (or its metabolite/analog) together with its pharmacologic class, exposure source, and mechanism-of-action metadata. Aggregate hits per drug and per matrix (plasma, feces, wastewater, food) and compute detection frequencies, e.g. midazolam rising from 46% to 69% in plasma and omeprazole from 0% to 21.4% in feces when metabolites/analogs are included. Validate by confirming computed frequencies and cohort-level associations (e.g., NSAIDs more frequent in females, chi-square p = 8.5e-05) reproduce the reference values at reporting resolution.

## Related tools

- **GNPS Drug Library** (Reference MS/MS spectral library (99,122 spectra, 4723 drugs, metadata for 4560 drugs) used as the search target for drug annotation) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
- **MZmine** (Feature detection, deconvolution, alignment, and MS/MS spectral extraction from raw LC-MS/MS files to produce .mgf spectra for library search) — https://github.com/mzmine/mzmine
- **microbe_masst (jobs.py batch search)** (Batch multi-spectrum search against GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN via the Fast Search API, with cosine score, m/z tolerance, and minimum matching peaks as parameters) — https://github.com/robinschmid/microbe_masst

## Examples

```
python code/jobs.py
```

## Evaluation signals

- Computed drug detection frequencies match reported values within reporting resolution: midazolam 46% vs 69% in plasma, omeprazole 0% vs 21.4% in feces; deviating cells are flagged
- Library-level invariant: the curated library contains 4723 unique drugs and 99,122 MS/MS spectra with metadata populated for 4560 drugs
- Cohort-level association reproduces: NSAID/analgesic detections significantly more frequent in females (chi-square p = 8.5e-05) on the American Gut Project 1993-individual fecal cohort
- Seasonality check: per-month detection counts show the expected March-to-June 2020 decline of dextromethorphan and antibiotic detections in the n=55 wastewater influent samples
- Batch search hygiene: rerun jobs.py until no new output is generated with skip_existing=True, since the Fast Search API can silently miss entries on a single pass

## Limitations

- Existing reference libraries largely capture only the parent drug; missing metabolite spectra cause missed drug exposure, which is why the library adds metabolized analogs
- Not intended for immediate clinical decision-making; it supports data-driven exploration where traditional exposure records are limited or unreliable
- Conventional/self-reported comparisons miss OTC medications, dietary supplements, non-adherence, and undocumented drug use, so library-based readouts may diverge from medication records
- Batch search via the Fast Search API can fail silently on individual entries, requiring repeated runs until no new output appears

## Evidence

- [abstract] the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics"
- [intro] GNPS Drug Library contains a collection of MS/MS reference spectra for drugs and their metabolites/analogs along with standardized, machine-readable pharmacologic metadata: "GNPS Drug Library contains a collection of MS/MS reference spectra for drugs and their metabolites/analogs along with standardized, machine-readable pharmacologic metadata"
- [intro] it is common that only metabolized versions of a drug are present in the sample, leading to missed drug exposure if only the parent drug is considered24: "it is common that only metabolized versions of a drug are present in the sample, leading to missed drug exposure if only the parent drug is considered24"
- [readme] Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question.: [evidence span withheld — anchor status: external_doc]
- [readme] Due to the Fast Search API some of the entries will fail. Nevertheless sequent re-runs should catch all the possible matches.: [evidence span withheld — anchor status: external_doc]
