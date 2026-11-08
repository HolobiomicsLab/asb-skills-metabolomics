---
name: metabolomics-workbench-data-acquisition
description: Use when a downstream analysis (network construction, partial-correlation
  estimation, differential network analysis) requires a real metabolomics dataset,
  and publicly available study data such as the TEDDY IA/T1D metabolomics measurements
  can be pulled from the Metabolomics Workbench.
when_to_use_negative:
- The analysis needs a small, curated example dataset — the DNEA R package already
  ships example data accessible via its vignette (browseVignettes("DNEA")), so no
  Workbench download is needed
- The target metabolites/phenotypes are not represented in publicly available Workbench
  studies (e.g. data under restricted access or from a different cohort)
- The downstream tool requires normalized or log-transformed input and a different
  preprocessing protocol than the >30%-missing filter with median imputation
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: DNEA R package
  role: Consumes the downloaded, preprocessed TEDDY peak-intensity data for joint
    network estimation; provides example data and a vignette if a public dataset download
    is not required
  repo: http://www.github.com/Karnovsky-Lab/DNEA/
- name: Metabox2
  role: Downstream consumer of the acquired IA-arm metabolomics data for building
    the marginal (Pearson) correlation network used in the density comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-024-05994-1
    title: 'DNEA: an R package for fast and versatile data-driven network analysis
      of metabolomics data'
  source_content_hash: sha256:b31d576e44a0d4c7eee8681f2bf443c0223ae699621985c5d3cbe88bb20278ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNEA R package
  - Metabox2
  - R
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolomics-workbench-data-acquisition

## Summary

Retrieve publicly available metabolomics peak-intensity datasets (e.g. TEDDY study data via Metabolomics Workbench accession ST001386/ST001636, project PR000950) and preprocess them—excluding metabolites missing in >30% of samples and median-imputing the rest—so they are suitable for network construction with tools like DNEA or MetaBox2.

## When to use

Use when a downstream analysis (network construction, partial-correlation estimation, differential network analysis) requires a real metabolomics dataset, and publicly available study data such as the TEDDY IA/T1D metabolomics measurements can be pulled from the Metabolomics Workbench.

## When NOT to use

- The analysis needs a small, curated example dataset — the DNEA R package already ships example data accessible via its vignette (browseVignettes("DNEA")), so no Workbench download is needed
- The target metabolites/phenotypes are not represented in publicly available Workbench studies (e.g. data under restricted access or from a different cohort)
- The downstream tool requires normalized or log-transformed input and a different preprocessing protocol than the >30%-missing filter with median imputation

## Inputs

- Metabolomics Workbench study accession (e.g. ST001386, ST001636; TEDDY project PR000950)
- Unnormalized metabolomics peak-intensity table (samples x metabolites)
- Sample metadata / experimental-group labels (e.g. IA arm vs T1D arm; age and sex covariates)

## Outputs

- Preprocessed metabolomics feature table with metabolites missing in >30% of samples removed and remaining missing values median-imputed
- Group-specific dataset subsets (e.g. IA-arm or T1D-arm peak-intensity matrices) ready for DNEA or MetaBox2 network construction

## How to apply

Identify the Metabolomics Workbench study accession for the target data (the DNEA paper used ST001386 and ST001636, containing 11,560 samples from 1818 TEDDY subjects, split into a 3525-sample T1D arm and a 9024-sample IA arm). Download the unnormalized peak-intensity matrix (optionally adjusted for covariates such as age and sex, as done for the TEDDY analysis) and select the experimental group(s) of interest, e.g. the IA arm used in the MetaBox2 comparison. Apply the paper's preprocessing filters: exclude metabolites missing in more than 30% of samples, then impute all remaining missing values with the median value of each metabolite. Confirm sample/group counts against the published dataset description before passing the processed feature table to network-estimation steps such as BICtune, stabilitySelection, and getNetworks in DNEA, or to MetaBox2 correlation-network construction.

## Related tools

- **DNEA R package** (Consumes the downloaded, preprocessed TEDDY peak-intensity data for joint network estimation; provides example data and a vignette if a public dataset download is not required) — http://www.github.com/Karnovsky-Lab/DNEA/
- **Metabox2** (Downstream consumer of the acquired IA-arm metabolomics data for building the marginal (Pearson) correlation network used in the density comparison)

## Examples

```
BiocManager::install("DNEA"); browseVignettes("DNEA")  # then load TEDDY peak intensities downloaded from Metabolomics Workbench ST001386/ST001636
```

## Evaluation signals

- Dataset dimensions match the published description: 11,560 samples from 1818 subjects, with 3525 samples in the T1D arm and 9024 samples in the IA arm
- No metabolite in the processed table is missing in more than 30% of samples
- All remaining missing values in the feature table are replaced by the per-metabolite median (no NAs remain)
- Group assignment is correct — e.g. the IA subset contains the 9024 IA-arm samples; 414 subjects tested positive for islet auto-antibodies and 50 developed T1D
- Unnormalized peak-intensity values (optionally adjusted for age and sex) are used as input, matching the paper's stated input convention

## Limitations

- Preprocessing choices (30% missingness cutoff, median imputation) are the paper's protocol; other pipelines may require different filtering or normalization
- Unnormalized peak intensities are used directly; scaling/batch-correction decisions are left to the downstream workflow
- Only publicly available Workbench studies (e.g. TEDDY ST001386/ST001636) can be acquired this way; the paper provides no software version numbers for the tools applied to these data

## Evidence

- [methods] The metabolomics data used in our analyses were obtained from the Metabolomics Workbench (ST001386 and ST001636).: "The metabolomics data used in our analyses were obtained from the Metabolomics Workbench (ST001386 and ST001636)."
- [results] The Metabolomics Workbench [26] contained 11,560 samples from 1818 subjects: "The Metabolomics data obtained from the Metabolomics Workbench [26] contained 11,560 samples from 1818 subjects"
- [methods] The data were filtered to exclude metabolites that were missing in more than 30% of the samples. For the remaining metabolites, all missing values were imputed with the median value of each metabolite.: "The data were filtered to exclude metabolites that were missing in more than 30% of the samples."
- [results] unnormalized peak intensity values adjusted for age and sex were used as input. DNEA analysis was performed on IA and T1D datasets using all samples: "unnormalized peak intensity values adjusted for age and sex were used as input. DNEA analysis was performed on IA and T1D datasets using all samples"
