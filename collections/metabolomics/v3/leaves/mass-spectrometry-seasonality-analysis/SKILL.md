---
name: mass-spectrometry-seasonality-analysis
description: Use when a metabolomics dataset (e.g. mzML/mzXML from MassIVE MSV000097575
  wastewater influent, n=55) has time-stamped samples and you need to quantify temporal
  drug-exposure trends from untargeted MS/MS annotations.
when_to_use_negative:
- Datasets lack sample collection timestamps, so no month-level detection trend can
  be computed — use plain drug screening instead of seasonality analysis.
- The goal is immediate clinical decision-making — the GNPS Drug Library is explicitly
  not intended for that purpose.
- Only parent-drug reference spectra are available and the samples contain predominantly
  metabolized drug versions — existing libraries that capture only the parent drug
  will miss these exposures.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Provides the Drug Library MS/MS reference spectra and library-search spectral
    matching (cosine score) used to annotate drug exposures per sample before month-level
    aggregation.
- name: microbe_masst (domainMASSTs)
  role: Batch MS/MS search of annotated spectra against indexed public metabolomics
    data via the Fast Search API (jobs.py), with a _matches.tsv output listing scans
    matching a searched spectrum — usable for retrospective re-analysis of archived
    datasets.
  repo: https://github.com/robinschmid/microbe_masst
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:54be13e0722a1475d94d01905b535d6a217429a0f8184787430ae3aa99c7b368
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - microbe_masst (domainMASSTs)
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mass-spectrometry-seasonality-analysis

## Summary

Quantifies temporal trends in drug exposures by aggregating GNPS Drug Library MS/MS spectral annotations per sample collection month, e.g. detecting the March-to-June 2020 decline of dextromethorphan and antibiotic detections in wastewater influent. This turns per-sample library-search hits into a month-by-month detection-count time series, enabling empirical exposure readouts where medical records do not exist.

## When to use

Use when a metabolomics dataset (e.g. mzML/mzXML from MassIVE MSV000097575 wastewater influent, n=55) has time-stamped samples and you need to quantify temporal drug-exposure trends from untargeted MS/MS annotations.

## When NOT to use

- Datasets lack sample collection timestamps, so no month-level detection trend can be computed — use plain drug screening instead of seasonality analysis.
- The goal is immediate clinical decision-making — the GNPS Drug Library is explicitly not intended for that purpose.
- Only parent-drug reference spectra are available and the samples contain predominantly metabolized drug versions — existing libraries that capture only the parent drug will miss these exposures.

## Inputs

- Time-stamped untargeted metabolomics mzML/mzXML files (e.g. wastewater influent dataset MassIVE MSV000097575, n=55 samples, March-June 2020)
- GNPS Drug Library reference MS/MS spectra of drugs, metabolites, and analogs with standardized machine-readable pharmacologic metadata (exposure sources, pharmacologic classes, therapeutic indications)

## Outputs

- drug_annotations.csv — per-sample drug annotation table with compound, pharmacologic class, and exposure source
- Per-compound detection counts aggregated by sample collection month (drug-by-month seasonality table)

## How to apply

Download the time-series mzML/mzXML files (e.g. wastewater influent from MassIVE accession MSV000097575, n=55 samples collected March-June 2020) and match sample MS/MS spectra against the GNPS Drug Library reference spectra via spectral similarity search using cosine score with mass-bmatch tolerance per GNPS library search conventions. Aggregate the per-sample annotations into a drug-by-sample table listing each annotated compound with its pharmacologic class and exposure source, using the library's machine-readable pharmacologic metadata. Then group per-compound detection counts by sample collection month to build a drug-by-month detection table; compounds whose reference spectra include metabolites/analogs should be counted on their annotated form so metabolized exposures are not missed. Judge success by reproducing the expected seasonality signature — a decline in dextromethorphan and antibiotic detections from March to June 2020 — directly from the aggregated per-sample annotation table saved as drug_annotations.csv.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Provides the Drug Library MS/MS reference spectra and library-search spectral matching (cosine score) used to annotate drug exposures per sample before month-level aggregation.)
- **microbe_masst (domainMASSTs)** (Batch MS/MS search of annotated spectra against indexed public metabolomics data via the Fast Search API (jobs.py), with a _matches.tsv output listing scans matching a searched spectrum — usable for retrospective re-analysis of archived datasets.) — https://github.com/robinschmid/microbe_masst

## Examples

```
python code/jobs.py  # files list: ("input_dir/wastewater_influent.mgf", "out/msv000097575_wastewater"), then aggregate _matches.tsv counts by collection month
```

## Evaluation signals

- Seasonality trend reproduced: per-compound monthly detection counts show the reported March-to-June 2020 decline for dextromethorphan and antibiotic detections in the wastewater influent data.
- drug_annotations.csv exists with per-sample rows annotated with pharmacologic class and exposure source from the GNPS Drug Library metadata.
- Every drug annotation traceable to a spectral match meeting GNPS library-search conventions (cosine score with mass-bmatch tolerance).
- Sample count reconciles with the source accession (n=55 wastewater influent samples, March-June 2020).
- Control check: the same aggregated table should reproduce the expected food-category detections (ampicillin/tetracycline in animal products, spinosad/thiabendazole in produce), confirming the annotation-to-aggregation pipeline is not biased toward the wastewater time series.

## Limitations

- Annotations depend on reference spectra; existing MS/MS libraries capture only the parent drug due to challenges in sourcing drug metabolite reference standards, so metabolized-only exposures can be missed.
- Conventional and empirical readouts can overlook over-the-counter medications, dietary supplements, non-adherence, undocumented drug use, and unrealized exposures via the food supply.
- The GNPS Drug Library is not intended for immediate clinical decision-making; detections support data-driven exploration rather than clinical use.
- The resource evolves via community deposition of reference libraries, so detection counts change as the library and indexed public datasets (GNPS/MassIVE, Metabolomics Workbench, MetaboLights, NORMAN) grow; batch Fast Search API runs may need to be re-run to catch all matches.

## Evidence

- [other] compute per-compound detection counts by collection month to quantify the March-to-June 2020 decline of dextromethorphan and antibiotic detections: [evidence span withheld — anchor status: not_found]
- [other] MassIVE accession MSV000097575 (n=55 samples collected March-June 2020): [evidence span withheld — anchor status: not_found]
- [other] Match sample MS/MS spectra against the GNPS Drug Library reference spectra via spectral similarity search (cosine score with mass-bmatch tolerance per GNPS library search conventions): [evidence span withheld — anchor status: not_found]
- [readme] A _matches.tsv file will be generated. This contains all the scans found to match your searched spectrum of interest in the data that have been currently indexed.: [evidence span withheld — anchor status: external_doc]
- [abstract] While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable.: "While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable."
