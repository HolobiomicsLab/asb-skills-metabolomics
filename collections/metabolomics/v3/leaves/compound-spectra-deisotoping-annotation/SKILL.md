---
name: compound-spectra-deisotoping-annotation
description: Use when an XCMS feature table from LC/MS or UPLC/ESI-QTOFMS data must
  be grouped into compound spectra, deisotoped, and annotated with ion species before
  downstream screens such as stable-isotope label difference searches.
when_to_use_negative:
- Input is already a fully annotated feature table with no ungrouped redundant features
  — grouping/deisotoping adds little value.
- Acquisitions lacking chromatographic separation or with saturated MS detector signals,
  which shift m/z and retention time centroids and truncate peak profiles.
- Single-polarity datasets when the goal specifically requires cross-polarity annotation
  verification via combinexsAnnos.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: CAMERA
  role: Core Bioconductor package implementing compound spectra extraction, isotope/adduct
    annotation, accurate mass proposal and cross-polarity validation
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Upstream feature detection/peak picking that produces the peak data fed into
    xsAnnotate
- name: R
  role: Implementation language; CAMERA is distributed via the Bioconductor repository
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:5f5d006518a87d26269dbf9330c880b3f971ff15b9b3a0dc5aee29c56a911e04
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - XCMS
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# compound-spectra-deisotoping-annotation

## Summary

Group LC/MS features into CAMERA compound spectra, deisotope and annotate adducts/fragments with accurate-mass proposals, optionally validating across opposite ionization modes. Reduces feature-list redundancy in untargeted metabolomics so downstream screening (e.g. label-difference feature pairs) operates on annotated, non-isotopic compounds.

## When to use

Use when an XCMS feature table from LC/MS or UPLC/ESI-QTOFMS data must be grouped into compound spectra, deisotoped, and annotated with ion species before downstream screens such as stable-isotope label difference searches.

## When NOT to use

- Input is already a fully annotated feature table with no ungrouped redundant features — grouping/deisotoping adds little value.
- Acquisitions lacking chromatographic separation or with saturated MS detector signals, which shift m/z and retention time centroids and truncate peak profiles.
- Single-polarity datasets when the goal specifically requires cross-polarity annotation verification via combinexsAnnos.

## Inputs

- XCMS-generated LC/MS peak/feature data (e.g. from UPLC/ESI-QTOFMS mzML or NetCDF acquisitions in positive and negative ion mode)
- Sample metadata distinguishing label-fed vs control samples (for label-aware filtering downstream)
- Optional compound/standard list with sum formulas and molar masses (e.g. MM39) for recovery validation

## Outputs

- Annotated compound spectra (CAMERA xsAnnotate/getPeaklist output) with isotope, adduct and fragment annotations
- Deisotoped/reduced peak lists (getReducedPeaklist) with proposed accurate compound masses
- Annotation-verified cross-polarity assignments linking positive and negative ion-mode compound spectra

## How to apply

Run CAMERA on XCMS peak data: construct an xsAnnotate object, apply findIsotopes (deisotoping), group features via retention time, m/z difference, intensity correlation (calcCiS/calcCaS) and peak shape similarity, then cluster with calcPC.lpc or calcPC.hcs and annotate ion species using the dynamic rule table. CAMERA does not use ad-hoc heuristics such as assuming the most intense feature is the [M+H]+ ion; instead it proposes the accurate compound mass even in highly complex data. For dual-polarity acquisitions, use combinexsAnnos to combine spectral information of opposite ion modes, keeping only common and trusted cross-polarity rules because these can override single-polarity annotations. Post-annotation cleanup typically includes eliminating isotopic peak pairs and applying an intensity threshold such as 1000 counts (e.g. getPeaklist/getReducedPeaklist).

## Related tools

- **CAMERA** (Core Bioconductor package implementing compound spectra extraction, isotope/adduct annotation, accurate mass proposal and cross-polarity validation) — https://github.com/sneumann/CAMERA
- **XCMS** (Upstream feature detection/peak picking that produces the peak data fed into xsAnnotate)
- **R** (Implementation language; CAMERA is distributed via the Bioconductor repository)

## Examples

```
R: ann <- xsAnnotate(xs); ann <- groupFWHM(ann); ann <- findIsotopes(ann); ann <- combinexsAnnos(ann)
```

## Evaluation signals

- CAMERA reported accurate-mass extraction for 89.7% (positive) and 90.3% (negative) of annotatable compounds — use this as a recovery benchmark on standards mixtures like MM39.
- Check that compound spectra group features by RT + m/z difference + intensity correlation without the [M+H]+-most-intense assumption.
- Cross-polarity annotations from combinexsAnnos should only override single-polarity results when using trusted common rules.
- Deisotoped output (getReducedPeaklist) should exclude isotopic peak pairs and features below the intensity threshold (e.g. 1000 counts).
- Unannotated spectra should be audited for the known failure modes: too few explained features or tied precedence scores.

## Limitations

- Missing molecular mass annotations occur when compound spectra contain too few explained features, tied precedence scores, or only molecular ion plus fragment ions with no further adducts.
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly unless the neutral loss is added to the rule set.
- Cross-polarity rules can override single-polarity annotations, so the rule table should only contain common and trusted combinations.
- MS detector saturation shifts m/z and retention time centroids and truncates peak profiles; sample concentration must avoid saturation.
- Feature detection is only the first step; grouping/annotation requires chromatographic separation and clean peak picking upstream.

## Evidence
