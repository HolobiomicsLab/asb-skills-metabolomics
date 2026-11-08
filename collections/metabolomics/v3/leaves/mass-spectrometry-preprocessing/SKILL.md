---
name: mass-spectrometry-preprocessing
description: Use when raw MS/MS data from complex mixtures (e.g., beer or urine extracts
  on a Q-Exactive Orbitrap) must be converted into fragment/neutral-loss documents
  before unsupervised substructure discovery with LDA.
when_to_use_negative:
- The analysis goal is classical spectral similarity clustering or targeted compound
  identification (e.g., MS2Analyzer-style rule-based matching) rather than unsupervised
  substructure discovery
- Input is already a preprocessed feature table or bag-of-fragments document — rerunning
  extraction would be redundant
- Data are information-poor MS2 spectra (e.g., fixed low collision energy on simple
  mixtures), where MS2LDA preprocessing adds little since motifs require rich fragment/neutral-loss
  content
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MS2LDA
  role: Performs bag-of-fragments conversion with neutral-loss extraction and noise
    filtering to preprocess MS/MS spectra before LDA inference
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: Thermo Xcalibur Tune 2.5
  role: Instrument control and data acquisition software producing the raw MS/MS files
    that feed preprocessing
- name: MS-DIAL
  role: Candidate tool for matching fragments to MS1 ions in data-independent acquisition
    data prior to motif extraction
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:419bd725cf7d68b7dced3ca2767170da786a88d4119f2fc3149bcde95259140e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS-DIAL
  - MS2LDA
  - Thermo Xcalibur Tune 2.5
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mass-spectrometry-preprocessing

## Summary

Convert raw LC-MS/MS spectra into cleaned bag-of-fragments documents for topic modeling by extracting neutral losses, filtering noise, and matching MS1 peaks to MS2 spectra, as implemented in the MS2LDA workflow.

## When to use

Use when raw MS/MS data from complex mixtures (e.g., beer or urine extracts on a Q-Exactive Orbitrap) must be converted into fragment/neutral-loss documents before unsupervised substructure discovery with LDA.

## When NOT to use

- The analysis goal is classical spectral similarity clustering or targeted compound identification (e.g., MS2Analyzer-style rule-based matching) rather than unsupervised substructure discovery
- Input is already a preprocessed feature table or bag-of-fragments document — rerunning extraction would be redundant
- Data are information-poor MS2 spectra (e.g., fixed low collision energy on simple mixtures), where MS2LDA preprocessing adds little since motifs require rich fragment/neutral-loss content

## Inputs

- Raw centroided LC-MS/MS data files (e.g., Beer3 positive-ionization mode file from dx.doi.org/10.5525/gla.researchdata.313)
- Positive and negative ionization mode spectra from a Thermo Q-Exactive Orbitrap (HESI II interface)
- MS1 peak lists requiring matching to MS2 spectra

## Outputs

- Bag-of-fragments documents per spectrum (fragments plus extracted neutral losses after noise filtering)
- Preprocessed peak tables of ~1,000 MS peaks per sample per ionization mode
- Stage-wise runtime benchmarks (e.g., runtime_benchmark.csv with feature-extraction wall-clock time)

## How to apply

Start from centroided MS/MS data (e.g., Beer3 positive-ionization mode file acquired on a Thermo Q-Exactive Orbitrap with an Ultimate 3000 RSLCnano LC system, instrument-controlled via Thermo Xcalibur Tune 2.5). Run the MS2LDA preprocessing stage: convert each spectrum into a bag of fragments, extract neutral losses, and filter out noise peaks. Ensure reliable matching of MS1 peaks to MS2 spectra, since LDA motif-to-molecule links depend on this assignment. After preprocessing, each beer sample should contain around 1,000 MS peaks in both positive and negative ionization modes. Time the feature-extraction stage for benchmarking; on laptop-class hardware it takes ~20 minutes per beer sample, consistent with the claim that MS2LDA is 'more costly than simpler tools, but not prohibitively so.'

## Related tools

- **MS2LDA** (Performs bag-of-fragments conversion with neutral-loss extraction and noise filtering to preprocess MS/MS spectra before LDA inference) — https://github.com/vdhooftcompmet/MS2LDA
- **Thermo Xcalibur Tune 2.5** (Instrument control and data acquisition software producing the raw MS/MS files that feed preprocessing)
- **MS-DIAL** (Candidate tool for matching fragments to MS1 ions in data-independent acquisition data prior to motif extraction)

## Examples

```
pip install ms2lda; python -c "import time; t0=time.time(); # run MS2LDA feature extraction on Beer3.mzML; print('feature extraction:', time.time()-t0, 's')"
```

## Evaluation signals

- Each preprocessed sample should contain around 1,000 MS peaks in both positive and negative ionization mode
- Feature-extraction wall-clock time should fall within the same order of magnitude as the reported ~20 minutes per beer sample on laptop-class hardware
- Noise peaks and neutral losses must be handled consistently: neutral losses extracted and noise filtered per the MS2LDA methods
- MS1-to-MS2 peak assignments must be verified as reliable, since Mass2Motif links to molecules depend on them
- Downstream LDA with 300 Mass2Motifs per data file should run without errors on the preprocessed documents, confirming document format validity

## Limitations

- Neutral losses are not currently supported by spectral matching tools and had to be omitted in automated annotation
- MS2LDA works best only for complex mixtures with information-rich MS2 spectra (e.g., ramped or stepped collision energy)
- No version number for MS2LDA is stated in the section

## Evidence

- [methods] Preprocessing converts spectra to bag-of-fragments documents: [evidence span withheld — anchor status: not_found]
- [discussion] MS1-MS2 matching requirement: [evidence span withheld — anchor status: not_found]
- [readme] README installation path: [evidence span withheld — anchor status: external_doc]
