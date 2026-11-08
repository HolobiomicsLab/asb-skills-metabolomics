---
name: reaction-matching-across-draft-reconstructions
description: Use when merging automatically generated draft GEMs from heterogeneous
  pipelines (KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools) whose metabolite/reaction/gene
  identifiers differ, to build a consensus reconstruction via MNXref translation and
  cosine-similarity reaction matching.
when_to_use_negative:
- The draft models already share a common identifier namespace and comparable GPRs,
  so direct content comparison is possible without MNXref translation and matching.
- The goal is gap filling or community interaction prediction rather than merging
  drafts — use COMMIT gap filling or SMETANA instead.
- The models to compare are curated reference models (e.g. iMZ1055, iRP911) lacking
  cross-references to other namespaces, which the paper notes hampers translation
  and comparison.
edam_topics:
- http://edamontology.org/topic_2259
- http://edamontology.org/topic_0102
tools:
- name: COBRA Toolbox v3.0
  role: readCbModel loads per-isolate draft reconstructions into .mat workspaces;
    MATLAB environment in which merge_metabolic_models.m runs
  repo: https://github.com/opencobra/cobratoolbox
- name: MetaNetX/MNXref
  role: Supplies the cross-reference tables (reac_prop.tsv, reac_xref.tsv, chem_prop.tsv,
    chem_xref.tsv) used to translate metabolite and reaction identifiers to MNXref
- name: COMMIT repository
  role: Hosts code/model_generation/merge_metabolic_models.m which writes the per-isolate
    consensus models to data/models/consensus, plus translateGeneIDs.m for gene-ID
    reconciliation
  repo: https://github.com/pwendering/COMMIT
- name: BLAST + bedtools
  role: One-to-one BLAST mapping (batch-makeblastdb.sh, batch-blastp.sh/batch-blastx.sh)
    to reconcile gene IDs across annotation pipelines before combining GPRs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:02c4252c078499798bac8883d5f9b505c408b0173e5000fb2505bc7452e2f93d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - AuReMe/Pathway Tools
  - BLAST + bedtools
  - COBRA Toolbox v3.0
  - COMMIT repository
  - CarveMe
  - KBase
  - MetaNetX/MNXref
  - RAVEN 2.0
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2259
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# reaction-matching-across-draft-reconstructions

## Summary

Merging metabolically redundant draft genome-scale reconstructions (e.g. from KBase, CarveMe, RAVEN 2.0, and AuReMe/Pathway Tools) into a single consensus model by translating all identifiers to the MNXref namespace and matching reactions via identifier equality plus cosine-similarity reaction matching, with gene IDs harmonized by one-to-one BLAST.

## When to use

Use when merging automatically generated draft GEMs from heterogeneous pipelines (KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools) whose metabolite/reaction/gene identifiers differ, to build a consensus reconstruction via MNXref translation and cosine-similarity reaction matching.

## When NOT to use

- The draft models already share a common identifier namespace and comparable GPRs, so direct content comparison is possible without MNXref translation and matching.
- The goal is gap filling or community interaction prediction rather than merging drafts — use COMMIT gap filling or SMETANA instead.
- The models to compare are curated reference models (e.g. iMZ1055, iRP911) lacking cross-references to other namespaces, which the paper notes hampers translation and comparison.

## Inputs

- Per-isolate draft GEMs from four approaches (KBase, RAVEN 2.0, CarveMe, AuReMe/Pathway Tools), read via readCbModel into .mat workspaces
- MetaNetX/MNXref cross-reference files: reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, chem_xref.tsv
- Genome/protein sequences per isolate for one-to-one BLAST gene-ID mapping (run with bedtools)

## Outputs

- Per-isolate consensus GEMs written to data/models/consensus
- Reaction equivalence mappings across the four drafts (MNXref ID matches plus cosine-similarity matches)
- Translated gene-ID tables enabling combined and comparable GPR rules

## How to apply

