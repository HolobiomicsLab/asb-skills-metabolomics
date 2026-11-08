---
name: metabolic-reconstruction-merging
description: Use when multiple automatically generated draft GEMs for the same isolates
  (e.g. from KBase, CarveMe, RAVEN 2.0, AuReMe) must be merged into a single consensus
  model and compared against curated reference models.
when_to_use_negative:
- When the goal is community-dependent gap filling rather than consensus generation
  — use COMMIT's gap-filling procedure instead, which considers metabolite permeability
  and community composition
- When a single curated reference model already exists for the organism and no merging
  across pipelines is required
- When reaction identifiers cannot be translated to a common namespace (comparison
  was hampered by lack of cross-references to other name spaces, with translations
  done only 'whenever possible')
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0602
tools:
- name: COBRA toolbox
  role: readCbModel and model loading/merging in MATLAB workflows
  repo: https://github.com/opencobra/cobratoolbox
- name: MetaNetX/MNXref
  role: cross-reference tables for translating metabolite and reaction identifiers
    to MNXref namespace
- name: RAVEN 2.0
  role: source of one of the four per-isolate draft reconstructions (KEGG-based pipeline
    with MSA for KO sequences)
  repo: https://github.com/SysBioChalmers/RAVEN
- name: CarveMe
  role: source of one of the four per-isolate draft reconstructions
  repo: https://github.com/cdanielmachado/carveme
- name: AuReMe/Pathway Tools
  role: source of one of the four per-isolate draft reconstructions
  repo: http://aureme.genouest.org/
- name: KBase
  role: source of one of the four per-isolate draft reconstructions
- name: BLAST/bedtools
  role: one-to-one gene mapping across annotation pipelines before combining GPR rules
- name: COMMIT pipeline code
  role: merge_metabolic_models.m and translateGeneIDs.m scripts in the pwendering/COMMIT
    repository
  repo: https://github.com/pwendering/COMMIT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:17926d26ecd57276c106dee36f66176a225ded8f6f37cb646d683035efa1fba9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - AuReMe/Pathway Tools
  - BLAST/bedtools
  - COBRA toolbox
  - COMMIT pipeline code
  - CarveMe
  - KBase
  - MetaNetX/MNXref
  - RAVEN 2.0
  - bedtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# metabolic-reconstruction-merging

## Summary

Builds consensus genome-scale metabolic reconstructions by merging per-isolate draft models from four automated pipelines (KBase, RAVEN 2.0, CarveMe, AuReMe/Pathway Tools) after translating all identifiers to the MNXref namespace, and evaluates the merged models against curated references. The consensus yields higher organism specificity and quality than any individual draft while remaining smaller than the sum of the underlying reconstructions.

## When to use

Use when multiple automatically generated draft GEMs for the same isolates (e.g. from KBase, CarveMe, RAVEN 2.0, AuReMe) must be merged into a single consensus model and compared against curated reference models.

## When NOT to use

- When the goal is community-dependent gap filling rather than consensus generation — use COMMIT's gap-filling procedure instead, which considers metabolite permeability and community composition
- When a single curated reference model already exists for the organism and no merging across pipelines is required
- When reaction identifiers cannot be translated to a common namespace (comparison was hampered by lack of cross-references to other name spaces, with translations done only 'whenever possible')

## Inputs

- Per-isolate draft genome-scale metabolic reconstructions from four approaches (KBase, RAVEN 2.0, CarveMe, AuReMe/Pathway Tools), in model files readable by COBRA readCbModel
- MetaNetX cross-reference tables: reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, chem_xref.tsv
- Genome/protein sequences for one-to-one BLAST gene-ID mapping across annotation pipelines
- Curated reference models iMZ1055 (B. megaterium) and iRP911 (M. extorquens)

## Outputs

- Per-isolate consensus models written to data/models/consensus
- Sensitivity and precision of consensus vs. individual drafts relative to reference models
- Counts of reactions, metabolites, and genes in consensus versus the sum across underlying drafts

## How to apply

