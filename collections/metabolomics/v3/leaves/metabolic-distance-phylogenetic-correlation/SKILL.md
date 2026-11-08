---
name: metabolic-distance-phylogenetic-correlation
description: Use when you have draft GEMs of related isolates from multiple reconstruction
  pipelines and need to know whether structural differences between the models track
  organism phylogeny rather than pipeline artifacts.
when_to_use_negative:
- Your goal is functional community simulation or cross-feeding analysis of gap-filled
  models — use SMETANA/COMMIT downstream analysis instead of structural distance comparison
- Models come from a single reconstruction pipeline only — the substantial structural
  differences this skill measures arise across the four pipelines, so within-pipeline
  comparisons are not informative
- Identifier translation to a common namespace is impossible or largely fails — without
  MNXref normalization the distance measures reflect ID-space differences, not biology
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: COBRA toolbox
  role: Reads SBML draft reconstructions with readCbModel and provides the Matlab
    environment for computing the eight distance measures and STATIS compromise matrices
  repo: https://github.com/opencobra/cobratoolbox
- name: COMMIT (pwendering/COMMIT)
  role: Provides createMnxrefTranslationTables and translateGeneIDs for normalizing
    metabolite/reaction/gene identifiers to MNXref before distance computation
  repo: https://github.com/pwendering/COMMIT
- name: KBase
  role: One of the four automated reconstruction pipelines whose draft outputs are
    structurally compared
- name: CarveMe
  role: One of the four automated reconstruction pipelines whose draft outputs are
    structurally compared
  repo: https://github.com/cdanielmachado/carveme
- name: RAVEN 2.0
  role: One of the four automated reconstruction pipelines whose draft outputs are
    structurally compared
  repo: https://github.com/SysBioChalmers/RAVEN
- name: AuReMe / Pathway Tools
  role: One of the four automated reconstruction pipelines whose draft outputs are
    structurally compared
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:0ec04e071bae29336a767967a4b9bd0573c8c7491f02d720dc153b306d3437f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opencobra/cobratoolbox
  tools:
  - AuReMe
  - AuReMe / Pathway Tools
  - COBRA toolbox
  - COMMIT (pwendering/COMMIT)
  - CarveMe
  - KBase
  - Pathway Tools
  - RAVEN 2.0
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# metabolic-distance-phylogenetic-correlation

## Summary

Quantifies structural differences among draft genome-scale metabolic reconstructions (e.g., from KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools) using eight pairwise distance measures combined into STATIS compromise matrices, then tests biological relevance by correlating them with 16S rRNA sequence (phylogenetic) distance.

## When to use

Use when you have draft GEMs of related isolates from multiple reconstruction pipelines and need to know whether structural differences between the models track organism phylogeny rather than pipeline artifacts.

## When NOT to use

- Your goal is functional community simulation or cross-feeding analysis of gap-filled models — use SMETANA/COMMIT downstream analysis instead of structural distance comparison
- Models come from a single reconstruction pipeline only — the substantial structural differences this skill measures arise across the four pipelines, so within-pipeline comparisons are not informative
- Identifier translation to a common namespace is impossible or largely fails — without MNXref normalization the distance measures reflect ID-space differences, not biology

## Inputs

- Draft genome-scale metabolic reconstructions in SBML format from four pipelines (KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools), one per isolate (432 At-SPHERE genomes)
- MNXref reference files: reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, chem_xref.tsv
- 16S rRNA sequence (phylogenetic) distance matrix for the isolates

## Outputs

- Matlab workspaces (.mat) of normalized reconstructions stored per data/models/<habitat>/<method>
- Eight pairwise distance matrices over the reconstruction set
- STATIS compromise matrices combining the distance measures
- Correlations (e.g., Spearman rho) between each structural distance measure and 16S rRNA distance

## How to apply

Run the automated reconstruction pipelines (KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools) on all genomes and apply technical adaptations converting every draft to a common format. Read each pipeline's SBML models with the COBRA toolbox function readCbModel, collect them into cell arrays, and save them as Matlab workspaces (.mat) under data/models/<habitat>/<method>. Translate metabolite, reaction, and gene identifiers to a common MNXref namespace using the reference files reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, chem_xref.tsv via createMnxrefTranslationTables, replacing MNXM01 with MNXM1 (both encode H+) and ignoring generic compartments; where pipelines used different gene IDs, map them one-to-one via BLAST using translateGeneIDs. Compute the eight pairwise distance measures between the normalized reconstructions and combine them into STATIS compromise matrices. Finally, correlate each distance measure (including Jaccard distance) with the 16S rRNA sequence distance across the isolates; in the At-SPHERE study Jaccard distances correlated significantly with phylogeny (0.63–0.75, mean 0.70, p < 0.001), establishing the measures as biologically relevant.

## Related tools

- **COBRA toolbox** (Reads SBML draft reconstructions with readCbModel and provides the Matlab environment for computing the eight distance measures and STATIS compromise matrices) — https://github.com/opencobra/cobratoolbox
- **COMMIT (pwendering/COMMIT)** (Provides createMnxrefTranslationTables and translateGeneIDs for normalizing metabolite/reaction/gene identifiers to MNXref before distance computation) — https://github.com/pwendering/COMMIT
- **KBase** (One of the four automated reconstruction pipelines whose draft outputs are structurally compared)
- **CarveMe** (One of the four automated reconstruction pipelines whose draft outputs are structurally compared) — https://github.com/cdanielmachado/carveme
- **RAVEN 2.0** (One of the four automated reconstruction pipelines whose draft outputs are structurally compared) — https://github.com/SysBioChalmers/RAVEN
- **AuReMe / Pathway Tools** (One of the four automated reconstruction pipelines whose draft outputs are structurally compared)

## Examples

```
models{i}=readCbModel(sprintf('data/models/%s/%s/model%d.xml',habitat,method,i)); % then translate IDs via createMnxrefTranslationTables and compute pairwise distances
```

## Evaluation signals

- Jaccard distance vs 16S rRNA distance correlations fall in the reported range 0.63–0.75 with mean 0.70 (p < 0.001) across the 432 isolates
- Average STATIS compromise distance reproduces ~0.64 (range 0.54–0.72)
- Per-measure correlations reproduce the reported rhos: SVD/cofactor ~0.25 and dead-end metabolites ~0.04
- All MNXM01 identifiers replaced with MNXM1 and generic compartments ignored before distance computation
- Each pipeline's SBML loads via readCbModel without errors and .mat workspaces exist under data/models/<habitat>/<method>

## Limitations

- Definitions of all eight distance measures are deferred to supplementary S1 Text, not the main text
- MNXref/identifier translation is only done 'whenever possible'; lack of reaction cross-references to other namespaces hampers comparisons against reference models
- 16S rRNA sequences used for the phylogenetic distance were provided personally (R. Garrido-Oter) rather than via a public accession
- Gene identifier harmonization relies on one-to-one BLAST mapping, which may fail for genes without clear orthologs across pipeline ID schemes

## Evidence

- [other] eight distance measures used: [evidence span withheld — anchor status: not_found]
- [discussion] identifier translation: "Reaction and metabolite identifiers were translated to MNXref identifiers using the MNXref reference files for metabolites and reactions."
- [intro] structural comparison finding: "The structural comparison revealed substantial differences across the draft genome-scale metabolic reconstructions generated by the four approaches"
- [other] Jaccard–phylogeny correlation: [evidence span withheld — anchor status: not_found]
