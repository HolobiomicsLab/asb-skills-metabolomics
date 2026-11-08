---
name: metabolite-list-overrepresentation-enrichment
description: Use when a harmonized (MMS-merged) or unmerged metabolite identifier
  list needs over-representation enrichment against SMPDB pathways to test whether
  merging changes significant pathway detection.
when_to_use_negative:
- The input is raw spectral data or a feature table rather than a curated, identifier-resolved
  list of metabolite compounds
- You only have a single compound list and no unmerged counterpart, so the with-MMS
  versus without-MMS contrast cannot be performed
- The target pathway universe is not SMPDB and results must not be compared against
  the article's reported enrichment outcome
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: MetaboAnalyst 5.0
  role: Executes the over-representation pathway enrichment of the filtered compound
    lists for both the MMS-merged and unmerged runs
- name: SMPDB
  role: Provides the pathway universe (Small Molecule Pathway Database) against which
    over-representation is tested
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.3390/metabo13121167
    title: 'A Metabolites Merging Strategy (MMS): Harmonization to Enable Studies’
      Intercomparison'
  source_content_hash: sha256:81d7dcca8619b1e34e30fa5657029a9e42d7aced38997bd81ff4c67f6b96ea9d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaboAnalyst 5.0
  - SMPDB
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolite-list-overrepresentation-enrichment

## Summary

Runs over-representation pathway enrichment on harmonized (MMS-merged) versus unmerged urinary metabolite compound lists using MetaboAnalyst 5.0 with Small Molecule Pathway Database (SMPDB) pathways, to quantify how dataset harmonization changes which pathways reach significance at FDR < 0.05.

## When to use

Use when a harmonized (MMS-merged) or unmerged metabolite identifier list needs over-representation enrichment against SMPDB pathways to test whether merging changes significant pathway detection.

## When NOT to use

- The input is raw spectral data or a feature table rather than a curated, identifier-resolved list of metabolite compounds
- You only have a single compound list and no unmerged counterpart, so the with-MMS versus without-MMS contrast cannot be performed
- The target pathway universe is not SMPDB and results must not be compared against the article's reported enrichment outcome

## Inputs

- MMS-merged urinary asthma metabolite dataset (391 metabolites, Supplementary Table S2), filtered to compounds found in at least two information sources
- Unmerged metabolite dataset (547 entries, same compounds without MMS harmonization), filtered to compounds found in at least two sources
- SMPDB pathway universe accessed through MetaboAnalyst 5.0

## Outputs

- Per-pathway p-value and FDR for the MMS-merged run
- Per-pathway p-value and FDR for the unmerged run
- Side-by-side comparison table saved as pathway_enrichment_comparison.csv

## How to apply

Load the MMS-merged urinary asthma metabolite dataset (391 metabolites, Supplementary Table S2) and filter to compounds found in at least two information sources, then submit this filtered compound list to MetaboAnalyst 5.0 for over-representation analysis with SMPDB as the pathway universe. In parallel, load the unmerged dataset (547 entries, i.e., the same compounds without MMS harmonization), apply the identical 'found in at least two sources' filter, and repeat the identical MetaboAnalyst 5.0 / SMPDB over-representation run. Extract the p-value and false discovery rate (FDR) for every pathway from both runs and assemble them into a single side-by-side comparison table, saving it as pathway_enrichment_comparison.csv. The rationale, per the source article, is that MMS harmonization by InChIKeys can expose significant pathways that remain hidden without merging, so the two runs must be identical except for the merging step to isolate that effect.

## Related tools

- **MetaboAnalyst 5.0** (Executes the over-representation pathway enrichment of the filtered compound lists for both the MMS-merged and unmerged runs)
- **SMPDB** (Provides the pathway universe (Small Molecule Pathway Database) against which over-representation is tested)

## Evaluation signals

- The MMS run yields exactly two significant pathways at FDR < 0.05: tryptophan metabolism and beta-alanine metabolism
- The unmerged (no-MMS) run yields zero significant pathways at FDR < 0.05 despite three relevant hits at p < 0.05
- The MMS run reports seven relevant pathways (p < 0.05), matching the paper's reported enrichment outcome
- The comparison table contains a p-value and FDR for every pathway from both runs, arranged side-by-side
- Both runs use identical MetaboAnalyst 5.0 / SMPDB settings and the same 'found in at least two sources' filter, differing only in MMS harmonization

## Limitations

- Only compounds found in at least two information sources are retained before enrichment, so pathway coverage depends on the filtering threshold
- Enrichment results depend on the MetaboAnalyst 5.0 version and the SMPDB pathway universe at the time of the run; changes there can break exact reproduction
- The contrast is only interpretable if the merging step (InChIKey-based MMS harmonization) is the sole difference between the two compound lists

## Evidence

- [abstract] MMS hid pathways finding: "MMS facilitated the identification of significant pathways hidden when no dataset merging strategy was followed"
- [abstract] standardization rationale: "This study highlights the need for standardized and unified metabolite datasets to enhance the reproducibility and comparability of metabolomics studies."
- [abstract] identifier sources: "non-systematic identifiers (PubChem, CheBI, HMDB, KEGG, LipidMaps, DrugBank, Bin ID and CAS number)"
