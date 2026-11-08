---
name: mass-offset-curation-filtering
description: 'Use when propagating GNPS Drug Library MS/MS references into an analog
  library: you have raw drug-analog matches from public metabolomics spectra and must
  remove matches whose mass offset is not among the 156 curated, chemically explained
  biotransformations.'
when_to_use_negative:
- The spectrum already receives a full library match in the GNPS library — it is a
  direct spectral hit of the parent drug/metabolite, not a propagated analog, and
  belongs to the library-match filter instead
- The analysis targets parent-drug screening only, where the GNPS Drug Library direct
  MS/MS library search suffices without analog propagation
- The goal is immediate clinical decision-making — the GNPS Drug Library is explicitly
  not intended for that purpose
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Provides the Drug Library MS/MS reference spectra, analog matching against
    public metabolomics data, and spectral clustering used by this filter
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:6858695efd05068230f1385d24f00517675de7b0d1ca1c444de910c9468f786c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# mass-offset-curation-filtering

## Summary

A curation filter that retains only drug-analog spectral matches whose parent-drug-to-analog mass offsets belong to a curated set of 156 chemically explained biotransformations, separating plausible propagated analogs from spurious GNPS Drug Library matches in untargeted metabolomics data.

## When to use

Use when propagating GNPS Drug Library MS/MS references into an analog library: you have raw drug-analog matches from public metabolomics spectra and must remove matches whose mass offset is not among the 156 curated, chemically explained biotransformations.

## When NOT to use

- The spectrum already receives a full library match in the GNPS library — it is a direct spectral hit of the parent drug/metabolite, not a propagated analog, and belongs to the library-match filter instead
- The analysis targets parent-drug screening only, where the GNPS Drug Library direct MS/MS library search suffices without analog propagation
- The goal is immediate clinical decision-making — the GNPS Drug Library is explicitly not intended for that purpose

## Inputs

- Raw drug-analog match table from GNPS Drug Library searches against public metabolomics MS/MS data
- Curated list of 156 chemically explained mass offsets (biotransformations)
- Curated drug metadata: exposure-source annotations (endogenous/dietary flags) and pharmacologic classes

## Outputs

- Filtered propagated analog library table (analog_library_filtered.csv) with 12,455 clustered non-duplicated MS/MS spectra covering analogs of 1277 drugs

## How to apply

Load the raw drug-analog match table produced by searching GNPS Drug Library spectra against public metabolomics data, together with curated metadata on exposure sources and pharmacologic classes. As one step within the four-step propagated-analog curation pipeline, first exclude analogs whose parent drug carries an endogenous or dietary-source exposure annotation, then compute the parent-drug-to-analog mass offset and drop any match whose offset is not in the curated list of 156 chemically explained mass offsets (modifications such as glucuronidation, sulfation, oxidation, etc.). This mass-offset filter sits upstream of the library-match filter (remove analogs already receiving a full GNPS library hit, since those are direct spectral matches rather than propagated analogs) and the dataset-testing filter (remove analogs over-detected when searched against 20 public test metabolomics datasets). Rationale: existing MS/MS reference libraries capture only parent drugs, so analog propagation must be constrained to chemically defensible mass shifts to avoid propagating false positives. After all four filters, propagate surviving spectra into the final clustered analog library.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Provides the Drug Library MS/MS reference spectra, analog matching against public metabolomics data, and spectral clustering used by this filter) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library

## Examples

```
filtered = df[df.mass_offset.isin(curated_offsets_156)]; filtered.to_csv('analog_library_filtered.csv', index=False)
```

## Evaluation signals

- Final filtered library contains exactly 12,455 clustered non-duplicated MS/MS spectra, as reported in the paper's results
- Surviving analogs span analogs of 1277 drugs
- No retained analog has a parent-drug-to-analog mass offset outside the 156 curated, chemically explained offsets
- No retained analog corresponds to a parent drug with an endogenous or dietary-source exposure annotation
- No retained analog spectrum is already a full GNPS library match, and none is over-detected across the 20 public test metabolomics datasets

## Limitations

- Only 156 mass offsets are considered chemically explained; analogs with uncurated biotransformations are systematically excluded and thus missed
- Analogs over-detected when searched against 20 public test datasets must be removed as likely non-exposure artifacts, risking loss of genuinely ubiquitous drugs
- The resource is not intended for immediate clinical decision-making

## Evidence

- [results] Four-filter curation workflow: [evidence span withheld — anchor status: not_found]
- [results] Final library size: [evidence span withheld — anchor status: not_found]
- [intro] Motivation for analog propagation: "it is common that only metabolized versions of a drug are present in the sample, leading to missed drug exposure if only the parent drug is considered24"
- [intro] Resource curation goal: "In this work, we curate the Global Natural Product Social Molecular Networking (GNPS) Drug Library to enable data science strategies on drug readouts."
