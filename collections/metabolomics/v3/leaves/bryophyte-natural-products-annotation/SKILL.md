---
name: bryophyte-natural-products-annotation
description: Use when a MAW run's final merged candidates (file_id_mergedResults-with-one-Candidates.csv)
  need chemical-class profiling — i.e. subclass abundances of classified annotations
  and which subclasses dominate (e.g. Lipids, Organic acids, Benzenoids) — for a metabolomics
  dataset.
when_to_use_negative:
- The task is candidate ranking or annotation quality evaluation (e.g. against a standards
  dataset) rather than class-distribution summarization of already-classified features.
- Only raw MS2 mzML spectra are available and the MAW workflow (spectral/compound
  dereplication + classification) has not yet been run — CANOPUS/ClassyFire assignments
  do not exist yet.
- The annotation source is manual literature annotation rather than MAW's computational
  CANOPUS/ClassyFire classification, so no ChemOnt hierarchy mappings exist.
tools:
- name: CANOPUS
  role: Computational classification tool predicting chemical classes of tentative
    candidates in MAW's R segment; its assignments feed the ChemOnt hierarchy aggregation
- name: ClassyFire
  role: Classification framework whose ChemOnt hierarchy (superclass/class/subclass)
    is used to organize and aggregate the class assignments
- name: MAW-R
  role: R segment of MAW that performs spectral/compound database dereplication with
    SIRIUS and the computational classification step
  repo: https://github.com/zmahnoor14/MAW
- name: MAW-Py
  role: Python segment performing final candidate selection (RDKit), producing the
    per-feature top-ranking candidates this skill summarizes
  repo: https://github.com/zmahnoor14/MAW
- name: Spectra
  role: R package used by MAW for spectral database dereplication (HMDB, MassBank,
    GNPS) upstream of candidate classification
  repo: https://github.com/zmahnoor14/MAW
- name: zmahnoor/maw-r:1.0.8
  role: Docker image in which the reproducible MAW R workflow, including classification,
    is executed
  repo: https://github.com/zmahnoor14/MAW
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-023-00695-y
    title: 'MAW: the reproducible Metabolome Annotation Workflow for untargeted tandem
      mass spectrometry'
  source_content_hash: sha256:7796721b984323ba136241f73c51767fd7c8da93ae81062a058fe41217c9f779
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zmahnoor14/MAW
  tools:
  - CANOPUS
  - ClassyFire
  - MAW-Py
  - MAW-R
  - Spectra
  - zmahnoor/maw-r:1.0.8
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bryophyte-natural-products-annotation

## Summary

Summarizes the chemical class assignments of MAW's top-ranking annotations (e.g. the bryophytes LC-MS2 dataset) by mapping CANOPUS/ClassyFire predictions onto the ClassyFire ChemOnt hierarchy and reporting superclass/class/subclass abundances, visualized as a sunburst (Fig. 4).

## When to use

Use when a MAW run's final merged candidates (file_id_mergedResults-with-one-Candidates.csv) need chemical-class profiling — i.e. subclass abundances of classified annotations and which subclasses dominate (e.g. Lipids, Organic acids, Benzenoids) — for a metabolomics dataset.

## When NOT to use

- The task is candidate ranking or annotation quality evaluation (e.g. against a standards dataset) rather than class-distribution summarization of already-classified features.
- Only raw MS2 mzML spectra are available and the MAW workflow (spectral/compound dereplication + classification) has not yet been run — CANOPUS/ClassyFire assignments do not exist yet.
- The annotation source is manual literature annotation rather than MAW's computational CANOPUS/ClassyFire classification, so no ChemOnt hierarchy mappings exist.

## Inputs

- MAW final merged candidates results for the dataset (file_id_mergedResults-with-one-Candidates.csv), one top-ranking candidate per MS2 feature
- CANOPUS/ClassyFire chemical class predictions per classified feature
- MAW containerized workflow (zmahnoor/maw-r:1.0.8) for reproducing the annotation + classification run

## Outputs

