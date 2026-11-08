---
name: bgc-molecule-linkage-analysis
description: Use when identified molecules from GNPS microbial mass spectrometry datasets
  have paired strain genome data and you need to link molecules to biosynthetic gene
  clusters via MIBiG/BLAST validation.
when_to_use_negative:
- Mass spectrometry data have no paired genomic data for the source strains (no BGC
  annotations to validate against)
- The goal is spectral library search or de novo structure prediction rather than
  metabologenomic BGC validation
- Strain genome BGCs have no MIBiG reference clusters available for BLAST comparison
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: molDiscovery
  role: Produces the mass spectrometry small molecule identifications on GNPS microbial
    datasets that are cross-validated against genome mining for BGC linkage
  repo: github.com/mohimanilab/molDiscovery
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:da6497ef795ac1a1f9e71cb2fc1e8b41224da7cf0f38e25d534282ee853e1d3d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - molDiscovery
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bgc-molecule-linkage-analysis

## Summary

Metabologenomic workflow that cross-validates mass spectrometry small molecule identifications (from molDiscovery) against genome mining by BLAST-comparing each identified molecule's linked gene cluster to MIBiG reference BGCs, recovering known and putative novel BGC–molecule pairs (e.g., 19 known and 3 putative links on 46 GNPS microbial datasets covering 922 strains).

## When to use

Use when identified molecules from GNPS microbial mass spectrometry datasets have paired strain genome data and you need to link molecules to biosynthetic gene clusters via MIBiG/BLAST validation.

## When NOT to use

- Mass spectrometry data have no paired genomic data for the source strains (no BGC annotations to validate against)
- The goal is spectral library search or de novo structure prediction rather than metabologenomic BGC validation
- Strain genome BGCs have no MIBiG reference clusters available for BLAST comparison

## Inputs

- molDiscovery identification results for all 46 GNPS microbial datasets (922 microbial strains)
- MIBiG-linked BGC annotations from the corresponding strain genomes
- MIBiG reference biosynthetic gene cluster sequences

## Outputs

- bgc_molecule_links.tsv — table of accepted molecule-to-BGC links with BLAST coverage and percent identity values
- Set of known BGC–molecule links (19) and putative novel BGC families (3)

## How to apply

Load molDiscovery identification results for the microbial datasets (e.g., the 46 GNPS microbial datasets covering 922 strains) together with MIBiG-linked BGC annotations from the corresponding strain genomes. For each identified molecule, run BLAST comparisons between its linked gene cluster and the MIBiG reference clusters to obtain alignment coverage and percent identity values. Apply the dual acceptance threshold — BLAST alignment coverage >= 90% AND identity >= 70% — before assigning a molecule-to-BGC link. Compile accepted links into a linkage table (bgc_molecule_links.tsv) and verify it reproduces the reported benchmark: 19 known molecule–BGC links (xantholysin A/B/C, orfamide A/B/C, hygromycin A, yersiniabactin, etc.) plus 3 putative novel BGC families (dinghupeptins, lipopeptin/neopeptin, longicatenamycin).

## Related tools

- **molDiscovery** (Produces the mass spectrometry small molecule identifications on GNPS microbial datasets that are cross-validated against genome mining for BGC linkage) — github.com/mohimanilab/molDiscovery

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('blast_results.tsv',sep='\t'); df[(df.coverage>=0.9)&(df.identity>=0.7)].to_csv('bgc_molecule_links.tsv',sep='\t',index=False)"
```

## Evaluation signals

- Linkage table contains exactly 19 known molecule–BGC links (including xantholysin A/B/C, orfamide A/B/C, hygromycin A, yersiniabactin) as reported
- Table contains exactly 3 putative novel BGC-to-molecule families (dinghupeptin, lipopeptin/neopeptin, longicatenamycin)
- Every accepted link satisfies both BLAST criteria: coverage >= 90% and identity >= 70%
- Output file bgc_molecule_links.tsv is well-formed TSV with coverage and percent identity columns populated for each link

## Limitations

- Strict dual threshold (coverage >= 90%, identity >= 70%) will exclude genuine but divergent or partially covered BGC matches
- molDiscovery identifications underpinning the links have reduced accuracy for molecules below ~600 Da because fewer fragments are generated for small molecules
- molDiscovery ignores local/global structural features (moiety), covers only a limited number of bond types and rearrangement rules, and naively treats benzene rings and large resonant structures as alternating single/double bonds
- Fragmentation is influenced by experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra

## Evidence

- [intro] On a subset of the GNPS repository with known genomes, molDiscovery correctly links 19 known and three putative biosynthetic gene clusters to their molecular products: "On a subset of the GNPS repository with known genomes, molDiscovery correctly links 19 known and three putative biosynthetic gene clusters to their molecular products"
- [other] BLAST acceptance criteria: [evidence span withheld — anchor status: not_found]
- [readme] README contact/testing pointer: "To test molDiscovery for academic users, please visit : 

https://metabologenomic.cbd.cs.cmu.edu/"
- [intro] Benchmark identifications: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
