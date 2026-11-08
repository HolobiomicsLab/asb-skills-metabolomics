---
name: replicate-consensus-spectrum-assembly
description: Use when replicate DDA or SWATH-DIA MS2 spectral files have been deconvoluted
  (DecoID-style regression for DDA, DecoMetDIA-style C++/Rcpp for DIA) and must be
  consolidated into per-feature consensus spectra with precursor-fragment associations
  and compound annotations.
when_to_use_negative:
- Input is already an MS1 feature/peak-abundance table (e.g. asari output) with no
  MS2 spectra to consolidate
- Only a single MS2 injection exists, so replicate-level clustering and intensity
  averaging across injections cannot be performed
- The goal is deconvolution of chimeric spectra itself rather than consolidation of
  already-deconvoluted replicate spectra
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MetaboAnalystR 4.0
  role: R package providing the MS/MS spectra deconvolution and compound annotation
    functions, replicate merging into consensus spectra, and the bundled ~1.5 million
    MS2 spectra reference database
  repo: https://github.com/xia-lab/MetaboAnalystR
- name: MetaboAnalyst 6.0
  role: Web platform whose enhanced Spectra Processing module performs MS2 peak detection/deconvolution
    and consensus assembly underlying this skill
- name: R
  role: Execution environment (R base version > 3.6.1 recommended) in which MetaboAnalystR
    deconvolution and consensus-assembly functions are run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkae253
    title: 'MetaboAnalyst 6.0: towards a unified platform for metabolomics data processing,
      analysis and interpretation'
  source_content_hash: sha256:bffd5146032866f5a96599d79557fa8a0b2c02406773bddd1edc272e04e46ea3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xia-lab/MetaboAnalystR
  tools:
  - MetaboAnalyst 6.0
  - MetaboAnalystR
  - MetaboAnalystR 4.0
  - R
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- replicate-ms2-spectrum-consensus
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# replicate-consensus-spectrum-assembly

## Summary

Merges deconvoluted MS2 spectra from replicate injections into consensus spectra by clustering fragments on m/z and retention time and averaging intensities, then annotates fragments against the MetaboAnalystR MS2 reference library (~1.5 million spectra) as implemented in the MetaboAnalyst 6.0 / MetaboAnalystR 4.0 Spectra Processing module.

## When to use

Use when replicate DDA or SWATH-DIA MS2 spectral files have been deconvoluted (DecoID-style regression for DDA, DecoMetDIA-style C++/Rcpp for DIA) and must be consolidated into per-feature consensus spectra with precursor-fragment associations and compound annotations.

## When NOT to use

- Input is already an MS1 feature/peak-abundance table (e.g. asari output) with no MS2 spectra to consolidate
- Only a single MS2 injection exists, so replicate-level clustering and intensity averaging across injections cannot be performed
- The goal is deconvolution of chimeric spectra itself rather than consolidation of already-deconvoluted replicate spectra

## Inputs

- Replicate DDA MS2 spectra files (deconvoluted via DecoID-style regression)
- Replicate SWATH-DIA MS2 spectra files (deconvoluted via DecoMetDIA-style algorithm)
- MetaboAnalystR-bundled MS2 reference spectra database (~1.5 million spectra)

## Outputs

- consensus_spectra.csv containing precursor-association fields and fragment-level compound-annotation fields
- Per-feature consensus MS2 spectra suitable for MS2 Peak Annotation and compound identification

## How to apply

After deconvolution of each replicate's chimeric MS2 spectra (DecoID database-dependent sparse regression for DDA; DecoMetDIA correlating extracted-ion chromatograms within each isolation window for SWATH-DIA, re-written in C++ via Rcpp), cluster deconvoluted fragment ions across replicate injections by m/z and retention time and average their intensities to build consensus spectra. Load the MetaboAnalystR 4.0-bundled MS2 reference spectra database (~1.5 million spectra) in R to annotate each consensus-spectrum fragment with fragment-level compound assignments and precursor associations. Tandem MS spectra are typically collected from pooled QC samples during untargeted experiments, so replicate QC MS2 files are the expected primary input. The rationale is that consolidating replicates suppresses stochastic fragment noise and, together with deconvolution, increases true-positive chemical identification rates (MetaboAnalystR 4.0 reports >40% improvement in identification TPR for both DDA and DIA datasets). Write the consolidated result as consensus_spectra.csv containing precursor-association and compound-annotation fields for downstream MS2 Peak Annotation and functional interpretation.

## Related tools

- **MetaboAnalystR 4.0** (R package providing the MS/MS spectra deconvolution and compound annotation functions, replicate merging into consensus spectra, and the bundled ~1.5 million MS2 spectra reference database) — https://github.com/xia-lab/MetaboAnalystR
- **MetaboAnalyst 6.0** (Web platform whose enhanced Spectra Processing module performs MS2 peak detection/deconvolution and consensus assembly underlying this skill)
- **R** (Execution environment (R base version > 3.6.1 recommended) in which MetaboAnalystR deconvolution and consensus-assembly functions are run)

## Evaluation signals

- consensus_spectra.csv exists and contains both precursor-association and compound-annotation fields
- Expert review confirms deconvoluted spectra of known reference compounds match their library spectra
- Fragments are clustered on m/z and retention time across replicate injections, with averaged (not concatenated) intensities
- Consensus spectra trace back to deconvoluted spectra in which fragments were assigned to precursor ions (DDA regression or DIA isolation-window EIC correlation)
- Annotations resolve against the MetaboAnalystR-bundled ~1.5 million MS2 reference spectra database

## Limitations

- DIA (SWATH) MS2 data require prior deconvolution to reconstruct precursor-ion associations before consensus assembly is meaningful
- Downstream metabolome annotations remain an acknowledged area for improvement in MetaboAnalyst 6.0
- Coverage gains depend on replicate pooled-QC MS2 acquisitions; identification TPR improvements (>40%) were reported for benchmark DDA and DIA datasets and may not transfer to all cohorts

## Evidence

- [intro] MS2 spectra from pooled QC feed consensus assembly: "To assist with compound identification, tandem MS (called MS/MS or MS2) spectra are often collected from pooled QC samples during the experiments"
- [intro] DIA deconvolution precedes consensus reconstruction: "DIA data generally require deconvolution of the MS2 data to reconstruct associations with their precursor ions"
- [readme] README: deconvolution module scope: "a streamlined MS/MS spectra deconvolution and compound annotation module for both data-dependent acquisition (DDA) or data-independent acquisition (DIA)"
- [readme] README: bundled reference spectra scale: "MetaboAnalystR 4.0 comes with a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
- [methods] Methods: local spectra databases: "a large collection of knowledgebases (~500,000 entries of metabolite sets) and spectra databases (~1.5 million MS2 spectra)"
