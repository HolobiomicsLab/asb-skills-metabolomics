---
name: germline-zone-annotation
description: Use when quantifying meiotic recombination foci per nucleus in C. elegans
  gonads and nuclei must be binned into zone 2 (early), 3 (mid), or 4 (late) pachynema
  before computing per-genotype mean and SD, e.g. for TOP-3::HA, GFP::RMH-1 or GFP::MSH-5
  foci.
when_to_use_negative:
- The data are whole-gonad or whole-animal counts not stratified by meiotic zone,
  so per-zone binning is meaningless
- The nuclei come from mitotic or premeiotic tip regions (outside pachynema zones
  2–4) or from a non-C. elegans germline whose zonal anatomy differs
- The question concerns crossover (CO) counts along chromosomes rather than per-nucleus
  recombination-focus counts, which the article analyzed separately
edam_operation: http://edamontology.org/operation_0226
tools:
- name: pandas
  role: compile genotype-zone mean/SD foci rows into a single CSV with columns genotype,
    zone, mean_foci_per_nucleus, sd and validate row counts
- name: Microsoft Excel
  role: alternative tabulation of the 16 GFP::RMH-1/GFP::MSH-5 genotype-zone rows
    into foci_means.csv
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac408
    title: The topoisomerase 3 zinc finger domain cooperates with the RMI1 scaffold
      to promote stable association of the BTR complex to recombination intermediates
      in the <i>Caenorhabditis elegans</i> germline
  source_content_hash: sha256:3a6fee5d990c440fd1e325af3539892b717ca9934bfdd186182b37bea043f326
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphPad Prism
  - Microsoft Excel
  - pandas
  edam_operation: http://edamontology.org/operation_0226
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# germline-zone-annotation

## Summary

Assigns C. elegans germline nuclei to discrete meiotic zones — zone 2 (early pachynema), zone 3 (mid pachynema), zone 4 (late pachynema) — so that recombination-foci counts per nucleus (TOP-3::HA, GFP::RMH-1, GFP::MSH-5) can be reported as genotype-zone mean ± SD tables compatible with the BTR-complex study's Figures 3–6.

## When to use

Use when quantifying meiotic recombination foci per nucleus in C. elegans gonads and nuclei must be binned into zone 2 (early), 3 (mid), or 4 (late) pachynema before computing per-genotype mean and SD, e.g. for TOP-3::HA, GFP::RMH-1 or GFP::MSH-5 foci.

## When NOT to use

- The data are whole-gonad or whole-animal counts not stratified by meiotic zone, so per-zone binning is meaningless
- The nuclei come from mitotic or premeiotic tip regions (outside pachynema zones 2–4) or from a non-C. elegans germline whose zonal anatomy differs
- The question concerns crossover (CO) counts along chromosomes rather than per-nucleus recombination-focus counts, which the article analyzed separately

## Inputs

- Scored foci counts per nucleus per genotype (TOP-3::HA, GFP::RMH-1, GFP::MSH-5) with each nucleus tagged as zone 2, 3 or 4 of the C. elegans germline
- Results-section passages reporting genotype-zone mean ± SD foci values (e.g. rtel-1; top-3-ZnF::ha zone 2: 8.8 ± 6.5; zone 3: 17.1 ± 5.7; zone 4: 7.2 ± 1.2)

## Outputs

- CSV table (e.g. foci_means.csv) with columns marker, genotype, zone, mean_foci_per_nucleus, sd — 16 rows for GFP::RMH-1/GFP::MSH-5 genotype-zone combinations, 13 rows for the TOP-3::HA compilation including the rmh-2(jf94) zone-2 value 9.4 ± 6

## How to apply

First map each scored nucleus to its germline zone using the study's scheme in which zone 2 is early pachynema, zone 3 is mid pachynema and zone 4 is late pachynema, since all reported foci statistics are stratified on this axis. Then, for each genotype-zone-marker combination, compute the mean number of foci per nucleus and its SD — e.g. GFP::RMH-1 zone 2 gives 15.2 (±6.9 SD) for gfp::rmh-1 versus 3.6 (±3.8 SD) for gfp::rmh-1; top-3-ZnF. Structure the output as flat rows with marker, genotype, zone, mean_foci_per_nucleus and sd columns, and write them to CSV with pandas or Microsoft Excel (the study used Prism 6/GraphPad for downstream statistics). Where the article reports no value for a combination (e.g. rmh-2(jf94) TOP-3::HA zones 3–4), leave those cells empty rather than interpolating. Finally, cross-check every mean/SD pair verbatim against the Results text, since these numbers feed direct comparisons such as the delayed-and-reduced early TOP-3 foci in the ZnF mutant.

## Related tools

- **pandas** (compile genotype-zone mean/SD foci rows into a single CSV with columns genotype, zone, mean_foci_per_nucleus, sd and validate row counts)
- **Microsoft Excel** (alternative tabulation of the 16 GFP::RMH-1/GFP::MSH-5 genotype-zone rows into foci_means.csv)

## Examples

```
import pandas as pd; pd.DataFrame(rows, columns=['marker','genotype','zone','mean_foci_per_nucleus','sd']).to_csv('foci_means.csv', index=False)
```

## Evaluation signals

- Zone values are restricted to {2, 3, 4} and each row's zone maps to the documented meaning (2=early, 3=mid, 4=late pachynema)
- The GFP::RMH-1/GFP::MSH-5 CSV parses with exactly 16 data rows; the TOP-3::HA compilation has exactly 13 data rows
- Mean/SD pairs match the Results text verbatim, e.g. GFP::RMH-1 zone 2: 15.2 ± 6.9 (gfp::rmh-1) and 3.6 ± 3.8 (gfp::rmh-1; top-3-ZnF); GFP::MSH-5 zone 2 values 8.6, 2.3, 1.8, 0.5 across the four genotypes
- Missing combinations (rmh-2(jf94) TOP-3::HA zones 3–4) are empty cells, not zeros or imputed values
- Genotype labels use the article's allele nomenclature exactly (top-3-ZnF, rtel-1; top-3-ZnF::ha, rmh-2(jf92), rmh-2(jf94))

## Limitations

- Zone-4 data are unavailable for some genotypes (rmh-2(jf94) TOP-3::HA), so late-pachynema comparisons cannot be completed for all backgrounds
- The rmh-2; top-3-ZnF double mutant tolerates the mln1 inversion chromosome poorly, making strains difficult to maintain and restricting which genotype-zone combinations can be scored
- RMH-2::FLAG expression in the top-3-ZnF background is highly variable with large standard deviations, reducing confidence in quantitative comparisons involving that genotype
- Zone assignment rests on immunofluorescence-based germline position, whose resolution the authors note is limited

## Evidence

- [methods] zone scheme defines zone 2 as early pachynema: [evidence span withheld — anchor status: not_found]
- [results] rtel-1; top-3-ZnF::ha zone 2-4 foci values: [evidence span withheld — anchor status: not_found]
- [discussion] early TOP-3 foci delayed and reduced in ZnF mutant: "Early TOP-3-containing meiotic recombination foci (seen in zones 2 and 3 in the wt) were delayed and reduced in number"
