---
name: microbial-genome-mining
description: Use when paired metabolomics (tandem MS/GNPS) and genomic data exist
  for microbial isolates, and the goal is to link identified molecules to known or
  putative novel biosynthetic gene clusters.
when_to_use_negative:
- No paired genomic data are available for the strains in the mass spectrometry datasets,
  so BGC cross-validation is impossible
- The goal is general small molecule identification from spectra alone without metabologenomic
  validation — use molDiscovery spectral search directly instead
- Molecules are identified outside microbial isolate context (e.g., clinical or environmental
  samples without genome mining), where the BGC-linking criteria do not apply
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: molDiscovery
  role: Identifies small molecules from tandem MS spectra via probabilistic spectral
    database search, providing the molecular identifications that are cross-validated
    against genome mining
  repo: github.com/mohimanilab/molDiscovery
- name: BLAST
  role: Compares each identified molecule's linked gene cluster against MIBiG reference
    clusters to compute alignment coverage and percent identity for link acceptance
- name: MIBiG
  role: Reference database of known biosynthetic gene clusters used as the ground
    truth for molecule-BGC linking
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:6e61092bcaede3b6e7c9bf14bbc12a52a3b365bc5aeb8f31bbe46f9915110153
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - MIBiG
  - molDiscovery
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# microbial-genome-mining

## Summary

Cross-validates mass-spectrometry small molecule identifications (from molDiscovery) against genome mining of paired microbial strains, linking molecular products to biosynthetic gene clusters (BGCs) via BLAST comparisons against MIBiG references.

## When to use

Use when paired metabolomics (tandem MS/GNPS) and genomic data exist for microbial isolates, and the goal is to link identified molecules to known or putative novel biosynthetic gene clusters.

## When NOT to use

- No paired genomic data are available for the strains in the mass spectrometry datasets, so BGC cross-validation is impossible
- The goal is general small molecule identification from spectra alone without metabologenomic validation — use molDiscovery spectral search directly instead
- Molecules are identified outside microbial isolate context (e.g., clinical or environmental samples without genome mining), where the BGC-linking criteria do not apply

## Inputs

- molDiscovery small molecule identification results for microbial tandem MS datasets (GNPS spectral matches)
- MIBiG-linked biosynthetic gene cluster annotations for the corresponding strain genomes
- Paired genome assembly/annotation files for the microbial isolates

## Outputs

- bgc_molecule_links.tsv — table of validated molecule-to-BGC links (19 known + 3 putative in the reference analysis)
- BLAST alignment coverage and percent identity values per candidate link

## How to apply

Load molDiscovery identification results for the microbial datasets (e.g., the 46 GNPS microbial datasets covering 922 strains) and the MIBiG-linked BGC annotations for the corresponding strain genomes. Run BLAST comparisons between each identified molecule's linked gene cluster and the MIBiG reference clusters to obtain alignment coverage and percent identity values. Assign a molecule-to-BGC link only when BLAST alignment meets both criteria: coverage >= 90% and identity >= 70%. Compile accepted links into a table, confirming known links (e.g., xantholysin A/B/C, orfamide A/B/C, hygromycin A, yersiniabactin) and flagging putative novel BGC families (dinghupeptins, lipopeptin/neopeptin, longicatenamycin). Rationale: genome-matching provides orthogonal validation that molDiscovery's spectral identifications recover true biosynthetic products rather than false positives.

## Related tools

- **molDiscovery** (Identifies small molecules from tandem MS spectra via probabilistic spectral database search, providing the molecular identifications that are cross-validated against genome mining) — github.com/mohimanilab/molDiscovery
- **BLAST** (Compares each identified molecule's linked gene cluster against MIBiG reference clusters to compute alignment coverage and percent identity for link acceptance)
- **MIBiG** (Reference database of known biosynthetic gene clusters used as the ground truth for molecule-BGC linking)

## Examples

```
blastn -query strain_bgc.fna -subject mibig_ref.fna -outfmt "6 qcovs pident" | awk '$1>=90 && $2>=70' > bgc_molecule_links.tsv
```

## Evaluation signals

- The linkage table contains exactly 19 known molecule-BGC links (including xantholysin A/B/C, orfamide A/B/C, hygromycin A, yersiniabactin) as reported
- Exactly 3 putative novel BGC-to-molecule family links are recovered (dinghupeptin, lipopeptin/neopeptin, longicatenamycin)
- Every accepted link satisfies both criteria: BLAST alignment coverage >= 90% and percent identity >= 70%
- All molecule identifications used downstream have 0% FDR from molDiscovery spectral search
- bgc_molecule_links.tsv is produced and each row carries its coverage/identity evidence values

## Limitations

- Linking validation depends on availability of paired genomic data; strains without genome mining cannot be cross-validated
- molDiscovery identifications are weakest for super small molecules (<400 Da) and computationally insufficient for heavy molecules (>1000 Da), limiting which molecules can be linked
- No p-value computation for general molecules (MCMC p-value approach exists only for peptidic molecules), so confidence assessment of links relies on the coverage/identity thresholds alone

## Evidence

- [intro] 19 known and 3 putative BGC links to molecules: "molDiscovery correctly links 19 known and three putative biosynthetic gene clusters to their molecular products"
- [readme] web server entry point for testing: "To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/"
- [intro] identifications at 0% FDR on GNPS: "molDiscovery identified 3185 unique small molecules at 0% false discovery rate (FDR), a six times increase compared to existing methods"
- [abstract] probabilistic matching rationale: "learning a probabilistic model to match small molecules with their mass spectra"
