---
name: cross-dataset-peak-intensity-correlation
description: Use when reanalyzing publicly deposited MRM mzML raw data to verify that
  an automatic peak-integration pipeline reproduces previously reported, manually
  reviewed peak intensities on a per-metabolite basis.
when_to_use_negative:
- Samples are excessively dirty, very-low-input, or recorded with a new LC method
  — the article states one should retreat to manual peak review for selected metabolites
  in these cases
- The metabolite database has no qualifier transitions — automRm requires at least
  one qualifier in addition to the quantifier for optimal performance
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: automRm
  role: R package performing ML-based peak picking, RT alignment, prototype chromatogram
    building, and peak reporting on MRM batches
  repo: https://gitlab.gwdg.de/joerg.buescher/automrm
- name: R
  role: Runtime environment (developed/tested in R 3.6.0 and 4.0.2)
- name: devtools
  role: Installs automRm from the GitLab host via devtools::install_gitlab
- name: mzR
  role: Parses mzML raw files during process_batch()
- name: openxlsx
  role: Writes peak intensities to peakinfo.xlsx
- name: caret
  role: Machine-learning backend for the random-forest peak picking/reporting models
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c05224
    title: 'automRm: An R Package for Fully Automatic LC-QQQ-MS Data Preprocessing
      Powered by Machine Learning'
  source_content_hash: sha256:bd7e56978ff18e3d79848ab3253c3d934b70b62d016f8d761b0f1bb7e82e607f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - automRm
  - caret
  - devtools
  - mzR
  - openxlsx
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cross-dataset-peak-intensity-correlation

## Summary

Validates automatic LC-QQQ-MS (MRM) peak preprocessing by rerunning automRm on public MetaboLights data sets (MTBLS429, MTBLS897) and correlating its peak areas against the originally published, manually reviewed signal intensities per metabolite.

## When to use

Use when reanalyzing publicly deposited MRM mzML raw data to verify that an automatic peak-integration pipeline reproduces previously reported, manually reviewed peak intensities on a per-metabolite basis.

## When NOT to use

- Samples are excessively dirty, very-low-input, or recorded with a new LC method — the article states one should retreat to manual peak review for selected metabolites in these cases
- The metabolite database has no qualifier transitions — automRm requires at least one qualifier in addition to the quantifier for optimal performance

## Inputs

- Public MetaboLights MRM raw data in mzML format (accessions MTBLS429 and MTBLS897)
- Metabolite settings / metabolite database (metabdb.xlsx) listing quantifier and qualifier MRM transitions with expected retention times
- Pretrained combined random-forest peak picking and peak reporting models
- update_prm.tsv batch configuration pointing to raw data, models, and metabolite database
- Originally published, manually reviewed peak intensities for correlation

## Outputs

- peakinfo.xlsx peak-intensity tables per data set
- Per-metabolite Pearson correlations between automRm peak areas and originally reported signal intensities
- Accepted/rejected peak calls from the peak-reporting random-forest model

## How to apply

Download the MRM raw data (mzML) and metabolite settings for MTBLS429 (pyrrolizidine alkaloids) and MTBLS897 (grape secondary metabolites) from MetaboLights, then install automRm via devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de'). Configure update_prm.tsv to point to the raw data, the pretrained combined random-forest peak picking/reporting models, and a metabolite database (metabdb.xlsx) listing quantifier/qualifier transitions and expected retention times, ensuring each analyte has at least one qualifier. Run process_batch() on each accession: mzR parses the mzML files, local maxima are ranked on smoothed quantifier traces (up to 5 candidates), RT shifts are estimated per metabolite by cross-correlation (fitted as a quadratic function of RT), RT-shifted chromatograms are summed into a prototype whose boundaries propagate back to each sample, and the peak-reporting random forest applies 20 quality scores plus the peak-picking score and its 0th/25th/50th/75th/100th percentiles across samples. Finally, write intensities to peakinfo.xlsx and compute the Pearson correlation between automRm peak areas and the originally reported intensities for each metabolite in each data set.

## Related tools

- **automRm** (R package performing ML-based peak picking, RT alignment, prototype chromatogram building, and peak reporting on MRM batches) — https://gitlab.gwdg.de/joerg.buescher/automrm
- **R** (Runtime environment (developed/tested in R 3.6.0 and 4.0.2))
- **devtools** (Installs automRm from the GitLab host via devtools::install_gitlab)
- **mzR** (Parses mzML raw files during process_batch())
- **openxlsx** (Writes peak intensities to peakinfo.xlsx)
- **caret** (Machine-learning backend for the random-forest peak picking/reporting models)

## Examples

```
devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de'); automRm::process_batch()
```

## Evaluation signals

- Pearson correlation > 0.95 between automRm peak areas and originally reported intensities for the majority of metabolites (paper: 22/34 metabolites in MTBLS429 and 20/30 in MTBLS897)
- Both public accessions (MTBLS429, MTBLS897) complete process_batch() and yield peakinfo.xlsx output files
- Reported peaks include signal from the qualifier transition in addition to the quantifier, as required by the metabdb.xlsx configuration
- Peak calls show low variation in integration among replicate measurements, consistent with automRm outperforming alternative software on this criterion

## Limitations

- automRm requires at least one qualifier in addition to the quantifier, which itself reduces the number of analytes detectable simultaneously in MRM mode
- Reaching quality similar to manual peak review requires sufficient training on relevant raw data
- Manual peak review is still needed for selected metabolites in special cases such as excessively dirty samples, very-low-input samples, or new LC methods

## Evidence

- [other] success requires correlation >0.95 for the majority of metabolites (the paper reports >0.95 for 22/34 metabolites in MTBLS429 and 20/30 in MTBLS897): [evidence span withheld — anchor status: not_found]
- [abstract] Next, we show that automRm can replicate results obtained by manual peak review on published data.: "automRm can replicate results obtained by manual peak review on published data"
- [methods] install automRm in R via devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de'): "devtools::install_gitlab('joerg.buescher/automRm@master', host = 'https://gitlab.gwdg.de')"
- [methods] automRm depends on several packages such as mzR7 for parsing of mzML files: "automRm depends on several packages such as mzR7 for parsing of mzML files"
- [discussion] For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier.: "For optimal performance, automRm requires the use of at least one qualifier in addition to the quantifier."
