---
name: metabolomics-feature-annotation-against-msms-library
description: Use when a feature table or MGF of MS/MS spectra from an untargeted metabolomics
  experiment needs compound-level annotations against a reference library (e.g., GNPS
  Drug Library) to enable drug-exposure readouts or lipid-class testing.
when_to_use_negative:
- The input is already an annotated feature table or the compound identities are known
  — annotation would duplicate existing work
- 'The goal is clinical decision-making: the article states the GNPS Drug Library
  is ''not intended for immediate clinical decision-making'' but for data-driven exploration
  of drug exposures'
- 'Only parent-drug-level screening is needed and metabolized versions are expected:
  note the article''s caveat that existing libraries historically ''capture only the
  parent drug'', which can miss drug exposure when only metabolites are present'
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Provides the GNPS Drug Library of MS/MS reference spectra for drugs and their
    metabolites/analogs used to annotate spectra and derive empirical drug-exposure
    readouts
- name: robinschmid/microbe_masst
  role: Contains jobs.py code leveraging the GNPS Fast Search API to batch-search
    multiple MS/MS spectra against indexed GNPS/MassIVE libraries and generate match/library/dataset
    output files
  repo: https://github.com/robinschmid/microbe_masst
- name: ninahaoqizhao/Manuscript_GNPS_Drug_Library
  role: Manuscript repository accompanying the GNPS Drug Library resource
  repo: https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:b9e1fc24357d8b782d04d66bc0ea75b718b1a71c9cc25c537bf4512c066d273d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/robinschmid/microbe_masst
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  - ninahaoqizhao/Manuscript_GNPS_Drug_Library
  - robinschmid/microbe_masst
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabolomics-feature-annotation-against-msms-library

## Summary

Annotate untargeted metabolomics MS/MS spectra against curated reference spectral libraries (e.g., the GNPS Drug Library, a tandem MS/MS resource for drug screening) to assign putative compound identities — such as antiretroviral drugs and N-acyl lipids — before downstream exposure or association analyses.

## When to use

Use when a feature table or MGF of MS/MS spectra from an untargeted metabolomics experiment needs compound-level annotations against a reference library (e.g., GNPS Drug Library) to enable drug-exposure readouts or lipid-class testing.

## When NOT to use

- The input is already an annotated feature table or the compound identities are known — annotation would duplicate existing work
- The goal is clinical decision-making: the article states the GNPS Drug Library is 'not intended for immediate clinical decision-making' but for data-driven exploration of drug exposures
- Only parent-drug-level screening is needed and metabolized versions are expected: note the article's caveat that existing libraries historically 'capture only the parent drug', which can miss drug exposure when only metabolites are present

## Inputs

- Untargeted metabolomics MS/MS spectra in MGF format (e.g., generated via MZmine molecular networking)
- List of USIs (.csv or .tsv) referencing spectra of interest
- Reference MS/MS library such as the GNPS Drug Library (drugs plus metabolites/analogs with pharmacologic metadata)
- Targeted class-specific reference library (e.g., 851-compound N-acyl lipid reference library)

## Outputs

- Library annotations per spectrum (e.g., _library.tsv listing matched GNPS library spectra, enabling Level 2 Metabolomics Standards Initiative annotations)
- Match tables (_matches.tsv, _datasets.tsv, _count_domain.tsv) with scans, matched library entries, and per-dataset sample match counts
- Per-sample drug-associated signal summary, e.g., a sample-by-ARV-exposure binary matrix and per-lipid abundance/test-statistic table (nacyl_lipid_arv_analysis.csv)

## How to apply

Match each acquired MS/MS spectrum against a curated reference library; for drug-exposure screening use the GNPS Drug Library, which integrates MS/MS references of drugs, their metabolites, and analogs with machine-readable pharmacologic metadata. Batch searches can be run via the GNPS Fast Search API, supplying a single .mgf file generated via MZmine or a list of USIs (.csv/.tsv), and adjusting minimum cosine score, m/z tolerance, and minimum matching peaks to the research question. Supplement the drug library with a targeted reference library for the compound class of interest (the HNRC analysis used an 851-compound N-acyl lipid reference library). Because Fast Search API entries can fail, re-run batch jobs until no new output is generated (skip_existing=True) so all library matches are captured. The resulting per-sample drug-associated signals are then summarized into a binary sample-by-exposure matrix for clustering and statistical testing (e.g., Kruskal-Wallis across exposure groups, significance at p<0.05).

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Provides the GNPS Drug Library of MS/MS reference spectra for drugs and their metabolites/analogs used to annotate spectra and derive empirical drug-exposure readouts)
- **robinschmid/microbe_masst** (Contains jobs.py code leveraging the GNPS Fast Search API to batch-search multiple MS/MS spectra against indexed GNPS/MassIVE libraries and generate match/library/dataset output files) — https://github.com/robinschmid/microbe_masst
- **ninahaoqizhao/Manuscript_GNPS_Drug_Library** (Manuscript repository accompanying the GNPS Drug Library resource) — https://github.com/ninahaoqizhao/Manuscript_GNPS_Drug_Library

## Examples

```
python code/jobs.py  # files list: ("input_directory/input_file.mgf", "output_directory/output_prefix"); re-run until skip_existing=True produces no new output
```

## Evaluation signals

- A _library.tsv output is produced listing GNPS library spectra matched to searched scans, supporting Level 2 Metabolomics Standards Initiative annotations
- Matches appear in _matches.tsv with corresponding scan numbers, and _datasets.tsv reports non-zero counts of unique matching samples per dataset
- Batch jobs are re-run until no new output is generated (skip_existing=True), indicating no Fast Search API failures remain
- Annotation-derived exposure groups yield group-discriminating signals: in the HNRC case, nine N-acyl lipids including histamine-C2:0 differed significantly (Kruskal-Wallis, p<0.05) across four exposure groups, whereas self-reported medication metadata showed no differences

## Limitations

- Existing MS/MS reference libraries capture only the parent drug (with very few exceptions) due to difficulty sourcing drug metabolite reference standards, leaving a gap in drug screening capacity when only metabolized versions are present
- The GNPS Drug Library is not intended for immediate clinical decision-making; it supports research-level, data-driven exploration of drug exposures
- Batch Fast Search API runs can fail on individual entries, so jobs must be re-run (skip_existing=True) and Python 3.10 is required

## Evidence

- [abstract] the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics: "the Global Natural Product Social Molecular Networking (GNPS) Drug Library, a tandem mass spectrometry (MS/MS) based resource designed for drug screening with untargeted metabolomics"
- [intro] In this work, we curate the Global Natural Product Social Molecular Networking (GNPS) Drug Library to enable data science strategies on drug readouts.: "In this work, we curate the Global Natural Product Social Molecular Networking (GNPS) Drug Library to enable data science strategies on drug readouts."
- [readme] A _library.tsv file will be generated. This contains a list of spectra from the GNPS libraries found to match your spectrum of interest.: [evidence span withheld — anchor status: not_found]
- [readme] You can run either a single .mgf file generated via MZmine, from the molecular networking in GNPS workflow, or a list of USIs provided either via a .csv or .tsv file.: [evidence span withheld — anchor status: not_found]
- [readme] Check and adjust the different parameters for the search, such as minimum cosine score, mz tolerance, and number of minimum matching peaks based on your research question.: [evidence span withheld — anchor status: external_doc]
