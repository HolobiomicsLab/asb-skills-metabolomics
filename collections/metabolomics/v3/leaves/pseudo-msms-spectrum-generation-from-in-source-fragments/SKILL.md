---
name: pseudo-msms-spectrum-generation-from-in-source-fragments
description: Use when input is MS1-only full-scan data (mzML, mzXML, or imzML) containing
  in-source fragments and the goal is structural (level 2/3) annotation without real
  MS/MS spectra — e.g. reanalysis of public repository LC-MS or MS imaging files.
when_to_use_negative:
- The data already contain genuine MS/MS scans — standard MS/MS library search (e.g.
  the ms2_id branch of the ms1-id lcms workflow) is the appropriate route
- Data were acquired with deliberate extra in-source fragmentation (isCID) designs
  — enhanced in-source fragmentation methods such as IDSL.CSA, eISA, or EISA-EXPOSOME
  are designed for that scenario instead
- The annotation target is lipid-only, where specialized tools like rMSIfragment or
  HERMES (in-source fragment annotation) may be preferred
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: ms1-id
  role: Python package implementing correlation-based feature clustering, pseudo MS/MS
    generation, peak scaling, and reverse spectral matching for LC-MS (mzML/mzXML),
    imaging (imzML), and mgf inputs
  repo: https://github.com/Philipbear/ms1_id
- name: MassCube
  role: Backend (v1.0.11) used by ms1-id for feature extraction, peak-peak correlation
    windows, and isotope peak assignment whose isotope calls are excluded from pseudo
    MS/MS generation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:f040e9c420c49b4d1a91c35ec5c9866f3dc12d65ba3bd61cf8ac2ad961b90568
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
  - MassCube
  - ms1-id
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pseudo-msms-spectrum-generation-from-in-source-fragments

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates pseudo MS/MS (composite) spectra from MS1-only full-scan LC-MS or imaging MS data by clustering in/post-source fragment ions with correlation analysis, enabling level 2/3 structural annotation via reverse spectral search against reference MS/MS libraries. This addresses the gap that over 40% of public LC-MS files contain only MS1 scans.

## When to use

Use when input is MS1-only full-scan data (mzML, mzXML, or imzML) containing in-source fragments and the goal is structural (level 2/3) annotation without real MS/MS spectra — e.g. reanalysis of public repository LC-MS or MS imaging files.

## When NOT to use

- The data already contain genuine MS/MS scans — standard MS/MS library search (e.g. the ms2_id branch of the ms1-id lcms workflow) is the appropriate route
- Data were acquired with deliberate extra in-source fragmentation (isCID) designs — enhanced in-source fragmentation methods such as IDSL.CSA, eISA, or EISA-EXPOSOME are designed for that scenario instead
- The annotation target is lipid-only, where specialized tools like rMSIfragment or HERMES (in-source fragment annotation) may be preferred

## Inputs

- Full-scan MS1-only LC-MS mzML or mzXML files (e.g. the 546-sample IBD stool dataset PR000639 from Metabolomics Workbench)
- MS imaging imzML (+ ibd) files
- Pseudo MS/MS spectra already generated in mgf format
- Indexed reference MS/MS spectral library (e.g. indexed GNPS MS/MS library, July 17 2024 version; build with peak_scale_k=10, peak_intensity_power=0.5)

## Outputs

- Pseudo MS/MS (composite) spectra per metabolic feature in mgf format
- Annotated feature table (e.g. aligned_feature_table.tsv) with level 2/3 structural annotations linked to features by retention time and precursor mass
- Unique InChIKey annotation counts per mode (e.g. 3010 HILIC pos, 293 HILIC neg, 227 C8 pos, 636 C18 neg for the IBD dataset; 3802 total)

## How to apply

Cluster metabolic features by correlating extracted ion chromatograms (XICs) within a ±1.5 s retention-time window, requiring at least 4 consecutive shared MS1 scans between two peaks and a Pearson correlation ≥ 0.80 (for imaging data, require at least 50 shared pixels with non-zero intensities between ion images). Exclude peaks identified as isotopes by the MassCube backend (v1.0.11) from the pseudo MS/MS spectrum. The resulting composite spectra treat the precursor plus its correlated in/post-source fragments as one deconvolved MS/MS-like spectrum, exportable as an mgf file for downstream precursor-tolerant (open search) reverse spectral matching. Because in-source fragments are low-energy while most public reference spectra (e.g. GNPS library) were collected at medium-to-high collision energies, apply square-root-transformed, mass-dependent peak scaling (scaling factor k = 10) to the reference spectra before matching, using cutoffs of minimum reverse cosine score 0.7, minimum 3 matched peaks, and minimum spectral usage 0.20 (0.05 for imaging). Keep the top 1 hit per unique precursor m/z and link annotations back to features via retention time and precursor mass.

## Related tools

- **ms1-id** (Python package implementing correlation-based feature clustering, pseudo MS/MS generation, peak scaling, and reverse spectral matching for LC-MS (mzML/mzXML), imaging (imzML), and mgf inputs) — https://github.com/Philipbear/ms1_id
- **MassCube** (Backend (v1.0.11) used by ms1-id for feature extraction, peak-peak correlation windows, and isotope peak assignment whose isotope calls are excluded from pseudo MS/MS generation)

## Examples

```
ms1_id lcms --project_dir lc_ms --sample_dir data --ms1_id_libs db/gnps.pkl db/gnps_k10.pkl --ms2_id_lib db/gnps.pkl
```

## Evaluation signals

- Per-mode unique InChIKey counts for the IBD stool dataset should match 3010 (HILIC positive), 293 (HILIC negative), 227 (C8 positive), 636 (C18 negative), totaling 3802 unique level 2/3 metabolites (save as ibd_ms1_annotation_counts.csv)
- Every retained annotation passes the LC-MS cutoffs: reverse cosine score ≥ 0.7, ≥ 3 matched peaks, spectral usage ≥ 0.20, top 1 hit per unique precursor m/z
- Feature pairs used in pseudo spectra share a Pearson correlation ≥ 0.80 within a ±1.5 s RT window with ≥ 4 consecutive shared MS1 scans
- No MassCube-assigned isotope peaks appear as fragments in the generated pseudo MS/MS spectra
- Reference spectra were square-root-transformed with mass-dependent scaling factor k = 10 before search (verify library was built with --peak_scale_k 10 --peak_intensity_power 0.5)

## Limitations

- Runtime can exceed 10 min per LC-MS file if --mass_detect_int_tol (default 2e5 for Orbitraps, 5e2 for QTOFs) is too strict; increase it to speed up processing
- For imaging data, fragments with m/z < 100 are not matched (use the gnps_minmz100 library), and spatial correlation requires at least 50 shared non-zero pixels
- Low-energy in-source fragments mismatch reference spectra acquired at medium-to-high collision energies unless peak scaling is applied; only ~<1% of MoNA reference spectra were acquired at ≤5 eV collision energies
- The paper's rendering of the matching-score equation and the mass-dependent peak-scaling equation is blank in the text, so the exact formulas must be taken from the source code (ms1-id GitHub/Zenodo)

## Evidence

- [readme] annotate pseudo MS/MS spectra: **mgf** files: [evidence span withheld — anchor status: external_doc]
