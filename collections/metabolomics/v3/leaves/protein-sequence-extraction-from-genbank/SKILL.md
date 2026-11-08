---
name: protein-sequence-extraction-from-genbank
description: Use when you have antiSMASH/gutSMASH output directories in GenBank format
  and need the predicted gene-cluster protein sequences as input for MASH MinHash
  redundancy filtering or BiG-SCAPE GCF grouping in BiG-MAP.
when_to_use_negative:
- Input gene clusters are already nonredundant and unique representatives are already
  chosen — MinHash sketching and medoid selection would only add redundancy filtering
  overhead
- The goal is to map metagenomic/metatranscriptomic reads or run differential abundance
  analysis (bigmap-map / bigmap-analyse) rather than build the nonredundant reference
  collection
- Input is raw genome FASTA rather than antiSMASH/gutSMASH output — the clusters must
  first be predicted (README states the module takes 'the output directories of any
  anti- or gutSMASH run')
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: BiG-MAP
  role: Hosts the bigmap-family module that performs the GenBank parsing, MASH sketching,
    medoid selection, and optional BiG-SCAPE GCF grouping
  repo: github.com/medema-group/BiG-MAP
- name: MASH
  role: MinHash-sketches the extracted protein sequences for fast pairwise similarity
    estimation used in the first redundancy-filtering step (default 0.8 cutoff)
- name: BiG-SCAPE
  role: Optional second redundancy-filtering round that groups representative gene
    clusters into GCFs using a 0.3 distance cutoff and the Pfam-A.hmm domain database
- name: antiSMASH
  role: Upstream predictor whose GenBank gene-cluster output directories supply the
    protein sequences to be extracted
- name: gutSMASH
  role: Upstream predictor of specialized primary metabolic gene clusters whose GenBank
    output directories supply the protein sequences to be extracted
  repo: victoriapascal/gutsmash
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00937-21
    title: 'BiG-MAP: an Automated Pipeline To Profile Metabolic Gene Cluster Abundance
      and Expression in Microbiomes'
  source_content_hash: sha256:dde96c42f938bfa6758bd3debf52504b5e607126d8846690d0da3069578f54ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiG-MAP
  - BiG-SCAPE
  - MASH
  - antiSMASH
  - gutSMASH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# protein-sequence-extraction-from-genbank

## Summary

Extracts protein sequences from antiSMASH/gutSMASH GenBank gene-cluster output directories so they can be sketched with MASH and clustered into nonredundant representative gene clusters within the BiG-MAP family module.

## When to use

Use when you have antiSMASH/gutSMASH output directories in GenBank format and need the predicted gene-cluster protein sequences as input for MASH MinHash redundancy filtering or BiG-SCAPE GCF grouping in BiG-MAP.

## When NOT to use

- Input gene clusters are already nonredundant and unique representatives are already chosen — MinHash sketching and medoid selection would only add redundancy filtering overhead
- The goal is to map metagenomic/metatranscriptomic reads or run differential abundance analysis (bigmap-map / bigmap-analyse) rather than build the nonredundant reference collection
- Input is raw genome FASTA rather than antiSMASH/gutSMASH output — the clusters must first be predicted (README states the module takes 'the output directories of any anti- or gutSMASH run')

## Inputs

- antiSMASH or gutSMASH output directories containing GenBank files of predicted gene clusters
- Optional: BiG-SCAPE installation path and processed Pfam-A.hmm database (hmmpressed) for the second redundancy-filtering round

## Outputs

- BiG-MAP.GCF.fna — reference FASTA of representative gene clusters for WGS read mapping
- BiG-MAP.GCF.bed — BED file to extract core regions in bigmap-map
- BiG-MAP.GCs.json — dictionary of MASH-based gene-cluster groups
- BiG-MAP.GCF.json — dictionary of BiG-SCAPE gene-cluster families

## How to apply

Load the output directories produced by antiSMASH or gutSMASH runs (bigmap-family accepts these directly as -D input dirs) and parse the GenBank files to extract the protein sequences of all predicted gene clusters per genome. Sketch the extracted sequences with MASH MinHash sketches for fast pairwise similarity estimation, then cluster at the -tg similarity threshold (default 0.8) to collapse redundant gene clusters; select the medoid (minimal cumulative distance) as each group's representative. Optionally apply a second redundancy-filtering round with BiG-SCAPE (requires a Pfam-A.hmm database processed with hmmpress) at a 0.3 distance cutoff to group representatives into GCFs. Write the nonredundant representative collection as BiG-MAP.GCF.fna (mapping reference), BiG-MAP.GCF.bed (core-region extraction), and the GCF dictionaries BiG-MAP.GCs.json and BiG-MAP.GCF.json.

## Related tools

- **BiG-MAP** (Hosts the bigmap-family module that performs the GenBank parsing, MASH sketching, medoid selection, and optional BiG-SCAPE GCF grouping) — github.com/medema-group/BiG-MAP
- **MASH** (MinHash-sketches the extracted protein sequences for fast pairwise similarity estimation used in the first redundancy-filtering step (default 0.8 cutoff))
- **BiG-SCAPE** (Optional second redundancy-filtering round that groups representative gene clusters into GCFs using a 0.3 distance cutoff and the Pfam-A.hmm domain database)
- **antiSMASH** (Upstream predictor whose GenBank gene-cluster output directories supply the protein sequences to be extracted)
- **gutSMASH** (Upstream predictor of specialized primary metabolic gene clusters whose GenBank output directories supply the protein sequences to be extracted) — victoriapascal/gutsmash

## Examples

```
bigmap-family -D /usr001/BiG-MAP_tutorial_genomes/ -b /usr001/BiG-SCAPE_location/ -pf /usr001/pfam_files_location/ -O /usr001/results_family/
```

## Evaluation signals

- All four expected outputs are produced (BiG-MAP.GCF.fna, BiG-MAP.GCF.bed, BiG-MAP.GCs.json, BiG-MAP.GCF.json) for a bigmap-family run with BiG-SCAPE
- BiG-MAP.GCs.json contains exactly one nonredundant representative per MASH similarity cluster, and each representative corresponds to the medoid (minimal cumulative MinHash distance) of its group
- Lowering -tg toward 0 yields more single-member families and raising it toward 1 yields one large family, confirming the threshold controls cluster granularity
- With -b and -pf supplied, BiG-MAP.GCF.json additionally contains BiG-SCAPE GCF assignments for the representative gene clusters

## Limitations

- BiG-MAP has 'only been tested on Ubuntu' per the README, so other platforms are unsupported
- antiSMASH/gutSMASH must be run beforehand; bigmap-family does not predict gene clusters itself
- The MASH 0.8 similarity cutoff is a default and can be changed via -tg; the BiG-SCAPE round is optional and requires downloading and hmmpressing the latest Pfam-A.hmm.gz from InterPro

## Evidence

- [full_text] The family module extracts protein sequences from the input gene-cluster GenBank files, sketches them with MASH to compute pairwise distances: [evidence span withheld — anchor status: not_found]
- [readme] README on module inputs: [evidence span withheld — anchor status: external_doc]
- [readme] README on the MASH cutoff: [evidence span withheld — anchor status: external_doc]
- [readme] README on BiG-SCAPE second round: [evidence span withheld — anchor status: external_doc]
- [readme] README on output files: [evidence span withheld — anchor status: external_doc]
- [intro] Article intro on MinHash filtering rationale: "BiG-MAP uses MinHash-based redundancy filtering to avoid ambiguous mapping, groups BGCs into gene cluster families (GCFs) with BiG-SCAPE (19)"
