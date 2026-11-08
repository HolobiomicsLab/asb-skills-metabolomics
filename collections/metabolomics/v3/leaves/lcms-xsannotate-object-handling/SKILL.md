---
name: lcms-xsannotate-object-handling
description: Use when LC/MS peak tables from positive and negative ESI ion-mode runs
  must be annotated jointly, or when single-polarity CAMERA annotations need cross-mode
  verification to recover molecular mass assignments.
when_to_use_negative:
- Input is already an aggregated feature/annotation table without the underlying xsAnnotate
  objects — combinexsAnnos requires the paired xsAnnotate objects, not flat peak lists.
- Only one ion polarity is available; the cross-mode verification approach explicitly
  combines spectral information from opposite ion modes and cannot apply to single-polarity
  data.
- The goal is de novo compound identification or elemental composition calculation
  rather than ion-species annotation and molecular mass assignment — CAMERA's rule-table
  approach is designed for the latter.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: CAMERA
  role: Bioconductor package providing xsAnnotate objects and the combinexsAnnos function
    for cross-polarity annotation verification
  repo: https://github.com/sneumann/CAMERA
- name: R
  role: Implementation language; CAMERA is installed from the Bioconductor repository
- name: XCMS
  role: Upstream feature detection/processing framework feeding peaklists into CAMERA
    xsAnnotate objects
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:6ce3cacd6fda5549af8b997cb5ef06e2fee495f75fb10b23a6ffff169eaf6b63
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - XCMS
  - combinexsAnnos
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# lcms-xsannotate-object-handling

## Summary

Skill for creating, loading, and combining CAMERA xsAnnotate objects — including cross-polarity verification of ion annotations via combinexsAnnos, which matches compound spectra acquired in opposite ion modes against a trusted rule table to raise annotation rates beyond single-polarity runs.

## When to use

Use when LC/MS peak tables from positive and negative ESI ion-mode runs must be annotated jointly, or when single-polarity CAMERA annotations need cross-mode verification to recover molecular mass assignments.

## When NOT to use

- Input is already an aggregated feature/annotation table without the underlying xsAnnotate objects — combinexsAnnos requires the paired xsAnnotate objects, not flat peak lists.
- Only one ion polarity is available; the cross-mode verification approach explicitly combines spectral information from opposite ion modes and cannot apply to single-polarity data.
- The goal is de novo compound identification or elemental composition calculation rather than ion-species annotation and molecular mass assignment — CAMERA's rule-table approach is designed for the latter.

## Inputs

- CAMERA xsAnnotate object for the positive ion-mode dataset (upstream single-polarity annotation run)
- CAMERA xsAnnotate object for the negative ion-mode dataset (upstream single-polarity annotation run)
- Cross-polarity rule table containing common and trusted ion-species combinations

## Outputs

- Cross-polarity-verified xsAnnotate annotation object with updated molecular mass assignments
- annotation_rates_comparison.tsv (columns: mode, number of annotated spectra, annotation rate)

## How to apply

First run CAMERA's annotation pipeline (isotope/adduct grouping via the dynamic rule table) separately on each ion mode to produce an xsAnnotate object per polarity. Load both xsAnnotate objects, then call combinexsAnnos with the paired objects to match compound spectra across opposite ion modes against the cross-polarity rule table, keeping only common and trusted combinations — the article stresses the cross-polarity rule table must be restricted because these rules can override single-polarity annotations. Compare annotated compound spectra counts before (single-polarity) and after cross-polarity verification, compute annotation rates for the combined approach relative to each single mode (single-mode baselines in the paper were 89.7% positive and 90.3% negative), and assemble counts/rates into a tab-separated table such as annotation_rates_comparison.tsv with columns for mode, number of annotated spectra, and annotation rate. Judge success when combined-mode annotation equals or exceeds both single-mode rates.

## Related tools

- **CAMERA** (Bioconductor package providing xsAnnotate objects and the combinexsAnnos function for cross-polarity annotation verification) — https://github.com/sneumann/CAMERA
- **R** (Implementation language; CAMERA is installed from the Bioconductor repository)
- **XCMS** (Upstream feature detection/processing framework feeding peaklists into CAMERA xsAnnotate objects)

## Examples

```
R> ann.pos <- loadRDS("xsannot_pos.rds"); ann.neg <- loadRDS("xsannot_neg.rds"); ann.comb <- combinexsAnnos(ann.pos, ann.neg)
```

## Evaluation signals

- Combined cross-polarity run annotates all compounds that each single polarity annotated (paper: all annotatable compounds correct after combining modes), plus any extra recoveries.
- Annotation rate of the combined approach is >= the single-mode baselines (89.7% positive, 90.3% negative for the MM39 evaluation).
- annotation_rates_comparison.tsv parses as TSV with exactly the columns mode, number of annotated spectra, and annotation rate.
- Only common and trusted cross-polarity rule combinations were used — check that the rule table was filtered, since untrusted rules may override single-polarity annotations.
- Annotation-rate comparison is reproducible from the two xsAnnotate object counts (before vs after combinexsAnnos).

## Limitations

- Missing annotations persist when compound spectra contain too few explained features, several hypotheses tie on precedence scores, or the spectrum contains only the molecular ion and fragment ions with no further adducts; in the last case direct annotation fails unless the neutral loss is added to the rule set.
- Cross-polarity rules can override single-polarity annotations, so the rule table must only contain common and trusted combinations.
- Sample concentration causing MS detector saturation shifts m/z and retention time centroids and truncates peak profiles, degrading annotation quality.

## Evidence
