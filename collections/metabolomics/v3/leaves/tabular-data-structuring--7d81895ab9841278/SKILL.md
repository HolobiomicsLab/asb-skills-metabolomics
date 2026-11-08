---
name: tabular-data-structuring
description: Use when a C. elegans meiosis paper reports foci-per-nucleus counts for
  specific genotypes and meiotic zones (e.g. GFP::RMH-1 or GFP::MSH-5 in top-3-ZnF
  or rmh-2 backgrounds) and these must be tabulated for statistical comparison or
  reproducibility checks.
when_to_use_negative:
- The article reports only qualitative presence/absence of foci without numeric means
  or SDs — no tabular rows can be filled.
- The quantification covers imaging modalities other than GFP-tagged focus counting
  (e.g. Western blot densitometry or crossover mapping) — different output schema
  required.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Microsoft Excel
  role: Used to assemble and inspect the 16-row foci_means table before applying the
    paper's statistical tests
- name: GraphPad Prism
  role: Software cited by the article for downstream statistical analysis of the extracted
    foci counts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac408
    title: The topoisomerase 3 zinc finger domain cooperates with the RMI1 scaffold
      to promote stable association of the BTR complex to recombination intermediates
      in the <i>Caenorhabditis elegans</i> germline
  source_content_hash: sha256:2826137b53e54e480b06ed17d216f25d6469c51a8aa7c5e5f20f9ff89fcff1e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GraphPad Prism
  - Microsoft Excel
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# tabular-data-structuring

## Summary

Extracts the paper's quantified GFP::RMH-1 and GFP::MSH-5 focus counts (means ± SD per meiotic zone) from the Results text into a structured 16-row CSV (foci_means.csv) for downstream statistical analysis in Prism 6/GraphPad or Microsoft Excel.

## When to use

Use when a C. elegans meiosis paper reports foci-per-nucleus counts for specific genotypes and meiotic zones (e.g. GFP::RMH-1 or GFP::MSH-5 in top-3-ZnF or rmh-2 backgrounds) and these must be tabulated for statistical comparison or reproducibility checks.

## When NOT to use

- The article reports only qualitative presence/absence of foci without numeric means or SDs — no tabular rows can be filled.
- The quantification covers imaging modalities other than GFP-tagged focus counting (e.g. Western blot densitometry or crossover mapping) — different output schema required.

## Inputs

- Results-section text passages reporting foci-per-nucleus means and SDs (Figure 4D for GFP::RMH-1; Figure 5A for GFP::MSH-5)
- Zone classification scheme where zone 2 = early pachynema, zone 3 = mid pachynema, zone 4 = late pachynema

## Outputs

- foci_means.csv — 16 data rows with columns marker, genotype, zone, mean_foci_per_nucleus, sd
- Downstream statistical summaries in GraphPad Prism 6 or Microsoft Excel

## How to apply

Parse the Results section text to locate the explicit numeric focus-quantification passages for GFP::RMH-1 (Figure 4D) and GFP::MSH-5 (Figure 5A). Extract each stated mean and SD for the genotype–zone–marker combinations using the paper's zone scheme (zone 2 = early pachynema, zone 3 = mid pachynema, zone 4 = late pachynema). Assemble exactly 16 rows covering: GFP::RMH-1 for gfp::rmh-1 and gfp::rmh-1; top-3-ZnF (zones 2–3), rmh-2(jf92); top-3-ZnF::ha (zones 2–4); and GFP::MSH-5 for gfp::msh-5, gfp::msh-5; top-3-ZnF, rmh-2(jf94) (zones 2–3), and rmh-2(jf92); top-3-ZnF::ha (zones 2–4). Write the table to foci_means.csv with columns marker, genotype, zone, mean_foci_per_nucleus, sd using pandas, and use Microsoft Excel or GraphPad Prism for downstream statistics as the study did.

## Related tools

- **Microsoft Excel** (Used to assemble and inspect the 16-row foci_means table before applying the paper's statistical tests)
- **GraphPad Prism** (Software cited by the article for downstream statistical analysis of the extracted foci counts)

## Examples

```
import pandas as pd; df = pd.read_csv('foci_means.csv'); assert len(df) == 16 and list(df.columns) == ['marker','genotype','zone','mean_foci_per_nucleus','sd']
```

## Evaluation signals

- Output file exists and parses as CSV with exactly 16 data rows.
- Zone 2 GFP::RMH-1 means match 15.2 (gfp::rmh-1) and 3.6 (gfp::rmh-1; top-3-ZnF).
- Zone 2 GFP::MSH-5 means match 8.6, 2.3, 1.8 and 0.5 across the four genotypes.
- Column headers are exactly marker, genotype, zone, mean_foci_per_nucleus, sd.
- Every row carries a non-negative numeric mean and sd, consistent with foci-per-nucleus counts.

## Limitations

- Highly variable RMH-2::FLAG expression levels caused large standard deviations that were not observed in the rmh-2::flag mutant alone — reproduce these outliers verbatim rather than smoothing them.
- The rmh-2; top-3-ZnF double mutant with the mln1 inversion chromosome is difficult to maintain, so the corresponding genotype rows may be under-represented.
- RMH-2 localization observations are limited by the resolution level of the immunofluorescence analysis, which caps how precisely focus counts per zone can be interpreted.

## Evidence

- [methods] Zone 2 mean values match the Results text: [evidence span withheld — anchor status: not_found]
- [results] Zone scheme explicitly defined: [evidence span withheld — anchor status: not_found]
- [methods] Validation criteria and file name: [evidence span withheld — anchor status: not_found]
- [discussion] Expression variability limitation: "Western blot analysis of RMH-2::FLAG whole worm extracts of the top-3-ZnF; rmh-2::flag mutant revealed highly variable expression levels, causing large standard deviations that we did not observe in"
