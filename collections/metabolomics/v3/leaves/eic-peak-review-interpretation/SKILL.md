---
name: eic-peak-review-interpretation
description: Use when building an in-house LC-MS RT/m/z metabolite standard library
  in metScribeR and every peak that passed filtering must be manually assessed for
  inclusion in the final library.
when_to_use_negative:
- The input is already a finished feature/library table with quality labels — this
  skill only applies to peaks that passed metScribeR filtering and still require manual
  review.
- You need automated adduct conflict resolution without human inspection — that is
  handled by the crossed-adducts check (find_adduct_conflicts with >50% intensity
  competition and RT tolerance), not by peak review.
- Your library construction requires MS/MS spectral data — metScribeR focuses on MS1
  and RT data and MS2 is not used or required.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: metScribeR
  role: R package whose Shiny 'Review Results' tab presents each filtered EIC peak
    with boundary, RT, and incidental-peak annotations and collects single-click Good/Bad/Multimodal
    labels
  repo: https://github.com/ncats/metScribeR
- name: R
  role: Runtime for metScribeR; storage_object.RDS state is loaded and manipulated
    in R during review
- name: Shiny
  role: UI framework serving the review interface with drop-down navigation, arrow
    buttons, and the Good Peak / Bad Peak / Multimodal/Indeterminate controls
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:17d3d68c4fdff9a848dbdcf5fa016e943e000022003bbfa06027c271a0533cfe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ncats/metScribeR
  repo_url: https://github.com/ncats/metScribeR
  tools:
  - R
  - Shiny
  - metScribeR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# eic-peak-review-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Manual curation step in the metScribeR workflow in which each EIC peak that survived automated filtering (m/z/RT tolerance, noise cutoff ≥10,000, 7+ data points, width 0–15 s, ≤250 peaks/EIC) is visually inspected in a Shiny app and labeled Good Peak, Bad Peak, or Multimodal/Indeterminate before library export.

## When to use

Use when building an in-house LC-MS RT/m/z metabolite standard library in metScribeR and every peak that passed filtering must be manually assessed for inclusion in the final library.

## When NOT to use

- The input is already a finished feature/library table with quality labels — this skill only applies to peaks that passed metScribeR filtering and still require manual review.
- You need automated adduct conflict resolution without human inspection — that is handled by the crossed-adducts check (find_adduct_conflicts with >50% intensity competition and RT tolerance), not by peak review.
- Your library construction requires MS/MS spectral data — metScribeR focuses on MS1 and RT data and MS2 is not used or required.

## Inputs

- metScribeR storage_object.RDS containing the post-filtering peak-reviewed state (or storage_object_initial)
- EIC chromatograms extracted from positive- and negative-ESI mzML files listed in standards_df.csv (pos_mode_mzML_file_path / neg_mode_mzML_file_path)
- Filtered peak candidates that passed the density filter, data smoothing, and quality thresholds (≥7 data points, half-max width 0–15 s, intensity ≥10,000, ≤250 peaks per EIC)

## Outputs

- Manual peak quality labels (Good Peak / Bad Peak / Multimodal/Indeterminate) attached to each reviewed adduct peak
- exported_metScribeR_library.csv with identity, m/z, RT, peak quality metrics, and manual quality label for all 'good' adducts
- exported_metScribeR_library_with_metrics.csv including all 'good', 'bad', and 'multimodal/indeterminate' adducts
- One .png figure per reviewed peak stored in the Figures directory for later reference
- Updated storage_object.RDS capturing the reviewed state

## How to apply

Load a saved metScribeR storage_object.RDS (or resume from storage_object_initial to avoid recomputing the ~30 min initial pass over ~12000 mzML files) and switch to the 'Review Results' tab. Read each chromatogram using the visual conventions of the app: blue and red vertical lines mark the beginning and ending boundaries of the candidate peak, the dashed vertical line marks the assigned RT, and solid black lines mark the boundaries of incidental peaks found in the same EIC that are not under review. Navigate peaks with the drop-down menu (subsets of 50) and arrow buttons, and record a confidence label by clicking Good Peak, Bad Peak, or Multimodal/Indeterminate. The rationale is that automated quality metrics alone cannot reliably separate true standard adduct peaks from noise or co-eluting signals, so human expertise gates what enters the exported CSV; the benchmark showed metScribeR-derived RTs from this review correlate 0.99 with fully manual RT assignment at roughly 10 s of effort per standard. Save progress at any time with the Export Library csv button on the View/Export Library tab, remembering the library must be refreshed with the 'update library' button before results appear in the displayed table.

## Related tools

- **metScribeR** (R package whose Shiny 'Review Results' tab presents each filtered EIC peak with boundary, RT, and incidental-peak annotations and collects single-click Good/Bad/Multimodal labels) — https://github.com/ncats/metScribeR
- **R** (Runtime for metScribeR; storage_object.RDS state is loaded and manipulated in R during review)
- **Shiny** (UI framework serving the review interface with drop-down navigation, arrow buttons, and the Good Peak / Bad Peak / Multimodal/Indeterminate controls)

## Examples

```
library(metScribeR); setwd('<project_dir>'); runMetScribeRShinyApp()  # then review peaks on the 'Review Results' tab
```

## Evaluation signals

- Every peak that passed filtering carries exactly one manual label (Good, Bad, or Multimodal/Indeterminate) in exported_metScribeR_library_with_metrics.csv.
- exported_metScribeR_library.csv contains only 'good' adducts; no 'bad' or 'multimodal/indeterminate' entries leak into the primary library.
- A .png figure exists in the Figures directory for each peak reviewed manually, matching the set of labeled peaks.
- RT and m/z values for reviewed good peaks are consistent with the dashed RT line and peak boundaries shown during review, and the same user-defined m/z and RT tolerances used for EIC creation are respected.
- On the IROA benchmark, metScribeR-derived RTs from this review should correlate ~0.99 with fully manual RT assignment at ~10 s per standard of effort.

## Limitations

- Review is single-click and therefore subjective; 'Multimodal/Indeterminate' peaks are excluded from the primary library, so borderline shapes may be lost.
- Review quality depends on upstream choices (density filter, data smoothing, noise level, m/z/RT tolerances) tuned on the Find Peaks tab; poor settings surface confusing chromatograms.
- For large submissions the initial computation (~30 min for 12000 mzML files) precedes review, though storage_object_initial avoids repeating it.
- Only peaks passing the automated filters are offered for review — peaks eliminated by filtering cannot be recovered in this step.

## Evidence
