---
name: osf-deposit-data-retrieval
description: Use when a published paper states that its 'input and output data are
  available on OSF' and downstream reproduction or benchmarking requires those exact
  files (e.g. annotation outputs, benchmarking datasets) before any local computation
  can start.
when_to_use_negative:
- The analysis needs freshly acquired MS/MS data or a re-run of annotation tools (MS-Finder,
  SIRIUS) rather than the paper's deposited annotation outputs
- The target database is not the one behind the deposit — e.g. ISDB-UNPD is hosted
  separately at oolonek.github.io/ISDB and KNApSAcK is accessible online but not downloadable
- The deposit files lack taxonomic information resolved against Catalogue of Life
  down to species level, which the taxo_scorer workflow requires
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0157
tools:
- name: ISDB-DNP
  role: Annotation tool whose deposited outputs from the OSF deposit are retrieved
    as input for taxonomically informed re-ranking
  repo: https://osf.io/bvs6x/
- name: taxo_scorer
  role: Downstream R workflow (2_taxo_scorer.Rmd) that consumes the retrieved annotation
    outputs and benchmarking dataset for taxonomically informed scoring
  repo: https://github.com/oolonek/taxo_scorer
- name: tima
  role: Maintained successor repository containing everything needed to perform Taxonomically
    Informed Metabolite Annotation on the retrieved data
  repo: https://github.com/taxonomicallyinformedannotation/tima
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3389/fpls.2019.01329
    title: Taxonomically Informed Scoring Enhances Confidence in Natural Products
      Annotation
  source_content_hash: sha256:9f09666ff02e08f6208443c8a388f5a9ffd3c17fb02d02088775ab8078f54112
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ISDB-DNP
  - MS-Finder
  - OSF deposit osf.io/bvs6x
  - ParBayesianOptimization
  - taxo_scorer
  - tima
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# osf-deposit-data-retrieval

## Summary

Retrieves the published input and output data of a study from its Open Science Framework (OSF) deposit so that reported analyses can be reproduced. In this workflow, the OSF deposit osf.io/bvs6x provides the ISDB-DNP annotation outputs and the 2,107-entry benchmarking dataset needed for taxonomically informed scoring.

## When to use

Use when a published paper states that its 'input and output data are available on OSF' and downstream reproduction or benchmarking requires those exact files (e.g. annotation outputs, benchmarking datasets) before any local computation can start.

## When NOT to use

- The analysis needs freshly acquired MS/MS data or a re-run of annotation tools (MS-Finder, SIRIUS) rather than the paper's deposited annotation outputs
- The target database is not the one behind the deposit — e.g. ISDB-UNPD is hosted separately at oolonek.github.io/ISDB and KNApSAcK is accessible online but not downloadable
- The deposit files lack taxonomic information resolved against Catalogue of Life down to species level, which the taxo_scorer workflow requires

## Inputs

- OSF deposit URL (https://osf.io/bvs6x/)
- ISDB-DNP annotation output files deposited on OSF
- 2,107-entry benchmarking dataset deposited on OSF
- Problematic_entries.csv (list of benchmarking entries with > 0.01 Da mass differences)

## Outputs

- Locally cached copies of the ISDB-DNP annotation outputs and benchmarking dataset ready for the taxo_scorer (2_taxo_scorer.Rmd) workflow
- Recorded deposit identifier/version associated with downstream results

## How to apply

Locate the OSF deposit URL cited in the article (here https://osf.io/bvs6x/) and download the deposited files relevant to the target analysis — for this paper, the ISDB-DNP annotation outputs and the 2,107-entry benchmarking dataset. Check file provenance against the article's description: the benchmarking dataset contains 2,107 distinct entries, DNP v 27.1 taxonomic fields resolved against Catalogue of Life, and a companion Problematic_entries.csv listing 197 entries with large mass differences (> 0.01 Da) that should be excluded or flagged downstream. Feed the retrieved annotation outputs into the taxo_scorer workflow (2_taxo_scorer.Rmd) and treat any averaged or optimized results as reproducible only if they match values reported in the paper (e.g. optimal taxo-informed scores 0.81, 1.62, 2.55). Record the deposit version/DOI in outputs so results remain traceable.

## Related tools

- **ISDB-DNP** (Annotation tool whose deposited outputs from the OSF deposit are retrieved as input for taxonomically informed re-ranking) — https://osf.io/bvs6x/
- **taxo_scorer** (Downstream R workflow (2_taxo_scorer.Rmd) that consumes the retrieved annotation outputs and benchmarking dataset for taxonomically informed scoring) — https://github.com/oolonek/taxo_scorer
- **tima** (Maintained successor repository containing everything needed to perform Taxonomically Informed Metabolite Annotation on the retrieved data) — https://github.com/taxonomicallyinformedannotation/tima

## Examples

```
wget -r -np https://osf.io/bvs6x/ && Rscript -e "read.csv('benchmarking_dataset.csv'); nrow(.)==2107"
```

## Evaluation signals

- Downloaded benchmarking dataset contains 2,107 distinct entries, matching the article's stated benchmarking dataset size
- Retrieved annotation outputs correspond to one of the evaluated tools (ISDB-DNP, Sirius, MS-Finder) described in the deposit
- Downstream taxo_scorer re-ranking on the retrieved data reproduces the reported optimal taxo-informed scores of 0.81 (family), 1.62 (genus), 2.55 (species)
- Problematic_entries.csv is present and the 197 entries with mass differences > 0.01 Da can be cross-referenced against the downloaded benchmarking dataset
- Deposit DOI/version is recorded alongside any results derived from the downloaded files

## Limitations

- All compounds of the benchmarking dataset are present in the DNP, so ISDB-DNP is favored in any benchmarking comparison performed on the deposited data
- The GNPS spectral libraries were also part of the Sirius training set, confounding tool comparisons using these files
- 197 benchmarking entries have large mass differences (> 0.01 Da) between experimental parent ion mass and calculated exact mass; erroneous entries cannot be identified by annotation tools
- The optimized score combination (0.81/1.62/2.55) is heavily dependent on this training set and should not be taken as absolute optimal values

## Evidence

- [discussion] OSF input files: "Input and output data are available on OSF at the following address (https://osf.io/bvs6x/)"
- [discussion] benchmarking dataset size: "constituted by 2107 distinct entries and was named benchmarking dataset"
- [discussion] problematic entries: "a number of entries (197) of the benchmarking dataset were found to have large mass difference (> 0.01 Da)"
- [discussion] optimization caveat: "Such results should be taken with caution, and not as absolute optimal values, as such optimization process are heavily dependent on the training sets"
- [discussion] taxo_scorer repo: "Scripts are available at https://github.com/oolonek/taxo_scorer"
- [other] deposit usage in task workflow: [evidence span withheld — anchor status: not_found]
