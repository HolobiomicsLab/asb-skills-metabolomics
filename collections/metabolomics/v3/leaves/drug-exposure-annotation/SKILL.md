---
name: drug-exposure-annotation
description: Use when untargeted metabolomics data (raw LC-MS/MS files, .mgf/.mzML/.mzXML,
  or MZmine feature tables) must be screened for drug exposures without clinical records,
  self-reports, or medication tracking.
when_to_use_negative:
- The goal is immediate clinical decision-making — the article explicitly states the
  GNPS Drug Library is not intended for that purpose
- The input data are already a fully annotated feature table with drug identities
  resolved, so spectral library matching would be redundant
- Only MS1 data are available (no MS/MS spectra), since the library is a tandem MS/MS-based
  resource and cannot match precursor-only features
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Hosts the GNPS Drug Library and performs spectral similarity search (cosine
    score, mass-bmatch tolerance) of sample MS/MS spectra against drug/metabolite
    reference spectra with pharmacologic metadata
- name: MZmine
  role: Feature detection, deconvolution, alignment, and MS/MS spectral extraction
    from raw LC-MS/MS files to produce the consensus feature table and .mgf spectra
    for library matching
  repo: https://github.com/mzmine/mzmine
- name: microbe_masst (domainMASSTs)
  role: Batch search of multiple MS/MS spectra via the GNPS Fast Search API (jobs.py)
    against indexed public data in GNPS/MassIVE, Metabolomics Workbench, Metabolights,
    and NORMAN
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:ff00d906a9530ad365c6e46270fdbbe8d58e56abacc8138abfeed86555bd02aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzmine/mzmine
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - MZmine
  - microbe_masst (domainMASSTs)
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# drug-exposure-annotation

## Summary

Empirically establish drug exposure records from untargeted metabolomics data by matching LC-MS/MS spectra against the GNPS Drug Library, whose reference spectra cover drugs, their metabolites/analogs, and machine-readable pharmacologic metadata (exposure source, pharmacologic class, therapeutic indication).

## When to use

Use when untargeted metabolomics data (raw LC-MS/MS files, .mgf/.mzML/.mzXML, or MZmine feature tables) must be screened for drug exposures without clinical records, self-reports, or medication tracking.

## When NOT to use

- The goal is immediate clinical decision-making — the article explicitly states the GNPS Drug Library is not intended for that purpose
- The input data are already a fully annotated feature table with drug identities resolved, so spectral library matching would be redundant
- Only MS1 data are available (no MS/MS spectra), since the library is a tandem MS/MS-based resource and cannot match precursor-only features

## Inputs

- Raw LC-MS/MS files (.mzML/.mzXML) from MassIVE datasets, e.g., MSV000080673 (American Gut Project, n=1993), MSV000097575 (wastewater influent, n=55, March–June 2020), Global FoodOmics (~3500 foods/beverages)
- MZmine-generated consensus feature table with deconvoluted, aligned features and extracted MS/MS spectra (.mgf)
- GNPS Drug Library reference MS/MS spectra with machine-readable pharmacologic metadata (exposure source, pharmacologic class, therapeutic indication)
- Sample metadata (age, sex, collection month, food category) for stratified exposure analysis

## Outputs

- Drug-by-sample annotation table (drug_annotations.csv) with pharmacologic class and exposure source per hit
- Per-individual drug exposure table and pharmacologic class frequency counts for the cohort
- Demographic association statistics (chi-square tests, e.g., NSAID female association p = 8.5e-05)
- Batch-search outputs: _matches.tsv, _library.tsv, _datasets.tsv, and _count_domain.tsv files

## How to apply

