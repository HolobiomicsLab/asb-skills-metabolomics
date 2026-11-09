---
name: central-versus-peripheral-metabolite-classification
description: Use when interpreting ML predictor features for a metabolite and you
  must determine whether that metabolite is in central vs peripheral metabolism based
  on topological proximity of predictors to its reactions in Recon3D.
when_to_use_negative:
- When no network model exists for the organism — Recon3D covers human metabolism,
  and the CCLE-derived models are not transferrable to in vivo conditions or other
  cell line panels
- When predictor features cannot be mapped to Recon3D reactions (e.g., non-metabolic
  genes absent from the Recon3D gene filter)
- When you need causal/mechanistic validation rather than 'guilt by association' topological
  proximity, which is descriptive
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: COBRA toolbox
  role: Identifies all Recon3D reactions associated with each relevant metabolite
    for the topological overlap computation
- name: Python
  role: Runs the recon_mapping scripts that extract genes from metabolite reactions
    and match them with top feature lists
  repo: sriram-lab/Metab8D
- name: Recon3D genome-scale metabolic model
  role: Provides the reaction network and gene-reaction associations used to define
    direct and first-degree-neighbor overlap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:4d6402502fb9edc8c710f79a5f882c53c7cfe097489ddff25b17ee69a5ddd832
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - COBRA toolbox
  - Python
  - Recon3D genome-scale metabolic model
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# central-versus-peripheral-metabolite-classification

## Summary

Classifies metabolites as central vs peripheral in the Recon3D network by testing whether a metabolite's top ML predictor features map within one step of its associated reactions, then stratifying overlap statistics (e.g., 50% vs 85% signaling-enzyme predictors) by position.

## When to use

Use when interpreting ML predictor features for a metabolite and you must determine whether that metabolite is in central vs peripheral metabolism based on topological proximity of predictors to its reactions in Recon3D.

## When NOT to use

- When no network model exists for the organism — Recon3D covers human metabolism, and the CCLE-derived models are not transferrable to in vivo conditions or other cell line panels
- When predictor features cannot be mapped to Recon3D reactions (e.g., non-metabolic genes absent from the Recon3D gene filter)
- When you need causal/mechanistic validation rather than 'guilt by association' topological proximity, which is descriptive

## Inputs

- Metab8D_network.xlsx top-20 feature lists with confidence scores (0-8) for each metabolite model
- Recon3D genome-scale metabolic model (with human_1_mapping.csv Recon3D-to-Human1 ID map)
- recon_mapping csv translating common gene names to BiggIDs
- Metabolite list (225-metabolite CCLE panel)

## Outputs

- overlap_statistics.csv with per-metabolite fraction of top predictors overlapping directly associated reactions and first-degree neighbors
- Central vs peripheral stratification of overlap by predictor type (signaling enzymes vs other features)
- Per-metabolite central/peripheral classification label

## How to apply

First use COBRA toolbox functions to enumerate all Recon3D reactions associated with each metabolite of interest, then map each top feature (from CNV, methylation, transcriptomics, or proteomics RF models) to its corresponding Recon3D reaction via the recon_mapping scripts and the common-gene-name-to-BiggID csv. Remove common metabolic cofactors (ATP, ADP, water, hydrogen, phosphate, diphosphate) so neighboring reactions are not overrepresented, then compute the fraction of mapped features overlapping the metabolite's directly associated reactions and their first-degree neighbors. Stratify overlap by metabolite position (central vs peripheral) and predictor type: peripheral metabolites show high overlap with enzyme-level predictors, while central metabolites require combinatorial predictors from signaling and redox pathways. The expected baseline is 92-96% of top-5 features per metabolite lying within one step of the metabolite's reactions.

## Related tools

- **COBRA toolbox** (Identifies all Recon3D reactions associated with each relevant metabolite for the topological overlap computation)
- **Python** (Runs the recon_mapping scripts that extract genes from metabolite reactions and match them with top feature lists) — sriram-lab/Metab8D
- **Recon3D genome-scale metabolic model** (Provides the reaction network and gene-reaction associations used to define direct and first-degree-neighbor overlap)

## Examples

```
import cobra; model = cobra.io.read_sbml_model('Recon3D_model.xml'); rxns = model.metabolites.get_by_id('atp_c').reactions
```

## Evaluation signals

- Overall top-5 feature overlap with directly associated reactions or first-degree neighbors falls in the 92-96% range across the four omic layers
- Central metabolites show higher reliance on signaling-enzyme predictors (50% vs 85% contrast) while peripheral metabolites skew toward corresponding enzyme levels (31% vs 12.5% contrast)
- Every top feature mapped via the BiggID translation table resolves to a valid Recon3D reaction; unmappable features are excluded explicitly
- Cofactor removal step confirms ATP, ADP, water, hydrogen, phosphate, and diphosphate are excluded before neighbor counting, so overlap is not inflated by shared cofactor reactions
- Results reconcile with RF model confidence scores from Metab8D_network.xlsx (features appearing in top 20 across 8 control experiments)

## Limitations

- The 'guilt by association' strategy may not identify causative mechanisms; associations from systems-level data are descriptive
- The 225-metabolite panel is only a fraction of cellular metabolites across compartments, limiting coverage of the classification
- Analysis was done in cancer cell lines in culture and may not reflect metabolism in vivo; models are not directly transferrable to other sources due to in vitro/in vivo differences and limited feature overlap
- Common cofactors must be removed or first-degree-neighbor overlap is overrepresented

## Evidence

- [discussion] COBRA toolbox reactions: "Functions from the COBRA toolbox allowed for the identification of all reactions associated with the relevant metabolites"
- [discussion] cofactor removal: "Common metabolic cofactors were removed to avoid overrepresenting neighboring reactions, including ATP, ADP, water, hydrogen, phosphate, and diphosphate"
- [abstract] central vs peripheral predictability: "Peripheral metabolites are predictable via levels of corresponding enzymes, while those in central metabolism require combinatorial predictors in signaling and redox pathways"
