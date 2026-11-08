---
name: genome-scale-metabolic-network-reconstruction
description: Use when starting from isolate genome assemblies/annotations (e.g., 432
  At-SPHERE Arabidopsis root isolate genomes) and you need draft genome-scale metabolic
  models (SBML) from automated pipelines, or multiple draft reconstructions that must
  be harmonized, gap-filled, or merged into a consensus.
when_to_use_negative:
- The models are already curated, gap-filled, and cross-referenced to external namespaces
  — this skill targets draft reconstructions, and comparison against reference models
  is hampered when identifiers lack cross-references
- Input is 16S rRNA sequence data or community abundance profiles without isolate
  genomes — reconstruction requires genome-scale input
- Only a single organism with a manually curated reference reconstruction exists —
  consensus generation across multiple automated pipelines is unnecessary
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: KBase
  role: Fully automated draft reconstruction pipeline (one of four applied to all
    At-SPHERE genomes)
- name: CarveMe
  role: Fully automated draft reconstruction pipeline; models subsequently stripped
    of the reconstruction medium before gap filling
  repo: https://github.com/cdanielmachado/carveme
- name: RAVEN 2.0
  role: MATLAB-based semi-automated reconstruction pipeline using KEGG/MetaCyc with
    gap-filling and quality control; KEGG-based reconstructions use KEGG Orthology
    MSA searches
  repo: https://github.com/SysBioChalmers/RAVEN
- name: AuReMe / Pathway Tools
  role: Automated draft reconstruction pipeline (fourth approach)
  repo: http://aureme.genouest.org/
- name: COBRA toolbox
  role: v3.0 MATLAB framework used to read SBML drafts (readCbModel), save .mat workspaces,
    and run LP-based gap filling (IBM CPLEX v12.9 tested)
  repo: https://github.com/opencobra/cobratoolbox
- name: COMMIT
  role: Community- and permeability-aware gap-filling of the consensus reconstructions
    for the Bulgarelli and Schlaeppi soil communities
  repo: https://github.com/pwendering/COMMIT
- name: memote
  role: Test suite used to assess the quality of the resulting functional models
- name: SMETANA
  role: Downstream analysis of gap-filled community models to predict directed metabolic
    interactions (helpers/beneficiaries)
  repo: https://github.com/cdanielmachado/smetana
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pcbi.1009906
    title: 'COMMIT: Consideration of metabolite leakage and community composition
      improves microbial community reconstructions'
  source_content_hash: sha256:12af5abffa1f22c04ab7b4d3ce622100b74207e820a7d9b9c57ab68e59dde21d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cdanielmachado/carveme
  tools:
  - AuReMe
  - AuReMe / Pathway Tools
  - COBRA toolbox
  - COMMIT
  - CarveMe
  - KBase
  - PathModel
  - Pathway Tools
  - RAVEN 2.0
  - SMETANA
  - memote
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 30
---

# genome-scale-metabolic-network-reconstruction

## Summary

Automated genome-scale metabolic reconstruction from isolate genomes using multiple pipelines (KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools), normalization to a common namespace via MNXref, and community-dependent gap filling (COMMIT) followed by consensus generation and memote-based quality assessment.

## When to use

Use when starting from isolate genome assemblies/annotations (e.g., 432 At-SPHERE Arabidopsis root isolate genomes) and you need draft genome-scale metabolic models (SBML) from automated pipelines, or multiple draft reconstructions that must be harmonized, gap-filled, or merged into a consensus.

## When NOT to use

- The models are already curated, gap-filled, and cross-referenced to external namespaces — this skill targets draft reconstructions, and comparison against reference models is hampered when identifiers lack cross-references
- Input is 16S rRNA sequence data or community abundance profiles without isolate genomes — reconstruction requires genome-scale input
- Only a single organism with a manually curated reference reconstruction exists — consensus generation across multiple automated pipelines is unnecessary

## Inputs

- Isolate genome assemblies/annotations (e.g., 432 At-SPHERE Arabidopsis thaliana isolate genomes)
- Pipeline-specific genome FASTQ/annotation inputs consumed by KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools
- MNXref reference files: reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, chem_xref.tsv
- Community composition definitions for the two soil communities (Bulgarelli, Schlaeppi)
- Minimal medium definition (intersection of KBase auxotrophic media, excluding amino acids and carbon sources except glucose)

## Outputs

- Draft genome-scale metabolic reconstructions in SBML format, one per genome per pipeline
- Matlab workspaces (.mat) of models read via readCbModel, organized under data/models/<habitat>/<method>
- MNXref-normalized models with translated metabolite/reaction/gene identifiers
- Gap-filled community models (COMMIT) with reduced added-reaction sets versus individual gap filling
- Consensus reconstruction (smaller than the sum of drafts) and memote quality assessment results

## How to apply

