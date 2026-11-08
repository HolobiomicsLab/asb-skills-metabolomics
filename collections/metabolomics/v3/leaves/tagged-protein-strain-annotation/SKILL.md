---
name: tagged-protein-strain-annotation
description: Use when reviewing or reproducing protein–protein interaction evidence
  from a paper that relies on endogenously tagged worm strains, and you need a structured
  table linking construct pairs (tag type and fusion position) to the assay outcomes
  and their figure locations.
when_to_use_negative:
- When the paper's interaction claims rely on genetic or phenotypic data only (e.g.
  crossover frequency counts, GFP::MSH-5 focus timing) and no tagged-protein physical-interaction
  assay is described.
- When a proteomics database (UniProt, WormBase) already provides the tagged-strain
  and assay annotations you need and re-deriving them from full text adds no value.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: pandas
  role: Constructs the interaction evidence DataFrame and writes/validates the two-row
    CSV
- name: Microsoft Excel
  role: Manual inspection/editing of the evidence table alongside the pandas output
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac408
    title: The topoisomerase 3 zinc finger domain cooperates with the RMI1 scaffold
      to promote stable association of the BTR complex to recombination intermediates
      in the <i>Caenorhabditis elegans</i> germline
  source_content_hash: sha256:f42e78870701a5b8815af0b89580b981b8c6c2d60fb9edf3d2646dc55b4dd626
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Microsoft Excel
  - Python
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tagged-protein-strain-annotation

## Summary

Annotates C. elegans strains by the fluorescent or epitope tags carried by their proteins (e.g. RMH-2::mCherry, TOP-3::HA, GFP::MSH-5, RMH-2::FLAG) and maps each tagged construct pair to the physical-interaction assay that depends on it, enabling reproducible evidence tracking.

## When to use

Use when reviewing or reproducing protein–protein interaction evidence from a paper that relies on endogenously tagged worm strains, and you need a structured table linking construct pairs (tag type and fusion position) to the assay outcomes and their figure locations.

## When NOT to use

- When the paper's interaction claims rely on genetic or phenotypic data only (e.g. crossover frequency counts, GFP::MSH-5 focus timing) and no tagged-protein physical-interaction assay is described.
- When a proteomics database (UniProt, WormBase) already provides the tagged-strain and assay annotations you need and re-deriving them from full text adds no value.

## Inputs

- Article Results text describing tagged strains and assays (e.g. the 'Top-3-Znf And Rmh-2 Physically Interact And Form Protein Complexes' section)
- List of tagged constructs mentioned in the text (RMH-2::mCherry, TOP-3::HA, RMH-2::FLAG, GFP::MSH-5, top-3-ZnF::ha)
- Pandas-compatible Python environment

## Outputs

- interaction_evidence_table.csv with columns assay, construct_pair, outcome, evidence_location
- A two-row table in which each row cites a specific figure or supplementary panel (Figure 7C; Supplementary Figure S5F)

## How to apply

First, scan the Results text for every passage that names a tagged strain or tagged fusion protein — for this article: RMH-2::mCherry, TOP-3::HA, GFP::MSH-5, RMH-2::FLAG, and the top-3-ZnF::ha allele — recording the protein name, tag fluorophore/epitope, and fusion position (C-terminal vs N-terminal) exactly as reported. Next, for each construct pair used in an interaction assay (yeast two-hybrid with RMH-2 as prey and TOP-3 as bait, or reciprocal co-IPs in the rmh-2::mCherry / top-3::HA strain), populate a row in a pandas DataFrame with columns assay, construct_pair, outcome, and evidence_location. Third, cite the exact evidence source for each row (Figure 7C for the two-hybrid row; Supplementary Figure S5F for the reciprocal co-IP row) rather than paraphrasing. Finally, write the rows to interaction_evidence_table.csv and validate that it contains exactly two data rows with all four columns populated. The rationale is that tagged-strain identity is the precondition for every biochemical interaction claim in the paper, so linking strain → tag → assay → figure makes the evidence traceable and machine-checkable.

## Related tools

- **pandas** (Constructs the interaction evidence DataFrame and writes/validates the two-row CSV)
- **Microsoft Excel** (Manual inspection/editing of the evidence table alongside the pandas output)

## Examples

```
import pandas as pd; df = pd.DataFrame([{'assay':'yeast two-hybrid','construct_pair':'RMH-2 prey / TOP-3 bait','outcome':'positive interaction','evidence_location':'Figure 7C'},{'assay':'reciprocal co-IP','construct_pair':'RMH-2::mCherry / TOP-3::HA','outcome':'positive interaction','evidence_location':'Supplementary Figure S5F'}]); df.to_csv('interaction_evidence_table.csv', index=False)
```

## Evaluation signals

- The output CSV contains exactly 2 data rows (one per assay: yeast two-hybrid, reciprocal co-IP).
- All four columns (assay, construct_pair, outcome, evidence_location) are populated and non-empty in every row.
- The yeast two-hybrid row's evidence_location cites Figure 7C, not another figure.
- The co-IP row's evidence_location cites Supplementary Figure S5F.
- construct_pair strings preserve the article's tag nomenclature (e.g. 'RMH-2::mCherry / TOP-3::HA') rather than abbreviating tags away.

## Limitations

- Supplementary figure content (e.g. Supplementary Figure S7, the summary model) is referenced but not available in the provided text, so some evidence_location citations cannot be independently verified from the source.
- RMH-2::FLAG expression levels were highly variable in the top-3-ZnF; rmh-2::flag background, causing large standard deviations — tagged-strain-dependent quantifications from that genotype may not be reproducible.
- Immunofluorescence resolution limited the localization observations of RMH-2, which constrains conclusions drawn purely from tagged-protein imaging.

## Evidence

- [results] A yeast two-hybrid assay using RMH-2 as prey and TOP-3 as bait (C-terminally or N-terminally tagged) revealed a physical interaction between the two proteins (Figure 7C).: [evidence span withheld — anchor status: not_found]
- [discussion] Western blot analysis of RMH-2::FLAG whole worm extracts: "Western blot analysis of RMH-2::FLAG whole worm extracts of the top-3-ZnF; rmh-2::flag mutant revealed highly variable expression levels"
- [intro] To investigate possible cooperation between TOP-3-ZnF and RMH-2, we generated the rmh-2(jf94); top-3-ZnF::ha double mutant and analyzed its phenotype.: "To investigate possible cooperation between TOP-3-ZnF and RMH-2, we generated the rmh-2(jf94); top-3-ZnF::ha double mutant and analyzed its phenotype."
