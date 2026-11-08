---
name: adduct-rule-table-matching
description: Use when grouping has produced CAMERA compound spectra from LC/MS peak
  lists (positive and/or negative ESI mode) and you need to assign ion-species annotations
  and propose accurate compound masses from adduct/isotope rules.
when_to_use_negative:
- You only have a peak/feature table with no chromatographic peak grouping — CAMERA
  needs compound spectra (grouped isotope/adduct peaks) as input, not a flat feature
  list.
- The compound spectrum contains only the molecular ion and fragment ions with no
  further adducts — annotation is not possible unless the neutral loss is added to
  the rule set.
- The sample concentration saturates the MS detector, since saturation causes shifts
  of m/z and retention time centroids and truncated peak profiles that corrupt rule-table
  matching.
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: CAMERA
  role: Bioconductor package providing the rule-table-driven ion-species annotation,
    findIsotopesWithValidation, and combinexsAnnos cross-polarity verification
  repo: https://github.com/sneumann/CAMERA
- name: R
  role: Implementation environment; CAMERA is installed from the Bioconductor repository
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/ac202450g
    title: 'CAMERA: An Integrated Strategy for Compound Spectra Extraction and Annotation
      of Liquid Chromatography/Mass Spectrometry Data Sets'
  source_content_hash: sha256:d47486cdbd7c9abdb57d6d4d43a7e82b36cdefe04b477db7cc3b7e7a3619b578
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sneumann/CAMERA
  tools:
  - CAMERA
  - R
  - combinexsAnnos
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# adduct-rule-table-matching

## Summary

Annotate quasi-molecular ion species ([M+H]+, adducts, isotopes) in LC/MS compound spectra by matching peak m/z differences against CAMERA's dynamic rule table, instead of ad-hoc heuristics like assuming the most intense peak is [M+H]+. Cross-polarity matching (combinexsAnnos) can further verify annotations using spectra acquired in opposite ion modes.

## When to use

Use when grouping has produced CAMERA compound spectra from LC/MS peak lists (positive and/or negative ESI mode) and you need to assign ion-species annotations and propose accurate compound masses from adduct/isotope rules.

## When NOT to use

- You only have a peak/feature table with no chromatographic peak grouping — CAMERA needs compound spectra (grouped isotope/adduct peaks) as input, not a flat feature list.
- The compound spectrum contains only the molecular ion and fragment ions with no further adducts — annotation is not possible unless the neutral loss is added to the rule set.
- The sample concentration saturates the MS detector, since saturation causes shifts of m/z and retention time centroids and truncated peak profiles that corrupt rule-table matching.

## Inputs

- CAMERA xsAnnotate object for the positive ion-mode dataset (from upstream single-polarity annotation)
- CAMERA xsAnnotate object for the negative ion-mode dataset
- CAMERA dynamic adduct rule table (cross-polarity rules restricted to common and trusted combinations)
- LC/ESI-MS peak list from feature detection (e.g. UPLC/ESI-QTOFMS data, m/z 100–1000)

## Outputs

- xsAnnotate object with annotated isotope and adduct peaks and proposed accurate compound masses per compound spectrum
- Combined cross-polarity annotation result (combinexsAnnos output)
- Tabular annotation counts/rates (e.g. annotation_rates_comparison.tsv with columns for mode, number of annotated spectra, and annotation rate)

## How to apply

Run CAMERA's annotation routines on the xsAnnotate objects from single-polarity runs: isotope peaks are annotated (findIsotopesWithValidation with parameters such as maxcharge, ppm, mzabs), then adduct relationships within each pseudospectrum are matched against the dynamic rule table to score hypotheses and assign the quasi-molecular ion. When both polarities are available, call combinexsAnnos with the paired positive and negative xsAnnotate objects, matching compound spectra across ion modes against a cross-polarity rule table that contains only common and trusted combinations, since these rules can override single-polarity annotations. Compare the number of annotated compound spectra before and after cross-polarity verification and compute annotation rates relative to each single mode. Avoid ad-hoc heuristics: CAMERA does not assume the most intense feature in a spectrum is the [M+H]+-ion; instead each candidate ion species in the spectrum earns a precedence score from the rule table.

## Related tools

- **CAMERA** (Bioconductor package providing the rule-table-driven ion-species annotation, findIsotopesWithValidation, and combinexsAnnos cross-polarity verification) — https://github.com/sneumann/CAMERA
- **R** (Implementation environment; CAMERA is installed from the Bioconductor repository)

## Examples

```
xsa.comb <- combinexsAnnos(xsa.pos, xsa.neg)
```

## Evaluation signals

- In the MM39 evaluation (mixture of 39 known compounds spanning m/z 161–822), CAMERA extracted accurate masses for 89.7% and 90.3% of annotatable compounds in positive and negative ion modes, respectively — compare your per-mode annotation rate against these reference values.
- After combining both ionization modes with combinexsAnnos, the combined approach should correctly determine molecular mass for all annotatable compounds (100%), and additionally annotate compounds missed by single-polarity runs.
- Sanity-check that annotations assigned via cross-polarity rules come only from common and trusted rule-table combinations, since these override single-polarity annotations.
- Verify that no annotation relied on the assumption that the most intense feature in a spectrum is the [M+H]+-ion (CAMERA explicitly does not use this heuristic).
- Check the output table schema: columns for mode, number of annotated spectra, and annotation rate.

## Limitations

- Missing molecular mass annotations occur either because the compound spectrum did not contain enough explained features, or in other cases several hypotheses had the same precedence scores.
- Compounds whose spectra contain only the molecular ion and fragment ions cannot be annotated directly, unless the neutral loss is added to the rule set.
- The cross-polarity rule table should only contain common and trusted combinations, because these rules can override the single-polarity annotations.
- CAMERA requires features to be grouped into compound spectra first (grouping + ion-species annotation are separate steps after feature detection); it does not annotate an ungrouped feature list.

## Evidence
