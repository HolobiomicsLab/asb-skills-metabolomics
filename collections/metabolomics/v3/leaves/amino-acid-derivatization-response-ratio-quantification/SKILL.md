---
name: amino-acid-derivatization-response-ratio-quantification
description: Use when GC-MS peak-area tables of amino acids paired with stable-isotope-labeled
  internal standards are available and you need to compare precision (RSD) across
  derivatization reagents or report one-to-one ISTD-normalized amino-acid quantities.
when_to_use_negative:
- Input is a large-scale untargeted GC-MS feature table lacking paired stable-isotope-labeled
  internal standards — use SERRF/SERDA-style normalization instead (the paper found
  neither internal-standard nor sum-based normalization achieved <30% median RSD across
  563 annotations).
- Peak areas come from LC-MS or non-derivatized analyses where the PCF/MTBSTFA/MSTFA
  reagent comparison does not apply.
- The goal is batch correction across 1.2 years of acquisitions with column changes
  rather than derivatization-precision benchmarking on a small sample set.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: R
  role: Implements the one-to-one ISTD normalization (peak-area division per cognate
    labeled standard) and per-trial RSD computation across the 30 BioIVT plasma samples
  repo: https://github.com/slfan2013/SERDA
- name: gas chromatography-mass spectrometry (GC-MS)
  role: Generates the derivatized amino-acid peak areas under PCF, MTBSTFA, and MSTFA
    derivatization that feed the response-ratio calculation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13080944
    title: Denoising Autoencoder Normalization for Large-Scale Untargeted Metabolomics
      by Gas Chromatography–Mass Spectrometry
  source_content_hash: sha256:1d90db279b9336ec0399cde5c138453ec48eb629e1e33380702e765f68859461
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/slfan2013/SERDA
  tools:
  - R
  - gas chromatography-mass spectrometry (GC-MS)
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# amino-acid-derivatization-response-ratio-quantification

## Summary

Quantifies derivatized amino acids in GC-MS by dividing each amino acid's raw peak area by its cognate stable-isotope-labeled internal standard, then computing per-compound RSDs across samples to benchmark derivatization agents (PCF vs MTBSTFA vs MSTFA). It grounds derivatization-reagent selection in measured precision rather than vendor claims.

## When to use

Use when GC-MS peak-area tables of amino acids paired with stable-isotope-labeled internal standards are available and you need to compare precision (RSD) across derivatization reagents or report one-to-one ISTD-normalized amino-acid quantities.

## When NOT to use

- Input is a large-scale untargeted GC-MS feature table lacking paired stable-isotope-labeled internal standards — use SERRF/SERDA-style normalization instead (the paper found neither internal-standard nor sum-based normalization achieved <30% median RSD across 563 annotations).
- Peak areas come from LC-MS or non-derivatized analyses where the PCF/MTBSTFA/MSTFA reagent comparison does not apply.
- The goal is batch correction across 1.2 years of acquisitions with column changes rather than derivatization-precision benchmarking on a small sample set.

## Inputs

- Peak-area table (e.g., Supplementary Table S1 as .xlsx) with 16 amino-acid peak areas and paired stable-isotope-labeled internal-standard peak areas per injection
- Sample metadata mapping 30 BioIVT plasma samples to derivatization agent (PCF, MTBSTFA, MSTFA) and monthly replicate trial

## Outputs

- istd_aminoacid_rsd.csv — summary RSD (%) per derivatization reagent, averaged across the three monthly replicate trials
- Per-amino-acid RSD values within each derivatization reagent and replicate trial

## How to apply

Load the peak-area table (e.g., Supplementary Table S1) containing the 16 detectable amino acids and their paired internal standards for each injection (30 BioIVT plasma samples per monthly replicate trial). Apply one-to-one ISTD normalization in R by dividing each amino acid's raw peak area by its cognate stable-isotope-labeled internal-standard peak area for every injection — never a global or sum-based normalization, which underperforms per-analyte isotope pairing. Within each derivatization reagent (PCF, MTBSTFA, MSTFA) and each monthly replicate trial, compute the RSD (%) of each ISTD-normalized amino acid across the 30 samples, then average the per-trial amino-acid RSDs across the three replicate trials to yield one summary RSD per reagent. Validate the reproduced averages against reference values (PCF 2.7%, MTBSTFA 8.9%, MSTFA 9.6%); PCF's low RSD reflects its in-solution derivatization with isotopically labeled internal standards. Write the summary table out (e.g., istd_aminoacid_rsd.csv).

## Related tools

- **R** (Implements the one-to-one ISTD normalization (peak-area division per cognate labeled standard) and per-trial RSD computation across the 30 BioIVT plasma samples) — https://github.com/slfan2013/SERDA
- **gas chromatography-mass spectrometry (GC-MS)** (Generates the derivatized amino-acid peak areas under PCF, MTBSTFA, and MSTFA derivatization that feed the response-ratio calculation)

## Examples

```
Rscript -e 'd<-read.csv("SuppTableS1.csv"); r<-d[,aa]/d[,istd]; rsd<-sd(r,na.rm=TRUE)/mean(r)*100; write.csv(rsd,"istd_aminoacid_rsd.csv")'
```

## Evaluation signals

- Reproduced average RSDs match reported references: PCF 2.7%, MTBSTFA 8.9%, MSTFA 9.6% (Figure 3 / Supplementary Table S1).
- Each amino acid is divided only by its own cognate labeled internal standard — verify the pairing map is one-to-one, not pooled or sum-based.
- RSDs must be computed within each of the three monthly replicate trials (30 samples each) before averaging across trials; a single pooled-RSD shortcut is a wrong result.
- Output file istd_aminoacid_rsd.csv contains exactly one summary RSD row per derivatization reagent (3 rows).

## Limitations

- Applies only to the 16 detectable amino acids with matched stable-isotope-labeled standards; unannotated or unlabeled metabolites cannot use this scheme.
- The paper shows per-analyte ISTD precision does not scale to large untargeted panels — neither internal standards nor sum-based normalizations achieved median precision below 30% across all 563 metabolite annotations.
- Precision was evaluated on BioIVT pooled plasma QC samples in three monthly replicate trials, so absolute RSDs may not transfer to other matrices or cohorts.

## Evidence

- [other] PCF derivatization gave the lowest precision with 2.7% average RSD, followed by MTBSTFA at 8.9% RSD and MSTFA at 9.6% RSD.: [evidence span withheld — anchor status: not_found]
- [abstract] neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations: "neither internal standards nor sum-based normalizations yielded median precision of less than 30% across all 563 metabolite annotations"
- [readme] Open it and edit data_file = "filelocation.xlsx" and run all the script lines.: "Download `SERDA runner.R`. Open it and edit `data_file = "filelocation.xlsx"` and run all the script lines."
- [abstract] external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD): "external cross-validation with BioIVT plasma pools yielded a median 34% relative standard deviation (RSD)"
