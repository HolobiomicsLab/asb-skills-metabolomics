---
name: hrmsms-spectral-data-parsing
description: 'Reference for preparing MetFrag2.2 input spectra: merging duplicate
  HR-MS/MS records, dropping spectra that contain only the precursor peak, and supplying
  parameter files, candidate lists and spectral data to MetFrag or its R/CLI variants.'
when_to_use_negative:
- The spectrum contains only the precursor peak and no other fragment peaks — MetFrag2.2
  cannot meaningfully rank candidates from such spectra.
- You need low-resolution or MS1-only data; this skill is specific to HR-MS/MS product-ion
  spectra used in MetFrag bond dissociation matching.
- You are re-training or evaluating a different fragmenter (e.g. CFM-ID on Orbitrap
  data) rather than preparing input for MetFrag2.2.
edam_operation: http://edamontology.org/operation_3637
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: MetFrag2.2
  role: Receives the parsed and merged HR-MS/MS spectra and performs in silico bond
    dissociation fragmentation and candidate scoring.
  repo: http://c-ruttkies.github.io/MetFrag/
- name: xcms (mzClust_hclust)
  role: Merges replicate/variant HR-MS/MS spectra into one peak list before MetFrag
    scoring.
- name: metfRag (MetFragR)
  role: R wrapper that consumes the parsed spectra and parameter files to run MetFrag
    scoring within R.
  repo: https://github.com/ipb-halle/MetFragRelaunched
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-016-0115-9
    title: 'MetFrag relaunched: incorporating strategies beyond in silico fragmentation'
  source_content_hash: sha256:89b0c114f778e8900e8b3113291742c0e2bcc1d1281772bf0d394bc5a895048d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetFrag2.2
  - metfRag (MetFragR)
  - xcms (mzClust_hclust)
  edam_operation: http://edamontology.org/operation_3637
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# hrmsms-spectral-data-parsing

## Summary

Parse and prepare high-resolution tandem mass spectrometry (HR-MS/MS) peak lists for candidate ranking with MetFrag2.2, including merging replicate spectra and retaining only spectra with informative fragment peaks.

## When to use

Reference for preparing MetFrag2.2 input spectra: merging duplicate HR-MS/MS records, dropping spectra that contain only the precursor peak, and supplying parameter files, candidate lists and spectral data to MetFrag or its R/CLI variants.

## When NOT to use

- The spectrum contains only the precursor peak and no other fragment peaks — MetFrag2.2 cannot meaningfully rank candidates from such spectra.
- You need low-resolution or MS1-only data; this skill is specific to HR-MS/MS product-ion spectra used in MetFrag bond dissociation matching.
- You are re-training or evaluating a different fragmenter (e.g. CFM-ID on Orbitrap data) rather than preparing input for MetFrag2.2.

## Inputs

- Unmerged HR-MS/MS spectra records for a single substance (e.g. MassBank records with instrument/energy variants)
- xcms package (R) for mzClust_hclust spectrum merging
- Parameter files with scoring weights and candidate/result/ranking files from the deposited EA, UF, EQEx, EQExPlus or HILL ZIP archive
- Candidate structure list (PubChem/ChemSpider retrieval or user-supplied SDF; InChIKeys from OpenBabel for suspect lists)

## Outputs

- Merged HR-MS/MS peak list (m/z and intensity pairs) ready for MetFrag2.2 fragment matching
- Filtered spectrum set containing only spectra with at least one non-precursor fragment peak
- Parameter files for MetFrag2.2 runs (spectral data, candidates, scoring settings)

## How to apply

Convert measured HR-MS/MS spectra of the same compound (e.g. from MassBank records with their instrument/energy variants) into a single merged peak list using the mzClust_hclust function in xcms with eppm = 5×10^-6 and eabs = 0.001 Da. Retain only spectra that contain at least one fragment peak other than the precursor, since spectra dominated by one or few fragments (e.g. a water loss) lack the information needed to distinguish candidates. Supply the merged spectra, candidate structures (from PubChem/ChemSpider/SDF uploads), and parameter files to MetFrag2.2 — via the command-line version (http://c-ruttkies.github.io/MetFrag/), the MetFragR package (install_github("ipb-halle/MetFragR/metfRag")), or the relaunched webapp — together with scoring settings for the bond dissociation approach. The parameter files including the spectral data, the candidate, result and ranking files of the used datasets (EA, UF, EQEx, EQExPlus and HILL) are shipped as ZIP archives at http://msbi.ipb-halle.de/download/CHIN-D-15-00088/, so inputs can be validated against those archives. Stereoisomer redundancy is reduced later by InChIKey filtering (first InChIKey block), but the merged, informative peak list is the entry point for in silico bond dissociation fragmentation.

## Related tools

- **MetFrag2.2** (Receives the parsed and merged HR-MS/MS spectra and performs in silico bond dissociation fragmentation and candidate scoring.) — http://c-ruttkies.github.io/MetFrag/
- **xcms (mzClust_hclust)** (Merges replicate/variant HR-MS/MS spectra into one peak list before MetFrag scoring.)
- **metfRag (MetFragR)** (R wrapper that consumes the parsed spectra and parameter files to run MetFrag scoring within R.) — https://github.com/ipb-halle/MetFragRelaunched

## Examples

```
Rscript -e 'library(xcms); merged <- mzClust_hclust(spectra, eppm = 5e-6, eabs = 0.001)'; then feed merged peaks plus candidates SDF to MetFrag2.2 via install_github("ipb-halle/MetFragR/metfRag").
```

## Evaluation signals

- Merged spectrum contains ≥2 peaks (precursor plus at least one fragment peak); spectra failing this are dropped before MetFrag2.2 runs.
- Merging parameters match the published values: eppm = 5×10^-6 and eabs = 0.001 Da (xcms mzClust_hclust).
- The parsed peak list, parameter file, candidate list and subsequent ranking results reproduce the entries in the deposited EA/UF/EQEx/EQExPlus/HILL ZIP archives at http://msbi.ipb-halle.de/download/CHIN-D-15-00088/.
- Subsequent MetFrag2.2 run yields normalized S_C_Frag scores in [0, 1] with the top-scoring candidate at exactly 1, confirming the spectral input was parsed into the expected format.

## Limitations

- Spectra dominated by one or only a few fragments (e.g. a water loss) explainable by most candidates do not contain enough information for candidate ranking.
- Parameter files and spectral data for the EA, UF, EQEx, EQExPlus and HILL datasets are only available as external ZIP archives, not within the repository itself.
- Merging via mzClust_hclust is applied only to spectra of the same substance; cross-compound merging is not supported.
- No changelog found; no version number for MetFrag2.2 is stated beyond the project name, so exact parser/scorer version compatibility is not documented.

## Evidence

- [results] These spectra were merged using the mzClust_hclust function in xcms [53] (parameters eppm = 5×10-6 and eabs = 0.001 Da): [evidence span withheld — anchor status: not_found]
- [results] retained where they contained at least one fragment peak other than the precursor: "retained where they contained at least one fragment peak other than the precursor"
- [intro] mass spectra that are dominated by one or only a few fragments (e.g. a water loss): "mass spectra that are dominated by one or only a few fragments (e.g. a water loss) that can be explained by most of the candidates simply do not contain enough information"
- [discussion] The parameter files including the spectral data, the candidate, result and ranking files: "The parameter files including the spectral data, the candidate, result and ranking files of the used EA, UF, EQEx, EQExPlus and HILL datasets are available at"
- [intro] or accepting the upload of a structure data file (SDF): [evidence span withheld — anchor status: not_found]
