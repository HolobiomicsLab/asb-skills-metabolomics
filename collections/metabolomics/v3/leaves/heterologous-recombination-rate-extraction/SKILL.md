---
name: heterologous-recombination-rate-extraction
description: Use when a manuscript's Results section reports heterologous recombination
  assay percentages and event fractions per genotype that must be compiled into a
  machine-readable table to test whether double mutants show a synergistic elevation
  of heterologous recombination.
when_to_use_negative:
- The values are already in a structured table/CSV — no re-extraction needed.
- The target phenotype is crossover counts, RAD-51 foci timing, or GFP::MSH-5 localization
  rather than heterologous recombination rates; those require different extraction
  skills.
- The manuscript reports only qualitative statements about heterologous recombination
  with no numeric rates or event fractions.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pandas
  role: Assemble extracted genotype, rate_pct, numerator_events, and denominator_events
    values into a DataFrame and write it to heterologous_recombination_rates.csv
- name: Python
  role: Execution environment for the pandas-based extraction and CSV-writing workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac408
    title: The topoisomerase 3 zinc finger domain cooperates with the RMI1 scaffold
      to promote stable association of the BTR complex to recombination intermediates
      in the <i>Caenorhabditis elegans</i> germline
  source_content_hash: sha256:cdd6dc0df22f7061e5fcee180f70236d37b9e883d2e542fe832d932d714f6796
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# heterologous-recombination-rate-extraction

## Summary

Extracts and tabulates heterologous recombination rates for each genotype (top-3-ZnF, rmh-2(jf94), rmh-2(jf168), and the rmh-2(jf168);top-3-ZnF double mutant) reported in the manuscript's Results section, then persists them as a structured CSV for cross-genotype comparison of TOP-3-ZnF/RMH-2 cooperation.

## When to use

Use when a manuscript's Results section reports heterologous recombination assay percentages and event fractions per genotype that must be compiled into a machine-readable table to test whether double mutants show a synergistic elevation of heterologous recombination.

## When NOT to use

- The values are already in a structured table/CSV — no re-extraction needed.
- The target phenotype is crossover counts, RAD-51 foci timing, or GFP::MSH-5 localization rather than heterologous recombination rates; those require different extraction skills.
- The manuscript reports only qualitative statements about heterologous recombination with no numeric rates or event fractions.

## Inputs

- Manuscript Results section text describing the heterologous recombination assay for each genotype
- Reported rate percentages and event fractions per genotype (e.g. 0.2%, 0.8% 8/999, 0.96% 12/1247, 8.7% 16/183)

## Outputs

- heterologous_recombination_rates.csv with columns genotype, rate_pct, numerator_events, denominator_events (one row per genotype)

## How to apply

Parse the Results section to locate the heterologous recombination assay figures for each genotype: top-3-ZnF (0.2%), rmh-2(jf94) (0.8%, 8/999), rmh-2(jf168) (0.96%, 12/1247), and the rmh-2(jf168);top-3-ZnF double mutant (8.7%, 16/183). Use pandas in Python to assemble these into a DataFrame with columns genotype, rate_pct, numerator_events, denominator_events, leaving numerator/denominator as NA for top-3-ZnF where no fraction is stated. Write the DataFrame to heterologous_recombination_rates.csv. The key decision point is recognizing that the double mutant's 8.7% rate is similar to rmh-1 and him-6 mutant rates rather than an order-of-magnitude synergy, so the table should be interpreted against those comparator genotypes.

## Related tools

- **pandas** (Assemble extracted genotype, rate_pct, numerator_events, and denominator_events values into a DataFrame and write it to heterologous_recombination_rates.csv)
- **Python** (Execution environment for the pandas-based extraction and CSV-writing workflow)

## Examples

```
import pandas as pd; df = pd.DataFrame({'genotype':['top-3-ZnF','rmh-2(jf94)','rmh-2(jf168)','rmh-2(jf168);top-3-ZnF'],'rate_pct':[0.2,0.8,0.96,8.7],'numerator_events':[pd.NA,8,12,16],'denominator_events':[pd.NA,999,1247,183]}); df.to_csv('heterologous_recombination_rates.csv', index=False)
```

## Evaluation signals

- heterologous_recombination_rates.csv exists and is well-formed CSV with the four required columns: genotype, rate_pct, numerator_events, denominator_events
- rate_pct values fall within the reported range 0.2–8.7%
- The double mutant row carries rate_pct = 8.7 with numerator_events = 16 and denominator_events = 183; 16/183 ≈ 8.74% is consistent with the stated rate
- top-3-ZnF row has numerator_events and denominator_events set to NA since no fraction is stated in the manuscript
- All four genotypes (top-3-ZnF, rmh-2(jf94), rmh-2(jf168), rmh-2(jf168);top-3-ZnF) are present as rows

## Limitations

- No event fraction is stated for the top-3-ZnF mutant (only the 0.2% rate), so numerator/denominator must be left as NA.
- The double mutant's denominator is small (183 events vs. 999 and 1247 for the rmh-2 single mutants), so the 8.7% estimate carries wider sampling uncertainty.
- The article notes the double mutant 'tolerates the introduction of the of mln1 inversion chromosome very poorly, making the cross difficult to maintain', which can limit reproducibility of the assay strains.

## Evidence

- [results] rmh-2 double mutant rate finding: [evidence span withheld — anchor status: not_found]
- [abstract] ZnF cooperation with RMH-2: "The ZnF domain cooperates with the RMI1 homolog RMH-2 to stabilize association of the BTR complex with recombination intermediates and to prevent recombination between heterologous DNA sequences."
- [abstract] minor defect in top-3-ZnF single mutant: "the mutant has increased numbers of crossovers along its chromosomes, with minor defects in repressing heterologous recombination, and a marked delay"
