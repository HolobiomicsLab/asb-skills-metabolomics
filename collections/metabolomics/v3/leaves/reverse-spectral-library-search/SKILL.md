---
name: reverse-spectral-library-search
description: Use when you have MS1-only LC-MS or imaging MS data (mzML/mzXML or imzML)
  that lack DDA MS/MS files, and you want level 2/3 structural metabolite annotations
  by leveraging in/post-source fragment ions against an indexed MS/MS library.
when_to_use_negative:
- The input is already a feature table without chromatographic or spatial correlation
  information to build pseudo MS/MS spectra
- The goal is molecular formula assignment (MSI level 4) only, as METASPACE-style
  target-decoy FDR approaches address that rather than library-based reverse search
- The query spectra are true DDA MS/MS at matching collision energies — for those,
  conventional forward MS/MS search (e.g., the gnps.pkl library via ms1_id's MS/MS
  annotation) applies instead of peak-scaled reverse search of in-source fragments
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ms1-id
  role: Performs the precursor-tolerant reverse spectral search of pseudo MS/MS spectra
    against peak-scale-adjusted indexed MS/MS libraries (annotate, lcms, msi, index
    subcommands)
  repo: https://github.com/Philipbear/ms1_id
- name: MassCube
  role: Backend used by ms1-id to extract metabolic features, assign isotope peaks
    (which are excluded from pseudo MS/MS generation), and support pseudo MS/MS generation
- name: IDSL.CSA
  role: Prior-art approach for matching pseudo MS/MS spectra; relevant as the conceptual
    precedent the article's reverse matching extends
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:b8877388c5c4d80a1a4242f36256b27d8c18b84e7af32202ad427f00c0dfed3a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/Philipbear/ms1_id
  tools:
  - IDSL.CSA
  - MassCube
  - ms1-id
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# reverse-spectral-library-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate full-scan MS1 data by searching pseudo MS/MS spectra (built from correlated in-source fragments) against reference MS/MS libraries using precursor-tolerant reverse spectral matching with mass-dependent peak-intensity scaling, as implemented in the ms1-id package.

## When to use

Use when you have MS1-only LC-MS or imaging MS data (mzML/mzXML or imzML) that lack DDA MS/MS files, and you want level 2/3 structural metabolite annotations by leveraging in/post-source fragment ions against an indexed MS/MS library.

## When NOT to use

- The input is already a feature table without chromatographic or spatial correlation information to build pseudo MS/MS spectra
- The goal is molecular formula assignment (MSI level 4) only, as METASPACE-style target-decoy FDR approaches address that rather than library-based reverse search
- The query spectra are true DDA MS/MS at matching collision energies — for those, conventional forward MS/MS search (e.g., the gnps.pkl library via ms1_id's MS/MS annotation) applies instead of peak-scaled reverse search of in-source fragments

## Inputs

- Pseudo MS/MS spectra in mgf format
- LC-MS raw data as mzML or mzXML files
- Imaging MS data as imzML (with ibd) files
- Indexed MS/MS spectral library (e.g., GNPS library indexed with peak_scale_k=10, peak_intensity_power=0.5)

## Outputs

- Per-spectrum annotation tsv files with compound matches and scores
- Aligned LC-MS feature table with MS1 and MS/MS annotations (aligned_feature_table.tsv)
- Dereplicated imaging annotations (ms1_id_annotations_derep.tsv)
- Level 2/3 structural annotations linked to metabolic features (top hit per precursor m/z)

## How to apply

First generate pseudo MS/MS (composite) spectra from in-source fragments by correlating metabolic features within ±1.5 s retention time, requiring at least 4 consecutive shared MS1 scans, a Pearson correlation ≥ 0.80, and excluding isotope peaks assigned by MassCube. Build or download an indexed reference library (e.g., the GNPS MS/MS library downloaded July 17, 2024) with mass-dependent peak scaling (scaling factor k = 10) and square-root-transformed (peak_intensity_power 0.5) intensities so that low-energy in-source fragments can match references collected at medium to high collision energies. Then run the precursor-tolerant (open) reverse spectral search, which accommodates multiple ion forms simultaneously, applying LC-MS cutoffs of minimum reverse cosine score 0.7, minimum 3 matched peaks, and minimum spectral usage 0.20 (use 0.05 spectral usage for MS imaging). Link accepted annotations back to features via retention time and precursor mass, keeping the top 1 hit per unique precursor m/z.

## Related tools

- **ms1-id** (Performs the precursor-tolerant reverse spectral search of pseudo MS/MS spectra against peak-scale-adjusted indexed MS/MS libraries (annotate, lcms, msi, index subcommands)) — https://github.com/Philipbear/ms1_id
- **MassCube** (Backend used by ms1-id to extract metabolic features, assign isotope peaks (which are excluded from pseudo MS/MS generation), and support pseudo MS/MS generation)
- **IDSL.CSA** (Prior-art approach for matching pseudo MS/MS spectra; relevant as the conceptual precedent the article's reverse matching extends)

## Examples

```
ms1_id annotate --input_file pseudo_msms.mgf --libs db/gnps.pkl db/gnps_k10.pkl --min_score 0.7 --min_matched_peak 3
```

## Evaluation signals

- All reported hits meet the LC-MS cutoffs: reverse cosine score ≥ 0.7, ≥ 3 matched peaks, spectral usage ≥ 0.20 (≥ 0.05 for MS imaging)
- Only one annotation is kept per unique precursor m/z (top 1 hit)
- Reproduces reference benchmark counts within ±10%: 567, 306, 484, 511, and 604 spectral matches for NIST fecal DDA MS/MS (42 eV), MS1 DDA, and MS1 0/10/20 eV modes; 3010/293/227/636 unique metabolites for the IBD HILIC+/HILIC−/C8+/C18− modes totaling 3802
- Reference libraries were indexed with peak_scale_k=10 and peak_intensity_power=0.5; scaled search of bile-acid/drug standards yields more scores > 0.7 than unscaled search
- Query features whose m/z exceeds the feature precursor m/z are not counted in modified cosine-based contexts (ions larger than the feature m/z must be removed for modified cosine calculations)

## Limitations

- Low-energy in-source fragments mismatch most public reference spectra acquired at medium to high collision energies; peak scaling (k = 10 with square-root transformation) is required to compensate but may still miss compounds
- Requires sufficient co-eluting shared scans — at least 4 consecutive shared MS1 scans and Pearson correlation ≥ 0.80 — so low-abundance or poorly resolved features yield no pseudo MS/MS
- Annotations are level 2/3 confidence; validation relies on library-version drift tolerance (e.g., GNPS July 17, 2024 version), and count reproducibility may drift with library updates

## Evidence

- [readme] two indexed libraries are searched against, and the result tsv files will be saved in the same directory as the input file: [evidence span withheld — anchor status: external_doc]
