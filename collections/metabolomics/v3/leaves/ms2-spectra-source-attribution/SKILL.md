---
name: ms2-spectra-source-attribution
description: Use when combining MS2 spectra from paired DDA and DIA-(SWATH) acquisitions
  of the same sample set and you need each spectrum record in the exported MGF labeled
  with its DDA or DIA provenance.
when_to_use_negative:
- The dataset was acquired in a single mode only (pure DDA or pure DIA) — there is
  no cross-mode provenance to attribute.
- Input is already a combined feature table or MGF without the raw DDA/DIA mzXML files,
  so centWave-based re-detection and per-spectrum tagging cannot be performed.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DaDIA.R
  role: R program that extracts metabolic features from both modes, merges the feature
    lists, and exports DDA/DIA-attributed MS2 spectra to MGF
  repo: https://github.com/Huan Lab/DaDIA.git
- name: XCMS
  role: centWave peak-picking engine invoked by DaDIA.R to detect features in the
    DDA and DIA mzXML inputs (version 3.11.4 or higher)
  repo: https://github.com/Huan Lab/DaDIA.git
- name: R
  role: Runtime environment for DaDIA.R (version 4.0 or above)
  repo: https://github.com/Huan Lab/DaDIA.git
- name: metaMS
  role: Required companion R package (version 1.25.1) for the DaDIA.R processing pipeline
  repo: https://github.com/Huan Lab/DaDIA.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.0c05022
    title: 'DaDIA: Hybridizing Data-Dependent and Data-Independent Acquisition Modes
      for Generating High-Quality Metabolomic Data'
  source_content_hash: sha256:b94fbef40e00b51f41d968b470d1c1c4dcbe503dac48c92262e239a5105d62f1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Huan Lab/DaDIA.git
  tools:
  - DaDIA.R
  - R
  - XCMS
  - metaMS
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# ms2-spectra-source-attribution

## Summary

Tags each MS2 spectrum exported from a DaDIA workflow with its acquisition origin — DDA (pooled QC runs) or DIA (individual biological samples) — while merging DDA- and DIA-derived feature lists into a single metabolite-intensity table via DaDIA.R.

## When to use

Use when combining MS2 spectra from paired DDA and DIA-(SWATH) acquisitions of the same sample set and you need each spectrum record in the exported MGF labeled with its DDA or DIA provenance.

## When NOT to use

- The dataset was acquired in a single mode only (pure DDA or pure DIA) — there is no cross-mode provenance to attribute.
- Input is already a combined feature table or MGF without the raw DDA/DIA mzXML files, so centWave-based re-detection and per-spectrum tagging cannot be performed.

## Inputs

- DDA mzXML files from pooled QC sample acquisitions
- DIA-(SWATH) mzXML files from individual biological sample acquisitions
- MoNA MS2 spectral library (91,819 standard MS2 spectra) for downstream annotation of the attributed spectra

## Outputs

- ms2_spectra.mgf — combined MS2 spectra with DDA/DIA origin tagged in each record's header
- metabolite_intensity_table.csv — merged DDA+DIA metabolite-intensity feature table

## How to apply

Run DaDIA.R (R ≥ 4.0, XCMS ≥ 3.11.4, metaMS 1.25.1) on the DDA and DIA-(SWATH) mzXML files, using centWave peak picking to detect features in both modes. Merge the DDA-derived and DIA-derived feature lists into one metabolite-intensity table, aligning cross-mode features with the unique feature information from the DDA data, and save it as metabolite_intensity_table.csv. Combine MS2 spectra from both modes into ms2_spectra.mgf, tagging each spectrum record's DDA or DIA origin in the MGF header. Before writing the MGF, confirm the number of samples set in DaDIA.R (line 38) matches the real number of DIA mzXML files, since the script counts on this to partition DIA input correctly. As a sanity anchor, in the human urine test data the exported MS2 spectra split approximately 63% DDA / 37% DIA; a very different split suggests mis-tagged or dropped spectra.

## Related tools

- **DaDIA.R** (R program that extracts metabolic features from both modes, merges the feature lists, and exports DDA/DIA-attributed MS2 spectra to MGF) — https://github.com/Huan Lab/DaDIA.git
- **XCMS** (centWave peak-picking engine invoked by DaDIA.R to detect features in the DDA and DIA mzXML inputs (version 3.11.4 or higher)) — https://github.com/Huan Lab/DaDIA.git
- **R** (Runtime environment for DaDIA.R (version 4.0 or above)) — https://github.com/Huan Lab/DaDIA.git
- **metaMS** (Required companion R package (version 1.25.1) for the DaDIA.R processing pipeline) — https://github.com/Huan Lab/DaDIA.git

## Examples

```
Rscript DaDIA.R  # after setting the DIA sample count on line 38 to the real number of DIA mzXML files; outputs metabolite_intensity_table.csv and ms2_spectra.mgf
```

## Evaluation signals

- ms2_spectra.mgf parses as valid MGF format with every spectrum record carrying a DDA or DIA origin tag in its header.
- metabolite_intensity_table.csv exists and contains the expected metabolite-intensity columns.
- The DDA/DIA source split of exported MS2 spectra is consistent with the reported benchmark: 63% DDA / 37% DIA in the urine test data.
- The sample count at line 38 of DaDIA.R equals the actual number of DIA mzXML files supplied, otherwise DIA spectra may be mis-attributed or omitted.
- Total MS2 feature count of the merged table is roughly the sum of mode contributions (DaDIA reported 4901 MS2 features vs 956 DDA and 4024 DIA(SWATH) in urine).

## Limitations

- Requires the exact toolchain versions (R ≥ 4.0, XCMS ≥ 3.11.4, metaMS 1.25.1); older XCMS versions will not run DaDIA.R.
- The hard-coded sample count in DaDIA.R (line 38) must be manually edited to match the number of DIA mzXML files — a mismatch silently breaks the DIA processing.
- The 63%/37% DDA/DIA split is dataset-specific (urine, RP(+) mode); other matrices or chromatographic modes will shift the proportions.

## Evidence