Run four fully automated reconstruction pipelines (KBase, CarveMe, RAVEN 2.0, AuReMe/Pathway Tools) on the input genomes with few parameters, then apply technical adaptations to convert all draft reconstructions to a common SBML format. Read each pipeline's SBML models with the COBRA toolbox function readCbModel, collect them into cell arrays, and save them as Matlab workspaces (.mat) organized per habitat and method (data/models/<habitat>/<method>). Translate metabolite, reaction, and gene identifiers to MNXref using reac_prop.tsv, reac_xref.tsv, chem_prop.tsv, and chem_xref.tsv (createMnxrefTranslationTables), replacing MNXM01 with MNXM1 (both encode H+) and ignoring generic compartments; where pipelines use different gene IDs, apply one-to-one BLAST mapping (translateGeneIDs). For community context, gap-fill with COMMIT, which respects community composition and metabolite permeability (Lipinski-style thresholds: HBD≤5, HBA≤10, MW≤500 Da, TPSA≤140 Å, RB≤10, −0.4≤XLogP3≤5.6) and uses weights such as transport=100, non-transport metabolic=50, directionality change=25, exchange=10^5; remove the CarveMe reconstruction medium first so gap filling starts from a minimal medium. Assess resulting functional models with the memote test suite, and optionally build a consensus by matching metabolite, reaction, and gene identifiers, which yields a model smaller than the sum of the drafts but of higher quality against reference models.

## Related tools

- **KBase** (Fully automated draft reconstruction pipeline (one of four applied to all At-SPHERE genomes))
- **CarveMe** (Fully automated draft reconstruction pipeline; models subsequently stripped of the reconstruction medium before gap filling) — https://github.com/cdanielmachado/carveme
- **RAVEN 2.0** (MATLAB-based semi-automated reconstruction pipeline using KEGG/MetaCyc with gap-filling and quality control; KEGG-based reconstructions use KEGG Orthology MSA searches) — https://github.com/SysBioChalmers/RAVEN
- **AuReMe / Pathway Tools** (Automated draft reconstruction pipeline (fourth approach)) — http://aureme.genouest.org/
- **COBRA toolbox** (v3.0 MATLAB framework used to read SBML drafts (readCbModel), save .mat workspaces, and run LP-based gap filling (IBM CPLEX v12.9 tested)) — https://github.com/opencobra/cobratoolbox
- **COMMIT** (Community- and permeability-aware gap-filling of the consensus reconstructions for the Bulgarelli and Schlaeppi soil communities) — https://github.com/pwendering/COMMIT
- **memote** (Test suite used to assess the quality of the resulting functional models)
- **SMETANA** (Downstream analysis of gap-filled community models to predict directed metabolic interactions (helpers/beneficiaries)) — https://github.com/cdanielmachado/smetana

## Examples

```
git clone https://github.com/pwendering/COMMIT && pip install smetana
```

## Evaluation signals

- Each pipeline's draft must load via readCbModel into .mat workspaces and pass MNXref translation without unresolved identifiers; MNXM01 must be replaced by MNXM1
- Consensus reconstruction should be considerably smaller than the sum of the number of reactions, metabolites, and genes of the underlying drafts
- COMMIT gap filling should decrease the number of added reactions compared to gap filling each reconstruction individually, without affecting genomic support
- Reproduction of reported statistics: average compromise distance 0.64 (range 0.54–0.72) and Jaccard–16S rRNA correlations of 0.63–0.75 (mean 0.70, p < 0.001) across the 432 isolates
- memote test suite results and model sensitivity/precision against reference models (true negatives cannot be determined, so only sensitivity and precision are valid)

## Limitations

- Gap-filling penalty weights were tuned ad hoc until satisfying results for selected reconstructions; no systematic weight-selection procedure is documented, and the full COMMIT procedure is deferred to supplementary S1 Text
- Molecular properties are sparse: only 26% of InChI keys mapped to PubChem CIDs and only 11% (11894) had XLogP3 values, so metabolites lacking properties are conservatively classified as not likely permeable
- Quality comparison to reference models is hampered by lack of reaction cross-references to other namespaces; translations were only performed 'whenever possible', and only sensitivity and precision (not specificity) can be computed
- 16S rRNA sequences used for phylogenetic validation were provided personally (Ruben Garrido Oter) and are not a public accession

## Evidence

- [intro] Four automated reconstruction pipelines applied to all genomes: "we applied four widely-used, fully automated approaches for metabolic network reconstruction relying on few parameters"
- [intro] Draft reconstructions converted to common format: "converted the draft reconstructions generated by the four approaches to a common format by multiple technical adaptations"
- [discussion] MNXref identifier translation: "Reaction and metabolite identifiers were translated to MNXref identifiers using the MNXref reference files for metabolites and reactions."
- [discussion] At-SPHERE genome source: "The 432 genomes that were used for genome-scale metabolic model reconstruction were downloaded from At-SPHERE (http://www.at-sphere.com) [38]."
- [intro] memote quality assessment: [evidence span withheld — anchor status: not_found]
- [readme] Consensus by identifier matching; four pipeline sources: "The consensus reconstructions were generated by merging draft reconstructions from four different approaches"
- [intro] COMMIT gap filling reduces added reactions: "COMMIT decreased the number of added reactions compared to the individual gap filling of the reconstructions"
