---
name: meiotic-recombination-genotype-annotation
description: Use when a paper reports heterologous recombination event counts or percentages
  across meiotic genotypes and you need a genotype-indexed table to compare mutant
  rates or test synergy of BTR-complex alleles.
when_to_use_negative:
- The analysis targets crossover number or D-loop discouragement phenotypes (e.g.
  top-3-ZnF; rtel-1 double mutant CO counts), not heterologous recombination rate
  quantification.
- Input data are recombination foci immunofluorescence measurements (e.g. GFP::MSH-5
  localization timing) rather than event counts/frequencies per genotype.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Python
  role: Used to parse reported recombination rates from the manuscript and assemble
    the genotype-annotated table
- name: pandas
  role: Builds the DataFrame with genotype, rate_pct, numerator_events, denominator_events
    columns and writes the validated CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac408
    title: The topoisomerase 3 zinc finger domain cooperates with the RMI1 scaffold
      to promote stable association of the BTR complex to recombination intermediates
      in the <i>Caenorhabditis elegans</i> germline
  source_content_hash: sha256:96e7f47209ec2a9d293eb22c530eb3c80b1287edccb3785138d85efb85d2559b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# meiotic-recombination-genotype-annotation

## Summary

Curates meiotic heterologous recombination rates per genotype (top-3-ZnF, rmh-2, rmh-1, him-6, and double mutants) from a C. elegans germline paper into a structured genotype-annotated table, enabling tests of whether TOP-3-ZnF and RMH-2 synergize to suppress recombination between heterologous DNA.

## When to use

Use when a paper reports heterologous recombination event counts or percentages across meiotic genotypes and you need a genotype-indexed table to compare mutant rates or test synergy of BTR-complex alleles.

## When NOT to use

- The analysis targets crossover number or D-loop discouragement phenotypes (e.g. top-3-ZnF; rtel-1 double mutant CO counts), not heterologous recombination rate quantification.
- Input data are recombination foci immunofluorescence measurements (e.g. GFP::MSH-5 localization timing) rather than event counts/frequencies per genotype.

## Inputs

- Manuscript Results text (PDF or plain text) reporting heterologous recombination rates per genotype
- Genotype labels including allele designations (top-3-ZnF, rmh-2(jf94), rmh-2(jf168), rmh-2(jf168);top-3-ZnF, rmh-1, him-6)

## Outputs

- heterologous_recombination_rates.csv with columns genotype, rate_pct, numerator_events, denominator_events

## How to apply

Parse the Results section and locate the heterologous recombination assay numbers for each genotype: top-3-ZnF (0.2%), rmh-2(jf94) (0.8%, 8/999), rmh-2(jf168) (0.96%, 12/1247), and the rmh-2(jf168); top-3-ZnF double mutant (8.7%, 16/183). Assemble these with pandas into a DataFrame with columns genotype, rate_pct, numerator_events, denominator_events, leaving numerator/denominator as NA where no fraction is stated (e.g. top-3-ZnF). Write the table to heterologous_recombination_rates.csv and validate that the four columns exist and rate_pct falls in the reported 0.2–8.7% range. The rationale is that the double mutant's markedly elevated rate (8.7% vs 0.8–0.96% for single rmh-2 mutants) is the key evidence that the ZnF domain cooperates with RMH-2 to discourage heterologous recombination.

## Related tools

- **Python** (Used to parse reported recombination rates from the manuscript and assemble the genotype-annotated table)
- **pandas** (Builds the DataFrame with genotype, rate_pct, numerator_events, denominator_events columns and writes the validated CSV)

## Examples

```
import pandas as pd; pd.DataFrame({'genotype':['top-3-ZnF','rmh-2(jf94)','rmh-2(jf168)','rmh-2(jf168);top-3-ZnF'],'rate_pct':[0.2,0.8,0.96,8.7],'numerator_events':[None,8,12,16],'denominator_events':[None,999,1247,183]}).to_csv('heterologous_recombination_rates.csv',index=False)
```

## Evaluation signals

- The output CSV contains exactly the four required columns: genotype, rate_pct, numerator_events, denominator_events.
- All rate_pct values fall within the reported 0.2–8.7% range.
- The rmh-2(jf168); top-3-ZnF double mutant row shows 8.7% (16/183), clearly elevated relative to rmh-2(jf94) at 0.8% (8/999) and rmh-2(jf168) at 0.96% (12/1247).
- Numerator/denominator fields are NA for genotypes where only a percentage is stated (e.g. top-3-ZnF at 0.2%).

## Limitations

- The rmh-2; top-3-ZnF double mutant tolerates the mln1 inversion chromosome very poorly, making crosses difficult to maintain and limiting double-mutant sample sizes (n=183 for the double mutant vs 999–1247 for rmh-2 single mutants).
- RMH-2::FLAG expression in the top-3-ZnF background is highly variable with large standard deviations, a reproducibility concern when extending the annotation to protein-level data.
- Supplemental Figure S7 (summary model) is referenced but may not be available in the parsed text.

## Evidence

- [abstract] ZnF-RMH-2 cooperation in suppressing heterologous recombination: "The ZnF domain cooperates with the RMI1 homolog RMH-2 to stabilize association of the BTR complex with recombination intermediates and to prevent recombination between heterologous DNA sequences."
- [intro] top-3-ZnF hypomorph phenotype motivates rate comparison: "Here, germline analysis of a top-3 ZnF deletion mutant shows that it is a hypomorph, with minor defects in discouraging heterologous recombination events and elevated numbers of COs."
- [intro] Synergy test via rmh-2; top-3-ZnF double mutant: "To investigate possible cooperation between TOP-3-ZnF and RMH-2, we generated the rmh-2(jf94); top-3-ZnF::ha double mutant and analyzed its phenotype."
- [discussion] Recombination analysis as a genotype-level assay: "recombination analysis in the top-3-ZnF mutant revealed an increased number of double COs compared with the wt"
