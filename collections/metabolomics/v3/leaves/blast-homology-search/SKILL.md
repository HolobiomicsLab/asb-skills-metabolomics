---
name: blast-homology-search
description: Use when mass spectrometry identifications must be linked to biosynthetic
  gene clusters (BGCs) in paired genomic data, e.g., validating molDiscovery molecule
  annotations on microbial strains by BLAST against MIBiG reference clusters.
when_to_use_negative:
- Input lacks paired genomic data — BLAST-based BGC linking requires genome mining
  annotations for the same strains as the mass spectra
- The goal is small molecule identification from spectra alone (use molDiscovery database
  search against PubChem/DNP instead of genome homology)
- Gene cluster candidates cannot meet the >=90% coverage and >=70% identity thresholds,
  e.g., highly divergent novel clusters where strict BLAST criteria would wrongly
  discard true links
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3172
tools:
- name: molDiscovery
  role: Produces the mass spectral molecule identifications whose linked gene clusters
    are cross-validated against genome mining via BLAST
  repo: github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:3d2440f09e4f1542859fe9b0152ffd813cd5a17024df6a7a3910e2e894544248
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# blast-homology-search

## Summary

Sequence homology search skill (BLAST-style alignment) used to cross-validate metabolite identifications against genome mining, linking identified small molecules to their biosynthetic gene clusters by comparing gene clusters to MIBiG references with coverage and identity thresholds.

## When to use

Use when mass spectrometry identifications must be linked to biosynthetic gene clusters (BGCs) in paired genomic data, e.g., validating molDiscovery molecule annotations on microbial strains by BLAST against MIBiG reference clusters.

## When NOT to use

- Input lacks paired genomic data — BLAST-based BGC linking requires genome mining annotations for the same strains as the mass spectra
- The goal is small molecule identification from spectra alone (use molDiscovery database search against PubChem/DNP instead of genome homology)
- Gene cluster candidates cannot meet the >=90% coverage and >=70% identity thresholds, e.g., highly divergent novel clusters where strict BLAST criteria would wrongly discard true links

## Inputs

- molDiscovery identification results for GNPS microbial tandem mass spectrometry datasets (46 datasets, 922 strains)
- MIBiG-linked biosynthetic gene cluster (BGC) annotations from strain genomes
- Nucleotide/protein sequences of molecule-linked gene clusters

## Outputs

- bgc_molecule_links.tsv — table of accepted molecule-to-BGC links with BLAST alignment coverage and percent identity values
- Set of known molecule-BGC links (19 expected) and putative novel BGC families (3 expected)

## How to apply

Load molDiscovery identification results for the GNPS microbial datasets and the MIBiG-linked BGC annotations for the corresponding strain genomes. For each identified molecule, run BLAST comparisons between its linked gene cluster and the MIBiG reference clusters to obtain alignment coverage and percent identity values. Accept a molecule-to-BGC link only when the BLAST alignment satisfies both criteria: coverage >= 90% and identity >= 70%. Compile accepted links into a linkage table (bgc_molecule_links.tsv); in the molDiscovery 46-dataset/922-strain metabologenomic evaluation this procedure recovers 19 known molecule-BGC links (e.g., xantholysin A/B/C, orfamide A/B/C, hygromycin A, yersiniabactin) plus 3 putative novel BGC families (dinghupeptins, lipopeptin/neopeptin, longicatenamycin).

## Related tools

- **molDiscovery** (Produces the mass spectral molecule identifications whose linked gene clusters are cross-validated against genome mining via BLAST) — github.com/mohimanilab/molDiscovery

## Examples

```
blastp -query molecule_bgc.faa -subject mibig_reference.faa -out bgc_hits.tsv -outfmt '6 qcovhsp pident' | awk '$1>=90 && $2>=70' > bgc_molecule_links.tsv
```

## Evaluation signals

- Every accepted link satisfies both thresholds: BLAST alignment coverage >= 90% and percent identity >= 70%
- On the 46 GNPS microbial datasets (922 strains), the linkage table contains exactly 19 known molecule-BGC links and 3 putative novel BGC families
- Recovered known links include xantholysin A/B/C, orfamide A/B/C, hygromycin A, and yersiniabactin as reported in the source study
- Output file bgc_molecule_links.tsv exists and each row carries coverage and identity columns passing the criteria

## Limitations

- Strict BLAST thresholds (>=90% coverage, >=70% identity) will miss genuinely novel BGC families; the study reports only 3 putative novel families (dinghupeptins, lipopeptin/neopeptin, longicatenamycin) beyond the 19 known links
- Requires paired mass spectrometry and genomic data for the same strains; the approach was validated only on microbial isolates whose genomes contain their known BGCs

## Evidence

- [intro] molDiscovery correctly links 19 known and three putative biosynthetic gene clusters to their molecular products: "molDiscovery correctly links 19 known and three putative biosynthetic gene clusters to their molecular products"
- [supplementary] Workflow criteria from source card: [evidence span withheld — anchor status: not_found]
- [results] Validation from source card: [evidence span withheld — anchor status: not_found]
- [readme] README access point: "To test molDiscovery for academic users, please visit : 

https://metabologenomic.cbd.cs.cmu.edu/"