Read each per-isolate draft with the COBRA toolbox function readCbModel and collect the four approaches' drafts into cell arrays saved as .mat workspaces. Translate metabolite and reaction identifiers to MNXref using MetaNetX cross-reference tables (reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, chem_xref.tsv), replacing MNXM01 with MNXM1 (both encode H+) and ignoring generic compartments. Match reactions across drafts by MNXref identifier matching combined with cosine-similarity reaction matching, and run code/model_generation/merge_metabolic_models.m to write per-isolate consensus models to data/models/consensus. Reconcile gene IDs across annotation pipelines by one-to-one BLAST mapping (batch-makeblastdb.sh, batch-blastp.sh/batch-blastx.sh with bedtools, then translateGeneIDs.m) so GPR rules can be combined. Compare consensus and draft contents against curated reference models iMZ1055 (B. megaterium) and iRP911 (M. extorquens); since true negatives cannot be determined, compute only sensitivity and precision.

## Related tools

- **COBRA toolbox** (readCbModel and model loading/merging in MATLAB workflows) — https://github.com/opencobra/cobratoolbox
- **MetaNetX/MNXref** (cross-reference tables for translating metabolite and reaction identifiers to MNXref namespace)
- **RAVEN 2.0** (source of one of the four per-isolate draft reconstructions (KEGG-based pipeline with MSA for KO sequences)) — https://github.com/SysBioChalmers/RAVEN
- **CarveMe** (source of one of the four per-isolate draft reconstructions) — https://github.com/cdanielmachado/carveme
- **AuReMe/Pathway Tools** (source of one of the four per-isolate draft reconstructions) — http://aureme.genouest.org/
- **KBase** (source of one of the four per-isolate draft reconstructions)
- **BLAST/bedtools** (one-to-one gene mapping across annotation pipelines before combining GPR rules)
- **COMMIT pipeline code** (merge_metabolic_models.m and translateGeneIDs.m scripts in the pwendering/COMMIT repository) — https://github.com/pwendering/COMMIT

## Examples

```
matlab -r "m=readCbModel('data/models/drafts/isolate_KBase.mat'); merge_metabolic_models()"  % writes consensus to data/models/consensus
```

## Evaluation signals

- Consensus shows higher sensitivity against reference models iMZ1055 and iRP911 than any individual draft (higher organism specificity and quality)
- Precision of the consensus is lower than the drafts, an expected trade-off of merging
- Consensus reaction, metabolite, and gene counts are considerably smaller than the sum of counts across the four underlying drafts
- All reactions matched across drafts carry valid MNXref identifiers, with MNXM01 replaced by MNXM1 and generic compartments ignored
- Gene IDs are reconciled one-to-one across pipelines via BLAST so GPR rules combine and compare without identifier collisions

## Limitations

- Since the true negatives could not be determined, only sensitivity and precision could be computed — no full specificity or accuracy
- Comparison to reference models is hampered by the lack of reaction cross-references to other name spaces; translations to MNXref were done only 'whenever possible'
- The consensus reflects agreement among four fully automated approaches, so the structural differences across these drafts propagate into what the consensus captures

## Evidence

- [intro] The consensus generation consisted of matching metabolite, reaction, and gene identifiers: "The consensus generation consisted of matching metabolite, reaction, and gene identifiers"
- [intro] the consensus of metabolic reconstructions shows a higher organism specificity as well as a higher quality than the draft reconstructions: "the consensus of metabolic reconstructions shows a higher organism specificity as well as a higher quality than the draft reconstructions"
- [discussion] the metabolite ID ’MNXM01’ was replaced with ’MNXM1’ since both encode H+.: "the metabolite ID ’MNXM01’ was replaced with ’MNXM1’ since both encode H+."
- [discussion] Since the true negatives could not be determined, only sensitivity and precision were calculated as measures to assess the quality of model prediction.: "Since the true negatives could not be determined, only sensitivity and precision were calculated as measures to assess the quality of model prediction."
- [readme] The consensus reconstructions were generated by merging draft reconstructions from four different approaches:: "The consensus reconstructions were generated by merging draft reconstructions from four different approaches:"
