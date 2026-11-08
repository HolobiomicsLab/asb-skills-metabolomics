---
name: reverse-cosine-spectrum-search
description: Use when pseudo MS/MS spectra reconstructed from MS1-only data must be
  annotated against a reference MS/MS library but contain fragment peaks not present
  in the reference spectra (in-source fragments of adducts, multimers, co-eluting
  species).
when_to_use_negative:
- When you have true MS/MS spectra with a single predefined precursor — the reverse
  search discards unmatched query peaks and is designed for composite/pseudo MS/MS
  spectra containing multiple ion forms; for MS2 data the modified cosine in forward
  mode is the appropriate comparator
- When reference spectra were collected at medium-to-high collision energies without
  applying the k=10 peak intensity scaling — the article reports that without peak
  scaling, ground-truth matches scoring >0.7 are substantially fewer
- For molecular-network edge construction, which instead requires modified cosine
  >= 0.8 and matched peaks >= 6 and removes all ions larger than the feature m/z from
  the pseudo MS/MS
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ms1_id
  role: Python package implementing the precursor-tolerant reverse cosine search against
    indexed MS/MS libraries (annotate subcommand for MGF pseudo MS/MS input)
  repo: https://github.com/Philipbear/ms1_id
- name: FlashEntropySearch
  role: Underlying flash entropy search framework modified for reverse cosine scoring
  repo: https://github.com/YuanyueLi/FlashEntropySearch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.10.14.618269
    title: 'Structural annotation of full-scan MS data: A unified solution for LC-MS
      and MS imaging analyses'
  source_content_hash: sha256:1e402026290f2f6210541b95d41f43d8524d274091795516618da48d7facae3d
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
  - FlashEntropySearch
  - ms1-id
  - ms1_id
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# reverse-cosine-spectrum-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Asymmetric precursor-tolerant reverse spectral search that treats a reference MS/MS spectrum as a template, discarding unmatched query peaks so pseudo MS/MS spectra containing extra ion forms (adducts, multimers, co-eluting contaminants) can be matched against indexed MS/MS libraries.

## When to use

Use when pseudo MS/MS spectra reconstructed from MS1-only data must be annotated against a reference MS/MS library but contain fragment peaks not present in the reference spectra (in-source fragments of adducts, multimers, co-eluting species).

## When NOT to use

- When you have true MS/MS spectra with a single predefined precursor — the reverse search discards unmatched query peaks and is designed for composite/pseudo MS/MS spectra containing multiple ion forms; for MS2 data the modified cosine in forward mode is the appropriate comparator
- When reference spectra were collected at medium-to-high collision energies without applying the k=10 peak intensity scaling — the article reports that without peak scaling, ground-truth matches scoring >0.7 are substantially fewer
- For molecular-network edge construction, which instead requires modified cosine >= 0.8 and matched peaks >= 6 and removes all ions larger than the feature m/z from the pseudo MS/MS

## Inputs

- Pseudo MS/MS spectra in MGF format reconstructed from MS1-only data (correlation of extracted ion chromatograms or ion images)
- Indexed MS/MS reference library (e.g. indexed GNPS library July 17, 2024 version; gnps.pkl and gnps_k10.pkl with peak_scale_k=10)
- Metabolic feature table (LC-MS) or ion images (MS imaging) for linking annotations to features

## Outputs

- ms1_annotation_table.csv with columns for compound name, precursor m/z, reverse cosine score, matched peak count, and spectral usage
- Top-1 annotation per unique precursor m/z per pseudo MS/MS spectrum
- Matched peak count and spectral usage metrics per annotation

## How to apply

First apply the mass-dependent peak intensity scaling to each reference MS/MS spectrum: scale fragment intensity by its m/z ratio alpha_i (fragment m/z over precursor m/z) raised to a scaling factor k=10, then apply a square-root intensity power transformation to both reference and query spectra (peak_intensity_power 0.5 in the ms1_id indexer). Load an indexed GNPS MS/MS reference library (July 17, 2024 version indexed via 'ms1_id index --peak_scale_k 10 --peak_intensity_power 0.5') and run the reverse cosine search — a flash-entropy framework modified for reverse cosine — in an open-search manner that considers every ion as a potential precursor, requiring that the precursor ion of each matching reference hit appears among the m/z values of the query pseudo MS/MS spectrum. After the search, retain only the top-1 hit for each unique precursor m/z among all candidate annotations, then filter with LC-MS cutoffs of minimum reverse cosine score 0.7, minimum matched peaks 3, and minimum spectral usage 0.20 (for MS imaging use the same score and matched-peak cutoffs but minimum spectral usage 0.05). The rationale for reverse matching is that a single predefined precursor cannot be assumed: pseudo MS/MS spectra contain multiple ion forms of the same molecule, and the reference spectrum acts as the template while extra query peaks are discarded during score calculation. Finally link each retained annotation to a metabolic feature using the retention time of the pseudo MS/MS spectrum and the annotation's precursor mass.

## Related tools

- **ms1_id** (Python package implementing the precursor-tolerant reverse cosine search against indexed MS/MS libraries (annotate subcommand for MGF pseudo MS/MS input)) — https://github.com/Philipbear/ms1_id
- **FlashEntropySearch** (Underlying flash entropy search framework modified for reverse cosine scoring) — https://github.com/YuanyueLi/FlashEntropySearch

## Examples

```
ms1_id annotate --input_file pseudo_msms.mgf --libs db/gnps.pkl db/gnps_k10.pkl --min_score 0.7 --min_matched_peak 3
```

## Evaluation signals

- All reported annotations must satisfy the LC-MS cutoffs: reverse cosine score >= 0.7, matched peaks >= 3, spectral usage >= 0.20 (or spectral usage >= 0.05 for MS imaging)
- For each unique precursor m/z in a pseudo MS/MS spectrum, only the top-1 annotation is retained — no multiple hits per precursor m/z
- The precursor ion of each matching reference hit must appear among the m/z values of the query pseudo MS/MS spectrum (precursor-tolerant requirement)
- Output annotation table (ms1_annotation_table.csv) must contain compound name, precursor m/z, reverse cosine score, matched peak count, and spectral usage columns
- In the ground-truth standard pool validation, peak scaling (k=10) should yield more matches scoring >0.7 for the 14 known molecules than the unscaled comparison

## Limitations

- The reverse search requires an indexed library built with the same peak_scale_k (k=10) and square-root intensity power (0.5) as the query spectra — mismatched scaling parameters will degrade matching
- Only ~1% of reference spectra in MoNA/NIST were acquired under collision energies <= 5 eV, so low-energy in-source fragments may match poorly even with scaling
- Minimum of 3 matched peaks means spectra with fewer than 3 matching fragments cannot pass the cutoff even with a high cosine score
- For MS imaging data, fragments with m/z < 100 are excluded from the library (gnps_minmz100.pkl) because they are not usually present in MS imaging data

## Evidence

- [methods] The method employs a reverse spectral search in which the reference spectrum acts as a template: the search does not assume a single predefined precursor (considering every ion as a potential precursor), and unmatched peaks in the query pseudo MS/MS spectrum are discarded when calculating matching scores.: [evidence span withheld — anchor status: not_found]
- [readme] If you have pseudo MS/MS spectra in mgf format, you can directly annotate them: [evidence span withheld — anchor status: external_doc]
