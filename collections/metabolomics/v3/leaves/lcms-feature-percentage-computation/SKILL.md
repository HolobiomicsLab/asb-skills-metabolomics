---
name: lcms-feature-percentage-computation
description: Use when analyzing how capillary voltage, ion energy, or collision energy
  settings change the number and percentage of ISF features detected relative to real
  metabolic features in LC-MS feature tables.
when_to_use_negative:
- When the goal is de novo recognition of in-source fragments themselves (i.e. running
  the find.level1/2/3 identification) rather than summarizing their counts and percentages
  across MS parameter conditions
- When the input is a raw mzXML needing MS1 feature extraction via XCMS.featuretable()
  rather than a summarized feature table with ISF relationships already annotated
- When the goal is adduct or isotope annotation via CAMERA, which ISFrag restricts
  to XCMS-only analysis
edam_operation: http://edamontology.org/operation_3628
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: ISFrag
  role: R package used to identify ISF features (find.level1(), find.level2(), find.level3(),
    get.ISFrag.results()) whose counts feed the percentage computation across MS parameter
    conditions
  repo: https://github.com/HuanLab/ISFrag.git
- name: R
  role: Language in which the percentage computation, data-frame organization, and
    trend figure rendering are implemented
  repo: https://github.com/HuanLab/ISFrag.git
- name: xcms
  role: Upstream MS1 feature extraction via XCMS.featuretable() producing the dataframe-formatted
    feature table consumed by ISFrag
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.1c01644
    title: 'ISFrag: De Novo Recognition of In-Source Fragments for Liquid Chromatography–Mass
      Spectrometry Data'
  source_content_hash: sha256:f519e86c1688d3066ff3f411d8491a1a8f7ebfae7936021af70867cbc230388d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HuanLab/ISFrag.git
  tools:
  - ISFrag
  - R
  - xcms
  edam_operation: http://edamontology.org/operation_3628
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lcms-feature-percentage-computation

## Summary

Computes the number of in-source fragment (ISF) features and their percentage relative to real metabolic features across MS parameter conditions, as implemented in the ISFrag workflow (HuanLab) for LC-MS metabolomics feature tables.

## When to use

Use when analyzing how capillary voltage, ion energy, or collision energy settings change the number and percentage of ISF features detected relative to real metabolic features in LC-MS feature tables.

## When NOT to use

- When the goal is de novo recognition of in-source fragments themselves (i.e. running the find.level1/2/3 identification) rather than summarizing their counts and percentages across MS parameter conditions
- When the input is a raw mzXML needing MS1 feature extraction via XCMS.featuretable() rather than a summarized feature table with ISF relationships already annotated
- When the goal is adduct or isotope annotation via CAMERA, which ISFrag restricts to XCMS-only analysis

## Inputs

- Complete feature table with ISF relationships annotated in additional columns (csv format, columns: m/z, retention time, min retention time, max retention time, followed by feature intensities)
- Detailed ISF precursor-fragment relationship dataframe
- Deposited count/percentage records per MS parameter condition
- mzXML files from DDA, DIA, or fullscan analyses for upstream XCMS feature extraction

## Outputs

- Data frame keyed by MS parameter setting with columns for real metabolic features, ISF features, and ISF percentage
- Trend figure (PNG) comparing feature counts and ISF percentage across parameter conditions

## How to apply

First obtain the ISFrag result feature table (or the complete feature table with ISF relationships annotated in additional columns) for each MS parameter condition, either by running the find.level1()/find.level2()/find.level3() identification pipeline via get.ISFrag.results() or by loading deposited counts. Organize the records into a data frame keyed by parameter setting (e.g. collision energy level) with columns for real metabolic features, ISF features, and ISF percentage. Compute the percentage of ISF features relative to real metabolic features for each condition to confirm deposited values. Render a trend figure with parameter condition on the x-axis and feature counts compared with ISF percentage, saving as PNG. The rationale, grounded in the article's abstract finding, is that increasing energies can raise both the number of real metabolic features and ISF features, yet the percentage of ISF features may not necessarily increase.

## Related tools

- **ISFrag** (R package used to identify ISF features (find.level1(), find.level2(), find.level3(), get.ISFrag.results()) whose counts feed the percentage computation across MS parameter conditions) — https://github.com/HuanLab/ISFrag.git
- **R** (Language in which the percentage computation, data-frame organization, and trend figure rendering are implemented) — https://github.com/HuanLab/ISFrag.git
- **xcms** (Upstream MS1 feature extraction via XCMS.featuretable() producing the dataframe-formatted feature table consumed by ISFrag)

## Examples

```
xcmsFT <- XCMS.featuretable(MS1directory = MS1directory, type = "single", peakwidth = c(5,20)); isfResults <- get.ISFrag.results(find.level1(find.level2(find.level3(xcmsFT))))
```

## Evaluation signals

- Reproduced counts and percentages for real metabolic features and ISF features match the deposited values for each MS parameter condition
- The computed ISF percentage equals the number of ISF features divided by the number of real metabolic features (or the equivalent denominator used in the deposited table) for every condition
- The trend figure shows both real metabolic feature counts and ISF feature counts increasing with energies, while the ISF percentage does not necessarily increase
- All three retention time columns in the source feature table are in seconds and the required column order (m/z, rt, rtmin, rtmax, intensities) is preserved
- Output figure is saved as a valid PNG file

## Limitations

- Percentage of ISF features does not necessarily increase even when energies increase the counts of both real metabolic and ISF features — must not be assumed monotonically rising
- Custom feature tables from MS-DIAL or MZmine2 must have columns in exactly the order: m/z, retention time, min retention time, max retention time, followed by per-sample intensities; otherwise ISFrag cannot read them
- CAMERA adduct and isotope annotation can only be used for XCMS-only ISFrag analysis
- ISFrag achieved 100% correct recognition of level 1 ISF features but only over 80% for level 2 ISF features when tested using metabolite standards

## Evidence
