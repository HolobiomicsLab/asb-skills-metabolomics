---
name: natural-product-annotation
description: Use when annotating GNPS tandem MS spectra (e.g., the ~8 million spectra
  of the GNPS microbial repository) with known natural-product structures, or when
  pairing MS identifications with genome-mining data to confirm molecule-to-BGC links.
when_to_use_negative:
- The target spectra are represented in a spectral library — spectral library search
  is described as the most reliable annotation method, so in silico searching is unnecessary
- The query molecules are very small (<400 Da) or very heavy (>1000 Da), where existing
  machine-learning fragmentation methods (including limitations acknowledged for molDiscovery)
  perform poorly
- No paired genomic data are available and the goal is BGC validation rather than
  simple spectrum annotation
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: molDiscovery
  role: 'Mass spectral database search engine: generates fragmentations and matches
    molecule–spectrum pairs with a learned probabilistic model; also used to link
    identified molecules to BGCs in the genome-known GNPS subset'
  repo: github.com/mohimanilab/molDiscovery
- name: BLAST
  role: Compares each identified molecule's linked gene cluster against MIBiG reference
    clusters to obtain alignment coverage and percent identity for metabologenomic
    validation
- name: MAGMa+
  role: Prior in silico identification method mentioned as a baseline that molDiscovery
    outperforms (scores matched and missed peaks)
- name: MetFrag
  role: Prior rule-based in silico search method mentioned as a comparison baseline
    (weighted peak count and bond dissociation energy scoring)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:bc3b0508e3d3af960374e6cce0bc1c637fbd0f5da8a7c9380830fda806a953ff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - MAGMa+
  - MetFrag
  - molDiscovery
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# natural-product-annotation

## Summary

Identifies small molecules from tandem mass spectra by searching them against molecular structure databases (e.g., PubChem, Dictionary of Natural Products) with molDiscovery's learned fragmentation-probability model, and optionally links identified molecules to biosynthetic gene clusters (BGCs) via metabologenomic validation against MIBiG-annotated genomes.

## When to use

Use when annotating GNPS tandem MS spectra (e.g., the ~8 million spectra of the GNPS microbial repository) with known natural-product structures, or when pairing MS identifications with genome-mining data to confirm molecule-to-BGC links.

## When NOT to use

- The target spectra are represented in a spectral library — spectral library search is described as the most reliable annotation method, so in silico searching is unnecessary
- The query molecules are very small (<400 Da) or very heavy (>1000 Da), where existing machine-learning fragmentation methods (including limitations acknowledged for molDiscovery) perform poorly
- No paired genomic data are available and the goal is BGC validation rather than simple spectrum annotation

## Inputs

- Tandem mass spectra (MS/MS) from GNPS datasets, e.g., 46 microbial datasets covering 922 strains
- Molecular structure databases for searching: PubChem, Dictionary of Natural Products (DNP)
- GNPS spectral library reference spectra (training data for the probabilistic model)
- MIBiG-linked biosynthetic gene cluster (BGC) annotations from strain genomes

## Outputs

- Annotated spectrum–molecule assignments with controlled FDR (e.g., unique molecule identifications at 0% FDR)
- Molecule-to-BGC linkage table (bgc_molecule_links.tsv) with BLAST coverage and percent identity per link

## How to apply

Load the tandem MS/MS spectra collected from the sample (e.g., GNPS microbial datasets covering 922 strains with paired genomic data) and search them against candidate structures from databases such as PubChem or the Dictionary of Natural Products using molDiscovery, which generates mass-spectral fragmentations and scores molecule–spectrum pairs with a probabilistic model trained on the GNPS spectral library. Filter identifications to a controlled false discovery rate — in the article, 3185 unique small molecules were identified at 0% FDR. For metabologenomic validation, load MIBiG-linked BGC annotations for the strain genomes, run BLAST between each identified molecule's linked gene cluster and the MIBiG reference clusters, and accept a molecule-to-BGC link only when alignment coverage >= 90% and percent identity >= 70%. Compile accepted links into a table (e.g., bgc_molecule_links.tsv); the reported result is 19 known molecule–BGC links (e.g., xantholysin A/B/C, orfamide A/B/C, hygromycin A, yersiniabactin) plus 3 putative novel BGC families (dinghupeptins, lipopeptin/neopeptin, longicatenamycin).

## Related tools

- **molDiscovery** (Mass spectral database search engine: generates fragmentations and matches molecule–spectrum pairs with a learned probabilistic model; also used to link identified molecules to BGCs in the genome-known GNPS subset) — github.com/mohimanilab/molDiscovery
- **BLAST** (Compares each identified molecule's linked gene cluster against MIBiG reference clusters to obtain alignment coverage and percent identity for metabologenomic validation)
- **MAGMa+** (Prior in silico identification method mentioned as a baseline that molDiscovery outperforms (scores matched and missed peaks))
- **MetFrag** (Prior rule-based in silico search method mentioned as a comparison baseline (weighted peak count and bond dissociation energy scoring))

## Evaluation signals

- The linkage table contains exactly 19 known and 3 putative molecule-to-BGC links, as reported for the 46 GNPS microbial datasets
- Every accepted molecule-to-BGC link satisfies BLAST coverage >= 90% and percent identity >= 70%
- Known pairs are recovered, including xantholysin A/B/C, orfamide A/B/C, hygromycin A, and yersiniabactin with their cognate BGCs
- On the full 8-million-spectrum GNPS search, identifications at 0% FDR reach ~3185 unique small molecules (about six times more than prior methods)
- A molecule assigned only where both its MS/MS spectrum and the cognate genome (922-strain subset) agree — spectra-only or genome-only hits should be flagged

## Limitations

- molDiscovery performs less accurately in the low mass range (<600 Da) because fewer fragments are generated for small molecules
- It ignores local/global structural features (e.g., moieties) and covers only a limited number of bond types and rearrangement rules; benzene rings and large resonant structures are naively treated as alternating single/double bonds
- No p-value computation for general molecules — the MCMC p-value approach exists only for peptidic molecules and is hard to extend
- Fragmentation accuracy depends on experimental factors (instrument type, dissociation technique, ionization mode, collision energy) not fully modeled; the default model was trained on combined CID/HCD positive-mode spectra
- Graph-based machine learning of structural features remains future work, not implemented

## Evidence

- [abstract] Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification: "Here, we present molDiscovery, a mass spectral database search method that improves both efficiency and accuracy of small molecule identification"
- [intro] On a subset of the GNPS repository with known genomes, molDiscovery correctly links 19 known and three putative biosynthetic gene clusters to their molecular products: "molDiscovery correctly links 19 known and three putative biosynthetic gene clusters to their molecular products"
- [intro] molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [readme] To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/: "To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/"