Acquire raw LC-MS/MS files (e.g., MassIVE accessions MSV000080673 for the American Gut Project cohort of 1993 individuals, MSV000097575 for 55 wastewater influent samples, or Global FoodOmics' ~3500 foods) and run MZmine feature detection, deconvolution, alignment, and MS/MS spectral extraction to build a consensus feature table. Match sample MS/MS spectra against GNPS Drug Library reference spectra (drugs plus metabolites/analogs) via spectral similarity search using cosine score with mass-bmatch tolerance per GNPS library search conventions, so hits carry pharmacologic class and exposure-source metadata rather than bare compound names. Aggregate annotations per individual/sample into a drug-by-sample table, then tally pharmacologic class frequencies and stratify by metadata (age, sex, collection month, food category) with chi-square tests — e.g., analgesics/NSAIDs were significantly more frequent in females (p = 8.5e-05) and dextromethorphan detections declined March–June 2020 in wastewater. Because only metabolized versions of a drug may be present, the metabolite/analog coverage of the library is what prevents missed exposures that parent-drug-only libraries would incur. For batch re-analysis of indexed public data, the microbe_masst repository's jobs.py leverages the GNPS Fast Search API to search many spectra at once against GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN, emitting _matches.tsv, _library.tsv, and _datasets.tsv outputs.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Hosts the GNPS Drug Library and performs spectral similarity search (cosine score, mass-bmatch tolerance) of sample MS/MS spectra against drug/metabolite reference spectra with pharmacologic metadata)
- **MZmine** (Feature detection, deconvolution, alignment, and MS/MS spectral extraction from raw LC-MS/MS files to produce the consensus feature table and .mgf spectra for library matching) — https://github.com/mzmine/mzmine
- **microbe_masst (domainMASSTs)** (Batch search of multiple MS/MS spectra via the GNPS Fast Search API (jobs.py) against indexed public data in GNPS/MassIVE, Metabolomics Workbench, Metabolights, and NORMAN) — https://github.com/robinschmid/microbe_masst

## Examples

```
python code/jobs.py  # files list entries: ("input_directory/input_file", "output_directory/output_prefix"); min cosine, mz tolerance, min matching peaks adjustable
```

## Evaluation signals

- Pharmacologic class frequencies of matched annotations reproduce reference distributions, e.g., 89 drugs detected in the American Gut cohort whose classes match common prescriptions
- Known demographic association is recovered: chi-square test yields p = 8.5e-05 for NSAID/paracetamol analgesics being more frequent in females
- Seasonality trend reproduces from the per-sample table: declining dextromethorphan and antibiotic detection counts March–June 2020 in wastewater influent
- Food-category detections appear as expected: ampicillin/tetracycline in fish/beef/turkey and spinosad/thiabendazole in produce categories
- Batch-search runs produce complete output sets (_matches.tsv, _library.tsv, _datasets.tsv per searched spectrum) with no residual matches after re-running jobs.py until no new output is generated

## Limitations

- Not intended for immediate clinical decision-making; it is a research-grade empirical readout
- Conventional exposure assessment misses over-the-counter medications, dietary supplements, non-adherence, undocumented drug use, and unrealized food-supply exposures — the library readout is empirical and does not reconstruct intent or dose
- Existing MS/MS libraries historically capture only parent drugs because sourcing drug metabolite reference standards is difficult; coverage of metabolites/analogs determines sensitivity to metabolized-only samples
- The GNPS Fast Search API can intermittently fail batch-search entries, so jobs.py must be re-run (skip_existing=True) a couple of times until no new output appears

## Evidence

- [abstract] GNPS Drug Library as MS/MS-based drug-screening resource: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics"
- [intro] Library contents with machine-readable metadata: "GNPS Drug Library contains a collection of MS/MS reference spectra for drugs and their metabolites/analogs along with standardized, machine-readable pharmacologic metadata"
- [abstract] Drug-exposure stratification outperforms self-report: "We uncover drug-associated alterations in microbiota-derived N-acyl lipids that are not captured when stratifying by self-reported medication use."
- [readme] Batch search mechanism from README: [evidence span withheld — anchor status: external_doc]
- [intro] Parent-drug-only libraries miss metabolized exposures: "it is common that only metabolized versions of a drug are present in the sample, leading to missed drug exposure if only the parent drug is considered"
