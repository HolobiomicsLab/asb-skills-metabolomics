---
name: microbiome-multiple-testing-correction
description: Decision support for taxon-level association testing on microbiome abundance
  tables (e.g. species-level ANCOM-BC2 output) where multiple hypotheses are tested
  and raw p-values must be adjusted before reporting differentially abundant taxa.
when_to_use_negative:
- The input is already an adjusted result table (e.g. a published ANCOM-BC2 output
  with q-values) — re-correcting would double-penalize.
- The analysis is a single pre-registered hypothesis on one taxon, where no multiplicity
  correction is needed.
- The abundance table is a compositional mock community where only a handful of taxa
  are tested and the correction would obscure real signals without changing conclusions.
edam_operation: http://edamontology.org/operation_3663
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3697
tools:
- name: ANCOM-BC2
  role: Performs the species-level differential-abundance test whose p-values are
    then adjusted; the paper uses it on the SPF Sacrifice fecal metagenomics data
    with genotype as the primary contrast
- name: tissueMASST
  role: Downstream context tool that links the corrected taxon findings to metabolite
    signatures (e.g. M. schaedleri/Helicobacter enrichment correlating with serum
    phenylacetyl-carnitine), but is not itself part of the correction step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2025.04.28.651123
    title: A Multi-Organ Murine Metabolomics Atlas Reveals Molecular Dysregulations
      in Alzheimer’s Disease
  source_content_hash: sha256:e585ce57e21aa148dfe780a2e7befc19e103e23f8f617bd78881c28430b071a9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANCOM-BC2
  - tissueMASST
  edam_operation: http://edamontology.org/operation_3663
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# microbiome-multiple-testing-correction

## Summary

Applies family-wise error rate or false-discovery-rate correction to taxon-level association tests (e.g. ANCOM-BC2, MaAsLin2) so that differentially abundant microbial species between conditions are reported with adjusted p-values, as done for the 3xTg vs WT fecal shotgun metagenomics comparison.

## When to use

Decision support for taxon-level association testing on microbiome abundance tables (e.g. species-level ANCOM-BC2 output) where multiple hypotheses are tested and raw p-values must be adjusted before reporting differentially abundant taxa.

## When NOT to use

- The input is already an adjusted result table (e.g. a published ANCOM-BC2 output with q-values) — re-correcting would double-penalize.
- The analysis is a single pre-registered hypothesis on one taxon, where no multiplicity correction is needed.
- The abundance table is a compositional mock community where only a handful of taxa are tested and the correction would obscure real signals without changing conclusions.

## Inputs

- Species-level (or ASV/genera-level) relative-abundance table with paired metadata (e.g. genotype 3xTg vs WT from the SPF Sacrifice cohort)
- Per-taxon association test output (p-values and log-fold changes) from a differential-abundance model such as ANCOM-BC2 or MaAsLin2

## Outputs

- Adjusted p-values (FDR or family-wise) per taxon, aligned to the original test table
- Exported result table (e.g. ancom_bc2_results.csv) containing log-fold changes, raw p-values, and adjusted p-values for every tested species

## How to apply

Run the taxon-level test (e.g. ANCOM-BC2 on a species-level abundance table from the 666-sample SPF Sacrifice fecal shotgun metagenomics dataset, with genotype 3xTg vs WT as the primary contrast) and then adjust the per-taxon p-values for the number of taxa tested. Choose the correction family to match the claim: Benjamini-Hochberg FDR when the goal is a ranked list of candidates for follow-up (as in the paper's ANCOM-BC2 species table with adjusted p-values), and Holm or Bonferroni when a single confirmed claim is required. Report adjusted p-values alongside log-fold changes, not raw p-values, and keep the uncorrected values in the exported table (e.g. ancom_bc2_results.csv) so the correction step is auditable. Apply the correction over all taxa in the tested model simultaneously — do not correct within subgroups or after subsetting to significant hits, which invalidates the FDR guarantee.

## Related tools

- **ANCOM-BC2** (Performs the species-level differential-abundance test whose p-values are then adjusted; the paper uses it on the SPF Sacrifice fecal metagenomics data with genotype as the primary contrast)
- **tissueMASST** (Downstream context tool that links the corrected taxon findings to metabolite signatures (e.g. M. schaedleri/Helicobacter enrichment correlating with serum phenylacetyl-carnitine), but is not itself part of the correction step)

## Examples

```
Rscript run_ancom_bc2.R --abundances species_table.tsv --metadata genotype.tsv --contrast "3xTg vs WT" --adjust-method BH --output ancom_bc2_results.csv
```

## Evaluation signals

- The exported table contains one adjusted p-value per tested species and no missing values among taxa that entered the test.
- Adjustment method and tested-taxon count are recorded in the output header or provenance metadata (e.g. BH-FDR over 500+ species).
- Direction of log-fold change is unchanged by correction: significant hits retain the reported signs (increased M. schaedleri/Helicobacter/Desulfovibrio/Escherichia/Shigella; decreased Akkermansia muciniphila/Bifidobacterium).
- Re-running the correction on the same raw p-values yields identical adjusted values (deterministic).
- Raw p-values remain in the output so the correction factor (raw p / adjusted p) can be spot-checked for monotonicity.

## Limitations

- Correction does not fix confounding: the paper notes 3xTg and WT mice were bred and housed separately before the experiment, which can drive apparent differential abundance regardless of statistical adjustment.
- Compositional abundance tables from different cohorts (Longitudinal vs Sacrifice, fresh fecal vs post-mortem colon) show systematic differences that correction cannot remove.
- FDR control depends on the tested family; subsetting to significant hits before correction or correcting within subgroups breaks the guarantee.

## Evidence

- [results] ANCOM-BC2 differential abundance testing at species level: [evidence span withheld — anchor status: not_found]
- [abstract] metagenomics dataset: "complemented by shotgun metagenomics sequencing data (666 samples)"
- [discussion] cohort confound: "3xTg and WT mice were bred and housed separately before the start of the experiment"
