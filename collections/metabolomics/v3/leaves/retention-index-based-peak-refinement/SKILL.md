---
name: retention-index-based-peak-refinement
description: Use when annotating consensus spectra from GC × GC–MS deconvolution and
  an MSP library provides Kovats retention indices for compounds, so that cosine-similarity
  matches can be refined or disambiguated by RT-to-RI conversion.
when_to_use_negative:
- 'No retention-index information is available: leave RI = NULL and rely on cosine
  similarity alone, as in the README example with RI = NULL'
- Input is already an annotated peak table or feature matrix — RI refinement belongs
  in Module IV annotation, not downstream data integration (Module V)
- The library lacks ri fields for candidate compounds, so RI refinement cannot discriminate
  between matches
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: GcDUO
  role: R package providing duoID(), rt2ri, getRI, and library-import functions for
    RI-refined peak annotation
  repo: https://github.com/mariallr/GcDuo
- name: R
  role: Platform in which GcDUO is implemented and executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf080
    title: 'GcDUO: an open-source software for GC × GC–MS data analysis'
  source_content_hash: sha256:3bed71263c0b6d24bd5fb0019d309a3c7c5f0b30a42b12a4a14a1c2cc6ebfe5b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mariallr/GcDuo
  repo_url: https://github.com/mariallr/GcDuo
  tools:
  - GcDUO
  - R
  - duoID
  - getMSP
  - getRI
  - id_rev
  - libID
  - rt2ri
  - spectramatrix
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# retention-index-based-peak-refinement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refines GC × GC–MS peak annotations produced by cosine-similarity library matching using Kovats retention-index (RI) variability filtering, as implemented in GcDUO's duoID() via the rt2ri/getRI functions and the RIrange parameter.

## When to use

Use when annotating consensus spectra from GC × GC–MS deconvolution and an MSP library provides Kovats retention indices for compounds, so that cosine-similarity matches can be refined or disambiguated by RT-to-RI conversion.

## When NOT to use

- No retention-index information is available: leave RI = NULL and rely on cosine similarity alone, as in the README example with RI = NULL
- Input is already an annotated peak table or feature matrix — RI refinement belongs in Module IV annotation, not downstream data integration (Module V)
- The library lacks ri fields for candidate compounds, so RI refinement cannot discriminate between matches

## Inputs

- GcDUO DuoResults object containing Module III consensus spectra ($cor_spectra)
- In-house MSP spectral library (imported via getMSP/spectramatrix)
- Library dictionary with compound name, exact mass, formula, InChIKey, and ri fields
- Kovats RT–RI alkane values (alkane series with retention times and indices)

## Outputs

- DuoResults object with $id_peak annotations for each feature
- annotated_peak_table.csv (annotations written per feature)

## How to apply

After importing the in-house MSP library with getMSP/spectramatrix and assembling the library dictionary (compound name, exact mass, formula, InChIKey, and ri fields), call duoID() on the Module III consensus spectra. Supply the RT–RI alkane series via the RI argument so GcDUO can convert each feature's retention time to a Kovats retention index (rt2ri/getRI), and set RIrange (e.g. 30) to define the accepted variability between the feature's computed RI and the library compound's RI. Matches whose cosine similarity exceeds match_cutoff (e.g. 0.7) are accepted, with RI acting as an orthogonal refinement that can reject or disambiguate spectra matches that are similar but chromatographically inconsistent. The rationale is that cosine similarity alone can be ambiguous for similar EI spectra, while RI agreement adds chromatographic evidence. Resulting annotations appear in the $id_peak slot of the returned DuoResults object.

## Related tools

- **GcDUO** (R package providing duoID(), rt2ri, getRI, and library-import functions for RI-refined peak annotation) — https://github.com/mariallr/GcDuo
- **R** (Platform in which GcDUO is implemented and executed)

## Examples

```
DuoResults <- GcDuo::duoID(DuoResults, lib, match_cutoff = 0.7, RI = alkanes, RIrange = 30)
```

## Evaluation signals

- Annotations are returned only where cosine similarity exceeds match_cutoff (e.g. > 0.7); features at or below the cutoff receive no identification
- Accepted annotations satisfy the RIrange constraint (e.g. RIrange = 30) between the feature's computed Kovats RI and the library compound's ri value
- Every annotated feature in $id_peak maps to a library entry with a non-NA inchikey/ri field in the libs_dic dictionary
- Features without library compounds exceeding the cutoff return no annotation, mirroring the fruitybeer case where no compound exceeded 0.7

## Limitations

- RI refinement is only possible when the alkane series (RT–RI values) and library ri fields are available; otherwise the argument is NULL and matching is cosine-similarity only
- In the fruitybeer validation, none of the compounds in the in-house library exhibited a cosine similarity higher than 0.7, so GcDUO returned no annotation despite the spectra showing characteristic fragments
- One fruitybeer compound was missing from the in-house library, so no annotation could be produced regardless of RI agreement

## Evidence
