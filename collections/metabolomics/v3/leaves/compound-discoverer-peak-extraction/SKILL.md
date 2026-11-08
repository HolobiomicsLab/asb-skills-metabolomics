---
name: compound-discoverer-peak-extraction
description: Use when starting from Thermo .raw LC-MS lipidomics files and a downstream
  peak table is needed — specifically when the pipeline requires Compound Discoverer
  3.3 .cdResult files as input to LipiDex 2 Peak Finder.
when_to_use_negative:
- Input is already a chromatographic peak/feature table (.csv from mzMine 2 or earlier
  Compound Discoverer versions) rather than Thermo .raw files — Peak Finder specifically
  requires version 3.3 .cdResult files.
- The analysis is targeted (a predefined analyte list with extracted ion chromatograms)
  rather than untargeted discovery feature extraction.
- The data are not Thermo .raw files or the downstream pipeline does not accept Compound
  Discoverer .cdResult output.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: Compound Discoverer
  role: Peak-extraction engine that generates the .cdResult peak table from Thermo
    .raw files; LipiDex 2 Peak Finder requires version 3.3.
  repo: https://www.thermofisher.com/us/en/home/industrial/mass-spectrometry/liquid-chromatography-mass-spectrometry-lc-ms/lc-ms-software/multi-omics-data-analysis/compound-discoverer-software.html
- name: LipiDex 2 Peak Finder
  role: Downstream consumer of the .cdResult peak table; reads it directly without
    intermediate .csv conversion and performs lipid identification and filtering.
  repo: https://github.com/coongroup/LipiDex-2/releases/latest
- name: LipiDex 2 Spectrum Searcher
  role: Performs spectral library matching (75% spectral purity, dot product 500/700)
    against the in silico lipid library after peak extraction.
  repo: https://github.com/coongroup/LipiDex-2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c00359
    title: LipiDex 2 Integrates MS<sup>n</sup> Tree-Based Fragmentation Methods and
      Quality Control Modules to Improve Discovery Lipidomics
  source_content_hash: sha256:abea179c53a63393a899d32efbb998118262ca802551977908d378d699422dd8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Compound Discoverer
  - LipiDex 2
  - LipiDex 2 Peak Finder
  - LipiDex 2 Spectrum Searcher
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# compound-discoverer-peak-extraction

## Summary

Reproduces untargeted LC-MS peak/feature extraction from Thermo .raw files using Compound Discoverer 3.3 with the LipiDex 2-compatible parameter set (m/z 100–5000, RT 0.4–21 min, 5e5 intensity threshold), producing the .cdResult peak table that LipiDex 2 Peak Finder consumes directly.

## When to use

Use when starting from Thermo .raw LC-MS lipidomics files and a downstream peak table is needed — specifically when the pipeline requires Compound Discoverer 3.3 .cdResult files as input to LipiDex 2 Peak Finder.

## When NOT to use

- Input is already a chromatographic peak/feature table (.csv from mzMine 2 or earlier Compound Discoverer versions) rather than Thermo .raw files — Peak Finder specifically requires version 3.3 .cdResult files.
- The analysis is targeted (a predefined analyte list with extracted ion chromatograms) rather than untargeted discovery feature extraction.
- The data are not Thermo .raw files or the downstream pipeline does not accept Compound Discoverer .cdResult output.

## Inputs

- Thermo .raw LC-MS/MS data files (e.g. 40 Nile rat plasma experimental samples from MSV000091033)
- Compound Discoverer 3.3 workflow with parameters: m/z 100–5000, RT 0.4–21 min, 5e5 intensity threshold
- In silico lipid spectral library from LipiDex 2 Library Generator (for downstream matching)

## Outputs

- Compound Discoverer 3.3 .cdResult peak table of chromatographic features
- Feature counts (expected ~14826 unidentified / 779 identified pre-filter; 13898 / 719 after Peak Finder filtering on the reference dataset)
- LipiDex2_PeakFinder_Results.csv after downstream LipiDex 2 processing

## How to apply

Configure a Compound Discoverer 3.3 peak-extraction workflow with m/z range 100–5000, retention time range 0.4–21 min, and an intensity threshold of 5e5, then run it on the Thermo .raw files (e.g. the 40 Nile rat plasma samples from MSV000091033). Record the exact parameter set so the extraction is reproducible. Export/retain the .cdResult file rather than converting to .csv, because LipiDex 2 Peak Finder reads Compound Discoverer .cdResult files directly, avoiding intermediate peak-table conversion. Downstream, apply LipiDex 2 Spectrum Searcher (75% spectral purity, dot product 500/700) and Peak Finder filtering; on the MSV000091033 reference run, the unfiltered extraction yields 14826 unidentified features and 779 identified lipid features, which Peak Finder filtering reduces to 13898 unidentified and 719 identified.

## Related tools

- **Compound Discoverer** (Peak-extraction engine that generates the .cdResult peak table from Thermo .raw files; LipiDex 2 Peak Finder requires version 3.3.) — https://www.thermofisher.com/us/en/home/industrial/mass-spectrometry/liquid-chromatography-mass-spectrometry-lc-ms/lc-ms-software/multi-omics-data-analysis/compound-discoverer-software.html
- **LipiDex 2 Peak Finder** (Downstream consumer of the .cdResult peak table; reads it directly without intermediate .csv conversion and performs lipid identification and filtering.) — https://github.com/coongroup/LipiDex-2/releases/latest
- **LipiDex 2 Spectrum Searcher** (Performs spectral library matching (75% spectral purity, dot product 500/700) against the in silico lipid library after peak extraction.) — https://github.com/coongroup/LipiDex-2

## Evaluation signals

- The .cdResult file is generated by Compound Discoverer version 3.3 (not an earlier version) and is passed directly to Peak Finder without .csv intermediate conversion.
- On MSV000091033, the extracted peak table contains exactly 14826 unidentified features and 779 identified lipid features.
- After LipiDex 2 Peak Finder filtering (adduct, dimer, and spurious feature removal), counts drop to 13898 unidentified and 719 identified.
- Extraction parameters match the reference set: m/z 100–5000, RT 0.4–21 min, 5e5 intensity threshold.
- Downstream QC metrics are healthy: features with >30% RSD in QC samples are flagged for removal and RT deviations are consistent with the ECN vs. RT model.

## Limitations

- Peak Finder requires Compound Discoverer version 3.3 .cdResult files specifically; output from other versions or other feature-detection software must be converted or is unsupported.
- Counts are dataset-dependent — the 14826/779 and 13898/719 figures reproduce only for the MSV000091033 Nile rat plasma dataset with the exact parameters listed.
- LipiDex 2 requires .NET 7.0 runtime, which the operating system may prompt to install when LipiDex2.exe is first run.

## Evidence
