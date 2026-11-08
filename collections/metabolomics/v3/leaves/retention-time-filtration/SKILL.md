---
name: retention-time-filtration
description: Use when a CAMERA/XCMS LC/MS feature list contains early-eluting contaminants
  (e.g. under the 45 s exclusion used for Arabidopsis rosette extracts) or when you
  must bound candidate metabolites by RT and accommodate retention-time shifts caused
  by isotope labeling.
when_to_use_negative:
- When the analysis is of non-chromatographic data (e.g. direct infusion MS) where
  retention time is not defined
- When retention-time information is absent or unreliable in the input peak list (e.g.
  m/z-only centroid data), as the filter would remove or keep features arbitrarily
- When the goal is to detect very early-eluting compounds that legitimately appear
  near the solvent front rather than filter them as artifacts
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CAMERA
  role: Provides the xsAnnotate/getPeaklist workflow whose per-feature retention times
    are filtered; integrates isotope and adduct annotation after RT-based feature
    exclusion
  repo: https://github.com/sneumann/CAMERA
- name: XCMS
  role: Upstream feature detection producing the peak list with retention times that
    the RT bounds are applied to
  repo: https://github.com/sneumann/CAMERA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:f2e068f41d50a82c5e028969d34b970ee4fdaac7cd9a2793cacee60f3006a098
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - XCMS
  - findIsotopes
  - getPeaklist
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# retention-time-filtration

## Summary

A filtering skill from CAMERA (Kuhl et al. 2011) that removes spurious LC/MS features or compound spectra by retention-time bounds, such as excluding features with retention times below 45 s and restricting candidate deuterium-labeled tryptophan metabolites to a 45–600 s window, with special handling for label-induced retention-time shifts.

## When to use

Use when a CAMERA/XCMS LC/MS feature list contains early-eluting contaminants (e.g. under the 45 s exclusion used for Arabidopsis rosette extracts) or when you must bound candidate metabolites by RT and accommodate retention-time shifts caused by isotope labeling.

## When NOT to use

- When the analysis is of non-chromatographic data (e.g. direct infusion MS) where retention time is not defined
- When retention-time information is absent or unreliable in the input peak list (e.g. m/z-only centroid data), as the filter would remove or keep features arbitrarily
- When the goal is to detect very early-eluting compounds that legitimately appear near the solvent front rather than filter them as artifacts

## Inputs

- CAMERA/XCMS feature table or xsAnnotate object with per-feature retention times (seconds) from UPLC/ESI-QTOFMS data (e.g. m/z 100–1000, MicrOTOF-Q-I)
- Candidate metabolite list with known or expected retention times (e.g. Trp-related metabolites)
- Positive and negative ion-mode compound spectra (xsAnnotate objects) with aligned feature RTs

## Outputs

- Filtered feature/compound-spectrum list with early-eluting (RT < 45 s) features removed
- RT-bounded candidate set (e.g. Trp-candidates within 45–600 s)
- Paired labeled/unlabeled feature matches identified within the 8 s sliding RT window

## How to apply

Apply retention-time bounds after feature detection and grouping but before or alongside isotope/adduct annotation. For routine Arabidopsis leaf data, exclude compounds with a retention time below 45 s. For targeted metabolite searches, restrict candidates to a defined RT interval — e.g. Trp-candidates between 45 and 600 s. When searching for deuterium-labeled counterparts of unlabeled metabolites, pair features between compound spectra within a sliding retention-time window of 8 s because deuterium labeling can slightly shift retention times. Rationale: early-eluting features (below ~45 s) often derive from column void or solvent front artifacts, while label-induced shifts would cause exact co-elution matching to fail.

## Related tools

- **CAMERA** (Provides the xsAnnotate/getPeaklist workflow whose per-feature retention times are filtered; integrates isotope and adduct annotation after RT-based feature exclusion) — https://github.com/sneumann/CAMERA
- **XCMS** (Upstream feature detection producing the peak list with retention times that the RT bounds are applied to) — https://github.com/sneumann/CAMERA

## Examples

```
xsAnnotateFiltered <- xsAnnotate[getPeaklist(xsAnnotate)$rt >= 45 & getPeaklist(xsAnnotate)$rt <= 600,]
```

## Evaluation signals

- After filtering, no feature with retention time below 45 s should appear in the compound-spectrum output for Arabidopsis leaf extracts
- Trp-candidate features must all fall inside the 45–600 s RT window
- Labeled/unlabeled metabolite pairs must be found within the 8 s sliding retention-time window (i.e. |RT_labeled − RT_unlabeled| ≤ 8 s)
- The labeled counterparts are detected only in label-fed samples, whereas unlabeled features are detected in both label-fed and control samples

## Limitations

- Deuterium labeling can slightly shift retention times, so exact RT matching fails; a sliding window of 8 s is required instead of a strict equality match
- The 45 s cutoff is instrument- and chromatography-specific (Arabidopsis rosette-leaf UPLC runs); it must be re-tuned for other columns, gradients, or instruments
- Saturation of the MS detector causes shifts of m/z and retention-time centroids, so RT filtering must be applied on unsaturated data to avoid misaligned bounds

## Evidence
