---
name: joint-pathway-enrichment-analysis
description: Use when a curated metabolite-protein subnetwork (e.g. from PIUMet Steiner
  forest analysis of high-contribution m/z peaks) must be mapped onto metabolic pathways
  to find disease-relevant pathways.
when_to_use_negative:
- Input is only a peak table or raw m/z list that has not yet been annotated and network-reconstructed
  via PIUMet — annotate peaks and run the Steiner forest step first
- Input is a single analyte type only (metabolites or proteins alone) where a standard
  over-representation analysis, not joint metabolite-protein pathway enrichment, is
  appropriate
edam_operation: http://edamontology.org/operation_3939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0602
tools:
- name: MetaboAnalyst
  role: Web platform performing joint pathway enrichment on the network's metabolites
    and proteins, with FDR < 0.05 as the significance cutoff
- name: PIUMet
  role: Upstream tool that constructs the disease-related metabolite-protein subnetwork
    (prize-collecting Steiner Forest) whose nodes are the enrichment input
- name: DeepMSProfiler
  role: Upstream deep-learning pipeline whose ensemble RISE feature heatmap identifies
    the high-contribution m/z peaks that seed the annotated metabolites entering the
    network
  repo: https://github.com/yjdeng9/DeepMSProfiler
- name: Python (>= 3.6)
  role: Environment for running DeepMSProfiler feature extraction and saving/submitting
    the network node lists
  repo: https://github.com/yjdeng9/DeepMSProfiler
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51433-3
    title: An end-to-end deep learning method for mass spectrometry data analysis
      to reveal disease-specific metabolic profiles
  source_content_hash: sha256:03f6150665973b75d23cd43d20855ee40a279448c98ea5bf42bb81899f74a55d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yjdeng9/DeepMSProfiler
  repo_url: https://github.com/yjdeng9/DeepMSProfiler
  tools:
  - DeepMSProfiler
  - MetaboAnalyst
  - PIUMet
  - Python (>= 3.6)
  edam_operation: http://edamontology.org/operation_3939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# joint-pathway-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Joint pathway enrichment that submits metabolites and proteins from a reconstructed disease metabolite-protein network to MetaboAnalyst, identifying metabolic pathways significantly perturbed in disease (here LUAD) at FDR < 0.05.

## When to use

Use when a curated metabolite-protein subnetwork (e.g. from PIUMet Steiner forest analysis of high-contribution m/z peaks) must be mapped onto metabolic pathways to find disease-relevant pathways.

## When NOT to use

- Input is only a peak table or raw m/z list that has not yet been annotated and network-reconstructed via PIUMet — annotate peaks and run the Steiner forest step first
- Input is a single analyte type only (metabolites or proteins alone) where a standard over-representation analysis, not joint metabolite-protein pathway enrichment, is appropriate

## Inputs

- Disease-specific metabolite-protein network from PIUMet (luad_metabolite_protein_network.tsv; 82 proteins and 121 metabolites)
- Annotated metabolite identifiers (HMDB v3) derived from high-contribution m/z peaks (contribution score > 0.70)
- Protein identifiers from the iRefIndex v13 interaction network

## Outputs

- Table of enriched metabolic pathways with FDR-adjusted p-values (retained at FDR < 0.05)
- Panel visualizations of the reconstructed network and enriched pathways (Fig. 5g–l style panel set)

## How to apply

First reconstruct the disease-specific metabolite-protein network (e.g. the LUAD network of 82 proteins and 121 metabolites extracted by PIUMet from annotated high-contribution m/z peaks, saved as luad_metabolite_protein_network.tsv). Then submit the combined list of network metabolites and proteins to MetaboAnalyst joint pathway enrichment, which jointly tests over-representation of both analyte types against metabolic pathway databases. Retain only pathways passing FDR < 0.05 as the significant disease-related pathway set. Finally, visualize the reconstructed network together with the enriched pathways as a panel set (corresponding to Fig. 5g–l in the source article). The rationale is that joint analysis of metabolites and proteins captures pathway perturbations that neither analyte type alone reveals, connecting the deep-learning-identified metabolic signals to interpretable biology.

## Related tools

- **MetaboAnalyst** (Web platform performing joint pathway enrichment on the network's metabolites and proteins, with FDR < 0.05 as the significance cutoff)
- **PIUMet** (Upstream tool that constructs the disease-related metabolite-protein subnetwork (prize-collecting Steiner Forest) whose nodes are the enrichment input)
- **DeepMSProfiler** (Upstream deep-learning pipeline whose ensemble RISE feature heatmap identifies the high-contribution m/z peaks that seed the annotated metabolites entering the network) — https://github.com/yjdeng9/DeepMSProfiler
- **Python (>= 3.6)** (Environment for running DeepMSProfiler feature extraction and saving/submitting the network node lists) — https://github.com/yjdeng9/DeepMSProfiler

## Examples

```
run_feature(job_dir='DeepMSProfiler/example/out/jobs007'); # then submit luad_metabolite_protein_network.tsv metabolites/proteins to MetaboAnalyst joint pathway enrichment, retain FDR < 0.05
```

## Evaluation signals

- The reconstructed network contains exactly 82 proteins and 121 metabolites before enrichment is attempted
- All reported enriched pathways satisfy FDR < 0.05
- Enrichment input node lists match the metabolites (HMDB v3 annotated) and proteins (iRefIndex v13) present in luad_metabolite_protein_network.tsv
- Visualized panels correspond to the network plus enriched pathways panel set (Fig. 5g–l equivalent)
- High-contribution peaks feeding annotation all have ensemble RISE contribution score > 0.70

## Limitations

- Annotations depend on HMDB v3 and Recon2 coverage; unannotated metabolite signals that DeepMSProfiler can retain may still drop out at the annotation/network step
- MetaboAnalyst is a web service, so enrichment is not fully scriptable end-to-end and results depend on its current pathway database versions
- Pathway significance is threshold-based (FDR < 0.05); borderline pathways near the cutoff are not retained

## Evidence

- [full_text] PIUMet constructs disease-related metabolite-protein networks from input m/z peaks using the prize-collecting Steiner Forest algorithm, integrating iRefIndex (v13), HMDB (v3) and Recon2 databases: [evidence span withheld — anchor status: not_found]
- [full_text] MetaboAnalyst joint pathway enrichment at FDR < 0.05: [evidence span withheld — anchor status: not_found]
- [full_text] Network size validation of 82 proteins and 121 metabolites: [evidence span withheld — anchor status: not_found]
