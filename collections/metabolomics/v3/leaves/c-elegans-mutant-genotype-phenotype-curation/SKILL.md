---
name: c-elegans-mutant-genotype-phenotype-curation
description: Use when a C. elegans genetics article reports per-genotype quantitative
  phenotypes (e.g. DAPI-stained body counts at diakinesis in Results figures such
  as 1E, 6C, S5E) that must be extracted verbatim into a machine-readable table.
when_to_use_negative:
- The phenotype of interest is only shown as a plotted panel with no numeric mean
  (±SD) stated in the Results text — extraction would require re-measurement, not
  curation
- The question concerns molecular interactions or localization (e.g. BTR complex recruitment,
  GFP::MSH-5 focus dynamics) rather than per-genotype quantitative phenotype values
- Input is raw imaging data (DAPI-stained nuclei) that must be counted de novo; this
  skill only curates counts already published in the article
edam_operation: http://edamontology.org/operation_0336
tools:
- name: Python re
  role: Regular-expression matching of the '(mean (±SD))' parenthetical patterns following
    genotype names to capture genotype, mean, and SD triplets
- name: pandas
  role: Assembles the 11-row genotype -> (mean, SD, source_figure) mapping and writes
    dapi_diakinesis_counts.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac408
    title: The topoisomerase 3 zinc finger domain cooperates with the RMI1 scaffold
      to promote stable association of the BTR complex to recombination intermediates
      in the <i>Caenorhabditis elegans</i> germline
  source_content_hash: sha256:1f5278d42bbedf447ca885e8cd7839b79b2beb94e00051c1bc2cbd66ce43b26e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Python re
  - pandas
  edam_operation: http://edamontology.org/operation_0336
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# c-elegans-mutant-genotype-phenotype-curation

## Summary

Curates quantitative genotype-phenotype data reported in C. elegans meiosis papers — e.g. mean (±SD) DAPI-stained body counts at diakinesis for alleles such as rtel-1, top-3-ZnF, rmh-2(jf94) — into a structured genotype -> (mean, SD, source figure) table, preserving verbatim published values.

## When to use

Use when a C. elegans genetics article reports per-genotype quantitative phenotypes (e.g. DAPI-stained body counts at diakinesis in Results figures such as 1E, 6C, S5E) that must be extracted verbatim into a machine-readable table.

## When NOT to use

- The phenotype of interest is only shown as a plotted panel with no numeric mean (±SD) stated in the Results text — extraction would require re-measurement, not curation
- The question concerns molecular interactions or localization (e.g. BTR complex recruitment, GFP::MSH-5 focus dynamics) rather than per-genotype quantitative phenotype values
- Input is raw imaging data (DAPI-stained nuclei) that must be counted de novo; this skill only curates counts already published in the article

## Inputs

- Article full-text file (PDF or text) of the C. elegans meiosis study
- List of target genotypes in scope (11 genotypes: rtel-1, top-3-ZnF;rtel-1, rmh-2(jf94), top-3-ZnF, rmh-2(jf94)/jf92;top-3-ZnF::ha, rmh-2;top-3-ZnF::ha;spo-11(ok79), rmh-2::mCherry, rmh-2::mCherry;top-3-ZnF, rmh-2(jf94);top-3-ZnF, rmh-1(jf54), rmh-1(jf54);top-3-ZnF)
- Figure anchors to search (Figure 1E, Figure 6C, Supplementary Figure S5E)

## Outputs

- dapi_diakinesis_counts.csv with columns genotype, mean_dapi_bodies, sd, source_figure and exactly 11 data rows

## How to apply