Read each per-isolate draft reconstruction from the four approaches with the COBRA toolbox function readCbModel, collecting them into cell arrays saved as .mat workspaces. Translate metabolite and reaction identifiers to MNXref using the MetaNetX cross-reference tables (reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, chem_xref.tsv), replacing MNXM01 with MNXM1 since both encode H+ and ignoring generic compartments. Match reactions across the four drafts by MNXref identifier matching combined with cosine-similarity reaction matching, then run code/model_generation/merge_metabolic_models.m to write per-isolate consensus models to data/models/consensus. Reconcile gene IDs across annotation pipelines via one-to-one BLAST mapping (batch-makeblastdb.sh with batch-blastp.sh/batch-blastx.sh and bedtools, then translateGeneIDs.m) so GPR rules can be combined. The rationale is that the four pipelines produce substantially different identifier namespaces, and consensus generation 'consisted of matching metabolite, reaction, and gene identifiers'.

## Related tools

- **COBRA Toolbox v3.0** (readCbModel loads per-isolate draft reconstructions into .mat workspaces; MATLAB environment in which merge_metabolic_models.m runs) — https://github.com/opencobra/cobratoolbox
- **MetaNetX/MNXref** (Supplies the cross-reference tables (reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, chem_xref.tsv) used to translate metabolite and reaction identifiers to MNXref)
- **COMMIT repository** (Hosts code/model_generation/merge_metabolic_models.m which writes the per-isolate consensus models to data/models/consensus, plus translateGeneIDs.m for gene-ID reconciliation) — https://github.com/pwendering/COMMIT
- **BLAST + bedtools** (One-to-one BLAST mapping (batch-makeblastdb.sh, batch-blastp.sh/batch-blastx.sh) to reconcile gene IDs across annotation pipelines before combining GPRs)

## Examples

```
matlab -batch "model = readCbModel('data/models/drafts/KBase/isolate.mat'); run('code/model_generation/merge_metabolic_models.m')"
```

## Evaluation signals

- The consensus reconstruction is considerably smaller than the sum of reactions, metabolites, and genes of the underlying four drafts.
- The proton is consistently represented under MNXM1 with MNXM01 replaced, so no duplicate H+ metabolite IDs remain in the merged model.
- Against curated references iMZ1055 (B. megaterium) and iRP911 (M. extorquens), the consensus shows higher sensitivity than the individual drafts.
- Every consensus reaction is traceable to at least one of the four drafts via MNXref ID match or cosine-similarity match.
- All gene IDs in merged GPRs map one-to-one via the BLAST-based translateGeneIDs.m step.

## Limitations

- MNXref translation is only performed 'whenever possible' because reference models like iMZ1055/iRP911 lack reaction cross-references to other namespaces.
- Only sensitivity and precision can be computed against reference models since true negatives cannot be determined.
- Identifier-matching alone may miss reactions whose MNXref IDs differ across pipelines; the cosine-similarity reaction matching step is required, and its thresholds are not fully specified in the main text (full procedure deferred to S1 Text).

## Evidence

- [intro] The consensus generation consisted of matching metabolite, reaction, and gene identifiers: "The consensus generation consisted of matching metabolite, reaction, and gene identifiers"
- [discussion] MNXM01 replaced with MNXM1: [evidence span withheld — anchor status: not_found]
- [discussion] MNXref identifier translation: "Reaction and metabolite identifiers were translated to MNXref identifiers using the MNXref reference files for metabolites and reactions."
- [readme] Consensus from four draft approaches: "The consensus reconstructions were generated by merging draft reconstructions from four different approaches:"
- [intro] Consensus smaller than sum of drafts: "the consensus reconstruction is considerably smaller and, thus, does not correspond to the sum of the number of reactions, metabolites, and genes"
- [discussion] Sensitivity and precision only: "Since the true negatives could not be determined, only sensitivity and precision were calculated as measures to assess the quality of model prediction."
