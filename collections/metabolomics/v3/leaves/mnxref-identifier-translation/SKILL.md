---
name: mnxref-identifier-translation
description: Use when SBML models reconstructed by different pipelines (KBase, CarveMe,
  RAVEN 2.0, AuReMe/Pathway Tools) carry incompatible self-defined identifiers and
  must be compared, merged into a consensus, or gap-filled against a shared MNXref
  database.
when_to_use_negative:
- Models already use a shared namespace (e.g., all MNXref) — retranslation adds no
  value
- Comparing models that rely on self-defined identifiers with no reaction cross-references
  to other name spaces — the article reports such comparisons are hampered since translations
  can only be done 'whenever possible'
- Single-pipeline analyses that never compare or merge reconstructions across sources
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: COBRA toolbox
  role: Provides readCbModel for loading SBML drafts and createMnxrefTranslationTables/translateGeneIDs.m
    for identifier translation in MATLAB
  repo: https://github.com/opencobra/cobratoolbox
- name: MetaNetX/MNXref
  role: Supplies the cross-reference TSV files (reac_prop.tsv, reac_xref.tsv, chem_prop.tsv,
    chem_xref.tsv) used as the mapping target namespace
- name: BLAST/bedtools
  role: One-to-one BLAST mapping (batch-makeblastdb.sh, batch-blastp.sh/batch-blastx.sh)
    reconciles gene identifiers across annotation pipelines before GPR merging
- name: RAVEN 2.0
  role: One of the four reconstruction pipelines whose KEGG-based SBML output requires
    translation to MNXref identifiers
  repo: https://github.com/SysBioChalmers/RAVEN
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:3a9915c7e4d3ed3d830495d93577e61973b615ba75da76c0fffde7a100f162e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - AuReMe
  - AuReMe/Pathway Tools
  - BLAST/bedtools
  - COBRA toolbox
  - COMMIT
  - CarveMe
  - Gurobi
  - HMMER
  - IBM CPLEX
  - KBase
  - MetaNetX/MNXref
  - Pathway Tools
  - RAVEN 2.0
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# mnxref-identifier-translation

## Summary

Translates metabolite, reaction, and gene identifiers from draft genome-scale metabolic reconstructions (SBML from KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools) into the common MetaNetX/MNXref namespace so models from different pipelines can be structurally compared or merged into a consensus.

## When to use

Use when SBML models reconstructed by different pipelines (KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools) carry incompatible self-defined identifiers and must be compared, merged into a consensus, or gap-filled against a shared MNXref database.

## When NOT to use

- Models already use a shared namespace (e.g., all MNXref) — retranslation adds no value
- Comparing models that rely on self-defined identifiers with no reaction cross-references to other name spaces — the article reports such comparisons are hampered since translations can only be done 'whenever possible'
- Single-pipeline analyses that never compare or merge reconstructions across sources

## Inputs

- Draft SBML metabolic reconstructions from KBase, CarveMe, RAVEN 2.0, and AuReMe/Pathway Tools (read via readCbModel)
- MNXref reference tables: reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, chem_xref.tsv (MetaNetX)
- Genome FASTA/protein files for one-to-one BLAST gene-ID mapping across annotation pipelines

## Outputs

- MNXref-normalized metabolite and reaction identifiers for each draft reconstruction
- One-to-one gene-ID mappings enabling combined GPR rules
- Matlab .mat workspaces (cell arrays) under data/models/<habitat>/<method> ready for consensus merging or distance computation

## How to apply

Read each pipeline's SBML model with the COBRA toolbox function readCbModel and build translation tables from the MetaNetX reference files reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, and chem_xref.tsv via createMnxrefTranslationTables. Apply the tables to map every metabolite and reaction to its MNXref identifier, replacing MNXM01 with MNXM1 because both encode H+, and ignoring generic compartments so cross-pipeline matching is not blocked by compartment labels. Where pipelines used different gene IDs, establish one-to-one gene mapping with BLAST (batch-makeblastdb.sh, batch-blastp.sh/batch-blastx.sh with bedtools) and apply translateGeneIDs.m so GPR rules can be reconciled. Save the normalized models as Matlab workspaces (.mat) cell arrays under data/models/<habitat>/<method> for downstream steps such as consensus merging (merge_metabolic_models.m), distance computation, or MNXref-based gap filling.

## Related tools

- **COBRA toolbox** (Provides readCbModel for loading SBML drafts and createMnxrefTranslationTables/translateGeneIDs.m for identifier translation in MATLAB) — https://github.com/opencobra/cobratoolbox
- **MetaNetX/MNXref** (Supplies the cross-reference TSV files (reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, chem_xref.tsv) used as the mapping target namespace)
- **BLAST/bedtools** (One-to-one BLAST mapping (batch-makeblastdb.sh, batch-blastp.sh/batch-blastx.sh) reconciles gene identifiers across annotation pipelines before GPR merging)
- **RAVEN 2.0** (One of the four reconstruction pipelines whose KEGG-based SBML output requires translation to MNXref identifiers) — https://github.com/SysBioChalmers/RAVEN

## Examples

```
createMnxrefTranslationTables('reac_prop.tsv','reac_xref.tsv','chem_prop.tsv','chem_xref.tsv')  % MATLAB, COBRA toolbox, then apply to readCbModel outputs
```

## Evaluation signals

- Proton identifier check: every occurrence of MNXM01 in the translated model is replaced by MNXM1 (both encode H+)
- Reaction sets from all four pipelines can be intersected by MNXref identifier plus cosine-similarity reaction matching without unmatched-database errors
- Gene translation yields one-to-one BLAST pairs; gene counts in the merged consensus are considerably smaller than the sum of the underlying drafts
- Translated .mat cell arrays load cleanly with readCbModel and support computation of the eight pairwise distance measures and STATIS compromise matrices
- Gap-filling against dbModel_MNXref_balanced succeeds only after translation, indicating namespace consistency

## Limitations

- Translations are only performed 'whenever possible'; models lacking reaction cross-references to external namespaces cannot be fully compared (only sensitivity and precision were computable, no true negatives)
- Generic compartments are ignored during translation, which may discard compartment-specific information
- Comparison quality depends on the currency/version of the MetaNetX cross-reference files used

## Evidence

- [discussion] Reaction and metabolite identifiers were translated to MNXref identifiers using the MNXref reference files for metabolites and reactions.: "Reaction and metabolite identifiers were translated to MNXref identifiers using the MNXref reference files for metabolites and reactions."
- [discussion] MNXM01 replaced with MNXM1: "the metabolite ID ’MNXM01’ was replaced with ’MNXM1’ since both encode H+."
- [intro] MetaNetX structural matching rationale: "the MetaNetX database are already structurally matched between various databases"
- [discussion] Comparison hampered by missing cross-references: "The comparison was hampered by the lack of reaction cross-references to name spaces other than self-defined identifiers."