Load the article full text and scan the Results sections anchored to the target figures (here Figure 1E, Figure 6C, and Supplementary Figure S5E) for sentences reporting DAPI-stained body counts at diakinesis. Use Python re to match the '(mean (±SD))' parenthetical pattern following each genotype name, capturing genotype, mean, and SD triplets while carefully distinguishing allelic designators (rmh-2(jf94) vs rmh-2(jf92)), tagged constructs (rmh-2::mCherry, top-3-ZnF::ha), and multi-mutant combinations such as the spo-11(ok79) triple mutant. Assemble the full genotype -> (mean, SD, source_figure) mapping — here 11 rows including rtel-1 (6.2), top-3-ZnF;rtel-1 (6.1), rmh-2(jf94);top-3-ZnF (6.8), rmh-1(jf54) (8.8), and rmh-2;top-3-ZnF::ha;spo-11(ok79) (12.1). Write the table with pandas to a CSV with columns genotype, mean_dapi_bodies, sd, source_figure. Validate by confirming the exact expected row count and that every mean/SD matches the Results sentence verbatim.

## Related tools

- **Python re** (Regular-expression matching of the '(mean (±SD))' parenthetical patterns following genotype names to capture genotype, mean, and SD triplets)
- **pandas** (Assembles the 11-row genotype -> (mean, SD, source_figure) mapping and writes dapi_diakinesis_counts.csv)

## Examples

```
df = pd.DataFrame(mapping, columns=['genotype','mean_dapi_bodies','sd','source_figure']); df.to_csv('dapi_diakinesis_counts.csv', index=False)
```

## Evaluation signals

- CSV contains exactly 11 data rows, one per genotype named in the task scope
- Each mean and SD value matches verbatim the number in the corresponding Results sentence (Figure 1E, 6C, or S5E passage), e.g. rtel-1 = 6.2 ±0.4 and top-3-ZnF;rtel-1 = 6.1 ±0.3
- Every row carries the correct source_figure annotation (1E, 6C, or S5E)
- Allele vs. tagged-construct distinctions are preserved: rmh-2(jf94)/rmh-2(jf92), rmh-2::mCherry, top-3-ZnF::ha, and the spo-11(ok79) triple mutant are not conflated
- Spot-check values agree with the published spread, e.g. rmh-2;top-3-ZnF::ha;spo-11(ok79) = 12.1 (the Spo-11-dependent high value) versus rmh-1(jf54);top-3-ZnF = 8.9

## Limitations

- Similar designators are easy to conflate during regex extraction: rmh-2(jf94) vs rmh-2(jf92) mentions, tagged lines (rmh-2::mCherry, top-3-ZnF::ha), and the spo-11(ok79) triple mutant must be distinguished explicitly
- Supplemental figures referenced in the text may not be present in the provided full text (e.g. Supplemental Figure S7 is referenced but not included), so figure-anchored values from supplementary panels depend on availability
- The article does not state biological replicate numbers or sample sizes in the discussion of described experiments, limiting provenance checks on the curated counts
- The top-3-ZnF allele is described as a hypomorph rather than a null, so curated phenotype values for it must not be interpreted as complete loss-of-function

## Evidence

- [results] rtel-1 single mutants averaged 6.2 (±0.4 SD) DAPI-stained bodies at diakinesis, and top-3-ZnF; rtel-1 double mutants averaged 6.1 (±0.3 SD): [evidence span withheld — anchor status: not_found]
- [intro] To understand whether the ZnF domain contributes to D-loop discouragement, we constructed the top-3-ZnF; rtel-1 double mutant.: "To understand whether the ZnF domain contributes to D-loop discouragement, we constructed the top-3-ZnF; rtel-1 double mutant."
- [abstract] In contrast to the gene disruption allele, the top-3-ZnF mutant is viable, with no replication defects; the allele appears to be a hypomorph.: "In contrast to the gene disruption allele, the top-3-ZnF mutant is viable, with no replication defects; the allele appears to be a hypomorph."
- [discussion] recombination analysis in the top-3-ZnF mutant revealed an increased number of double COs compared with the wt: "recombination analysis in the top-3-ZnF mutant revealed an increased number of double COs compared with the wt"
