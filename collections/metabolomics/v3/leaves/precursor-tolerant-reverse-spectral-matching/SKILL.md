---
name: precursor-tolerant-reverse-spectral-matching
description: Use when annotating full-scan MS data (MS1-only mzML/mzXML/imzML or pseudo
  MS/MS mgf) against a reference MS/MS library, especially when the true precursor
  m/z is unknown or multiple adducts/multimers must be matched at once.
when_to_use_negative:
- The data already contain true MS/MS scans — use conventional MS/MS library search
  (forward cosine/modified cosine) instead of pseudo MS/MS reverse matching
- You need strict isomeric-level annotation with a confirmed precursor mass rather
  than precursor-tolerant level 2/3 annotation of MS1-only features
- For molecular networking, where modified cosine (>= 0.8, >= 6 matched peaks) with
  ions larger than the feature m/z removed is required instead
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: ms1-id
  role: Python package that generates pseudo MS/MS spectra from MS1 features and performs
    precursor-tolerant reverse spectral matching against indexed MS/MS libraries (ms1_id
    annotate / lcms / msi commands)
  repo: https://github.com/Philipbear/ms1_id
- name: GNPS library
  role: Reference MS/MS spectral library indexed with peak_scale_k 10 and peak_intensity_power
    0.5 as the reverse-matching target (gnps.pkl, gnps_k10.pkl, gnps_minmz100 variants)
  repo: https://github.com/Philipbear/ms1_id
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:744f3e59a9c709782886d206c8417b9ba3aa6cd019917f8e24b1b84ecd88a3bd
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
  - GNPS library
  - METASPACE
  - ms1-id
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# precursor-tolerant-reverse-spectral-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reverse spectral matching of deconvolved MS1 pseudo MS/MS spectra (from in-source fragments) against peak scale-adjusted MS/MS libraries such as GNPS, using a precursor-tolerant (open) search that accommodates multiple ion forms simultaneously to achieve level 2/3 structural annotation of MS1-only LC-MS and MS imaging data.

## When to use

Use when annotating full-scan MS data (MS1-only mzML/mzXML/imzML or pseudo MS/MS mgf) against a reference MS/MS library, especially when the true precursor m/z is unknown or multiple adducts/multimers must be matched at once.

## When NOT to use

- The data already contain true MS/MS scans — use conventional MS/MS library search (forward cosine/modified cosine) instead of pseudo MS/MS reverse matching
- You need strict isomeric-level annotation with a confirmed precursor mass rather than precursor-tolerant level 2/3 annotation of MS1-only features
- For molecular networking, where modified cosine (>= 0.8, >= 6 matched peaks) with ions larger than the feature m/z removed is required instead

## Inputs

- Pseudo MS/MS spectra in mgf format generated from MS1 in-source fragments
- Full-scan LC-MS files (mzML or mzXML)
- MS imaging files (imzML with ibd)
- Indexed MS/MS reference library (e.g., indexed GNPS library downloaded July 17, 2024; gnps_minmz100 variant for imaging data with fragments below m/z 100 removed)

## Outputs

- Annotation result table (tsv, e.g., aligned_feature_table.tsv for LC-MS or ms1_id_annotations_derep.tsv for imaging) with library hits, reverse cosine scores, matched peaks, and ion forms
- Level 2/3 structural annotations for MS1-only features

## How to apply

First generate pseudo MS/MS (composite) spectra by clustering MS1 features via correlation of XICs (LC-MS) or ion images (imaging MS), excluding isotope peaks flagged by MassCube. Then perform precursor-tolerant reverse spectral matching: the query pseudo MS/MS is compared against peak scale-adjusted reference spectra (indexed GNPS library with peak_scale_k 10 and peak_intensity_power 0.5), so fragments above the feature m/z are tolerated and multiple ion forms (adducts, multimers) match in one search. Apply annotation cutoffs appropriate to the platform: minimum score 0.7, minimum matched peaks 3, and minimum spectral usage 0.20 for LC-MS versus 0.05 for MS imaging data. Reverse matching is used because the pseudo MS/MS contains precursor-related and unremoved low-mass ions, so reference peaks absent from the query are not penalized the way forward cosine would.

## Related tools

- **ms1-id** (Python package that generates pseudo MS/MS spectra from MS1 features and performs precursor-tolerant reverse spectral matching against indexed MS/MS libraries (ms1_id annotate / lcms / msi commands)) — https://github.com/Philipbear/ms1_id
- **GNPS library** (Reference MS/MS spectral library indexed with peak_scale_k 10 and peak_intensity_power 0.5 as the reverse-matching target (gnps.pkl, gnps_k10.pkl, gnps_minmz100 variants)) — https://github.com/Philipbear/ms1_id

## Examples

```
ms1_id annotate --input_file pseudo_msms.mgf --libs db/gnps.pkl db/gnps_k10.pkl --min_score 0.7 --min_matched_peak 3
```

## Evaluation signals

- Annotations meet the platform-specific cutoffs: minimum score 0.7, minimum matched peaks 3, minimum spectral usage 0.20 (LC-MS) or 0.05 (MS imaging)
- In the spiked-standards benchmark (mouse liver, seven spotted standards: ATP, ADP, adenosine, guanine, glutamic acid, glutamine, glutathione), 5 of 7 standards receive annotations; guanine (m/z 150.0421) and glutamic acid (m/z 146.0459) correctly fail due to insufficient in-source fragments above m/z 100
- The peak-scaled condition (k=10, square-root transformed) yields more reverse cosine scores > 0.7 for ground-truth compounds than the non-scaled condition, as shown for the 14 known molecules in bile acid and drug standard pools
- Matched ion forms should be consistent with the precursor-tolerant search (adducts/multimers accommodated), not restricted to a single fixed precursor m/z
- For MALDI imaging runs, only the gnps_minmz100 library variant should be used since imaging data lack fragments below m/z 100

## Limitations

- Standards that do not produce sufficient in-source fragment ions within the collected m/z range (e.g., guanine and glutamic acid, with few fragments above m/z 100 in MS imaging data) cannot be confidently annotated
- In-source fragments match reference spectra acquired at low collision energy, while most public reference spectra are medium-to-high energy; peak intensity scaling mitigates but does not eliminate this mismatch (<1% of MoNA reference spectra were acquired at <= 5 eV)
- The article's rendered score and scaling equations are missing from the text, so exact score computation must be recovered from the ms1-id source code
- Runtime can exceed 10 min per LC-MS file for poor-quality data, requiring adjustment of --mass_detect_int_tol (default 2e5 Orbitrap, 5e2 QTOF)

## Evidence

- [readme] For MS imaging data (fragments with mz < 100 are removed, as they are not usually included in MS imaging data): [evidence span withheld — anchor status: external_doc]