- subclass_abundances.csv — subclass counts and percentages of annotated features
- class_sunburst.png — sunburst of superclass/class/subclass hierarchy with subclass segments sized by abundance (article Fig. 4)

## How to apply

Start from MAW's final merged candidates output for the dataset of interest and keep only the top-ranking annotation per feature (MAW prioritizes a final candidate from the candidate list for each feature). Extract the chemical class assignments produced by MAW's computational classification step, which runs in the R segment via CANOPUS alongside SIRIUS/CSI:FingerID dereplication. Map each assignment onto the ClassyFire ChemOnt hierarchy and aggregate counts at superclass, class, and subclass levels. Compute subclass abundances as percentages of annotated features (write subclass_abundances.csv) and render the superclass/class/subclass hierarchy as a sunburst chart (class_sunburst.png) with subclass segments sized by abundance. Note the article's caveats: lipid-class annotation via SIRIUS/CANOPUS can yield false positives because lipid isomers co-elute, and many bryophyte natural product classes are hard to acquire with tandem MS, so unclassified/unannotated features are expected and abundances are percentages of annotated features only.

## Related tools

- **CANOPUS** (Computational classification tool predicting chemical classes of tentative candidates in MAW's R segment; its assignments feed the ChemOnt hierarchy aggregation)
- **ClassyFire** (Classification framework whose ChemOnt hierarchy (superclass/class/subclass) is used to organize and aggregate the class assignments)
- **MAW-R** (R segment of MAW that performs spectral/compound database dereplication with SIRIUS and the computational classification step) — https://github.com/zmahnoor14/MAW
- **MAW-Py** (Python segment performing final candidate selection (RDKit), producing the per-feature top-ranking candidates this skill summarizes) — https://github.com/zmahnoor14/MAW
- **Spectra** (R package used by MAW for spectral database dereplication (HMDB, MassBank, GNPS) upstream of candidate classification) — https://github.com/zmahnoor14/MAW
- **zmahnoor/maw-r:1.0.8** (Docker image in which the reproducible MAW R workflow, including classification, is executed) — https://github.com/zmahnoor14/MAW

## Examples

```
docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64
```

## Evaluation signals

- Reproduced subclass abundances match the reported values — Lipids 33.92%, Organic acids 17.02%, Benzenoids 13.94%, Phenylpropanoid and polyketides 11.34% — within rounding tolerance.
- All four dominant subclass labels (Lipids, Organic acids, Benzenoids, Phenylpropanoid and polyketides) appear in subclass_abundances.csv.
- Every aggregated assignment maps onto a valid ChemOnt superclass/class/subclass node (no unmapped class strings).
- Abundances sum to percentages of annotated features only; features without top-ranking classification are excluded from the denominator.
- Sunburst renders all three hierarchy levels with subclass segment angular sizes proportional to reported percentages.

## Limitations

- SIRIUS lipid-class annotation can produce false positives due to the complex nature of lipids; lipid isomers co-elute, challenging in silico lipid annotation — treat dominant Lipid percentages with caution.
- Many bryophyte natural product classes are experimentally challenging to acquire with tandem mass spectrometry, leaving features unannotated, so subclass percentages cover only annotated features.
- In MAW v1 only SIRIUS (CSI:FingerID) is integrated for compound database annotation; MetFrag is not yet integrated, so reproduced results depend on the SIRIUS-based classification path.

## Evidence

- [readme] The classification of the tentative candidates from the input data are classified using [CANOPUS](https://bio.informatik.uni-jena.de/software/canopus/) and: [evidence span withheld — anchor status: external_doc]
- [intro] workflow step list: "MAW performs (1) MS2 data preprocessing, (2) spectral database dereplication, (3) compound database dereplication, (4) candidate selection, and (5) chemical compound classification using various"
- [methods] docker image: "docker pull zmahnoor/maw-r:1.0.8 --platform=linux/amd64"
- [discussion] lipid caveat: "the annotation of lipid classes using SIRIUS can lead to false positives due to the complex nature of lipids"
