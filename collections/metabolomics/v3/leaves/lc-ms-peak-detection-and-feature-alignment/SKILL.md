---
name: lc-ms-peak-detection-and-feature-alignment
description: Use when raw LC-MS/MS chromatogram files (e.g., beer extracts in positive
  and negative ionization modes) must be converted into a bag-of-fragments matrix
  with MS2 features as rows and MS2 spectra as columns before LDA inference.
when_to_use_negative:
- Input is already a feature table or preprocessed peak matrix — no raw chromatogram
  peak detection or MS1–MS2 pairing is needed
- Data are simple mixtures with low-information MS2 spectra (e.g., fixed collision
  energy producing few fragments) rather than complex mixtures with ramped or stepped
  collision energy
- Downstream tool does not accept neutral losses — MS2LDA motifs include neutral-loss
  features that spectral matching tools currently omit
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0092
tools:
- name: XCMS
  role: Detects MS1 peaks from raw chromatographic data as the first preprocessing
    step
- name: PeakML/mzMatch
  role: Refines XCMS-detected features so each preprocessed sample yields ~1,000 MS
    peaks
- name: RMassBank
  role: Reliably pairs each MS1 peak with its corresponding MS2 fragmentation spectrum
- name: MS2LDA.Preprocessing.load_and_clean
  role: Removes noise entries from fragment and neutral-loss lists before corpus generation
  repo: https://github.com/vdhooftcompmet/MS2LDA
- name: MS2LDA.Preprocessing.generate_corpus
  role: Aligns MS2 features across all spectra and builds the bag-of-fragments corpus
    including extracted neutral losses
  repo: https://github.com/vdhooftcompmet/MS2LDA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1073/pnas.1608041113
    title: Topic modeling for untargeted substructure exploration in metabolomics
  source_content_hash: sha256:3bad43ce990a20a3edb7f2b2fa704d1e7a567f9d5b2c9e60b99a59b103e2b210
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/vdhooftcompmet/MS2LDA
  tools:
  - MS2LDA.Preprocessing.generate_corpus
  - MS2LDA.Preprocessing.load_and_clean
  - PeakML/mzMatch
  - RMassBank
  - XCMS
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lc-ms-peak-detection-and-feature-alignment

## Summary

Detect and align LC-MS/MS features from raw spectra using XCMS, PeakML/mzMatch, and RMassBank, then build the MS2 feature-by-spectrum integer count matrix that downstream unsupervised substructure discovery (MS2LDA) requires as input.

## When to use

Use when raw LC-MS/MS chromatogram files (e.g., beer extracts in positive and negative ionization modes) must be converted into a bag-of-fragments matrix with MS2 features as rows and MS2 spectra as columns before LDA inference.

## When NOT to use

- Input is already a feature table or preprocessed peak matrix — no raw chromatogram peak detection or MS1–MS2 pairing is needed
- Data are simple mixtures with low-information MS2 spectra (e.g., fixed collision energy producing few fragments) rather than complex mixtures with ramped or stepped collision energy
- Downstream tool does not accept neutral losses — MS2LDA motifs include neutral-loss features that spectral matching tools currently omit

## Inputs

- Raw LC-MS/MS data files (e.g., four beer extract files from dx.doi.org/10.5525/gla.researchdata.313) in positive and negative ionization modes
- MS1 peak lists refined by XCMS and PeakML/mzMatch (~1,000 peaks per sample)
- MS2 fragmentation spectra paired to MS1 peaks by RMassBank

## Outputs

- A bag-of-fragments matrix with MS2 features (fragments and neutral losses) as rows and MS2 spectra as columns, with values as nonnegative integer occurrence counts
- A well-formed CSV of integer count values ready for LDA inference

## How to apply

Detect MS1 peaks on raw chromatogram files with XCMS and refine features with PeakML/mzMatch so that each preprocessed sample yields roughly 1,000 MS peaks in both positive and negative ionization modes. Reliably pair each MS1 peak with its corresponding MS2 fragmentation spectrum using RMassBank. Apply MS2LDA.Preprocessing.load_and_clean to remove noise entries from the fragment and neutral-loss lists, then use MS2LDA.Preprocessing.generate_corpus to align MS2 features across all spectra of each sample, building a bag-of-fragments representation that includes extracted neutral losses. Assemble the aligned features into a single matrix with fragments/losses as rows and MS2 spectra as columns, transforming intensities into integer occurrence counts.

## Related tools

- **XCMS** (Detects MS1 peaks from raw chromatographic data as the first preprocessing step)
- **PeakML/mzMatch** (Refines XCMS-detected features so each preprocessed sample yields ~1,000 MS peaks)
- **RMassBank** (Reliably pairs each MS1 peak with its corresponding MS2 fragmentation spectrum)
- **MS2LDA.Preprocessing.load_and_clean** (Removes noise entries from fragment and neutral-loss lists before corpus generation) — https://github.com/vdhooftcompmet/MS2LDA
- **MS2LDA.Preprocessing.generate_corpus** (Aligns MS2 features across all spectra and builds the bag-of-fragments corpus including extracted neutral losses) — https://github.com/vdhooftcompmet/MS2LDA

## Examples

```
git clone https://github.com/vdhooftcompmet/MS2LDA.git
```

## Evaluation signals

- Output CSV exists and is well-formed
- All matrix values are nonnegative integers (occurrence counts, not raw intensities)
- Per-sample peak counts are consistent with the reported ~1,000 MS peaks per sample in both positive and negative ionization modes
- Matrix rows include both fragment features and extracted neutral-loss features, not fragments alone
- Number of matrix columns matches the number of MS2 spectra in each preprocessed sample

## Limitations

- Works best for complex mixtures where a large number of metabolites are fragmented and information-rich MS2 spectra are available (e.g., generated by ramped or stepped collision energy)
- Neutral losses are not currently supported by spectral matching tools and had to be omitted in automated annotation
- Computationally more costly than simpler tools, but not prohibitively so
- No version number for MS2LDA is stated in the published methods

## Evidence

- [discussion] MS2LDAreliesonreliablematchingofMS1peakstoMS2spec-tra: [evidence span withheld — anchor status: not_found]
