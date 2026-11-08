---
name: small-molecule-database-search-configuration
description: Use when tandem mass spectra (e.g., from GNPS) must be searched against
  a structure database such as the Dictionary of Natural Products or PubChem, and
  search configuration decisions — database redundancy, fragmentation model scope,
  ionization mode — must be made and validated.
when_to_use_negative:
- The compound is expected to be present in a reference spectral library — spectral
  library search is the most reliable annotation method and should be preferred over
  in silico database search
- The query spectra are negative-mode and no fragmentation model retrained on negative
  spectra is available — the default model reaches only ~36% accuracy
- The task is de novo structure prediction rather than searching against known databases;
  de novo prediction is rarely used in practice and this skill only annotates against
  existing structures
tools:
- name: molDiscovery
  role: 'Primary database search engine: generates mass spectrometry fragmentations
    and applies a learned probabilistic model to score molecule-spectrum pairs'
  repo: github.com/mohimanilab/molDiscovery
- name: MetFrag
  role: Rule-based baseline alternative that scores candidates by weighted peak count
    and bond dissociation energy
- name: CFM-ID
  role: Machine-learning baseline that predicts fragmentation spectra via a stochastic
    Markov process
- name: MAGMa+
  role: Baseline that scores both matched and missed peaks for molecule-spectrum pairs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23986-0
    title: 'MolDiscovery: learning mass spectrometry fragmentation of small molecules'
  source_content_hash: sha256:d8bb0f8f1230a80c6359ad1e8bb07b47de7dacd65c65e93ddf02e0c12d90b6fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CFM-ID
  - MAGMa+
  - MetFrag
  - molDiscovery
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# small-molecule-database-search-configuration

## Summary

Configure small molecule identification as an in silico database search of tandem mass spectra against a molecular structure database using molDiscovery's learned probabilistic fragmentation model, with explicit choices of database (e.g., non-redundant DNP vs. PubChem), search scale, and awareness of the model's experimental-domain limits.

## When to use

Use when tandem mass spectra (e.g., from GNPS) must be searched against a structure database such as the Dictionary of Natural Products or PubChem, and search configuration decisions — database redundancy, fragmentation model scope, ionization mode — must be made and validated.

## When NOT to use

- The compound is expected to be present in a reference spectral library — spectral library search is the most reliable annotation method and should be preferred over in silico database search
- The query spectra are negative-mode and no fragmentation model retrained on negative spectra is available — the default model reaches only ~36% accuracy
- The task is de novo structure prediction rather than searching against known databases; de novo prediction is rarely used in practice and this skill only annotates against existing structures

## Inputs

- Tandem mass spectra to annotate (e.g., GNPS tandem mass spectra; GNPS spectral library reference spectra in positive or negative ion mode)
- Small molecule structure database, e.g., the non-reduant (unique) Dictionary of Natural Products (DNP) or PubChem
- Known structure annotations for validation spectra (e.g., GNPS spectral library compound identities)

## Outputs

- Ranked list of candidate molecular structures per query spectrum (top-scoring DNP/PubChem candidate)
- Top-1 identification accuracy (e.g., fraction of query spectra whose top candidate matches the GNPS library annotation; ~36% for default model on negative mode vs. unique DNP)
- Set of unique identified small molecules at a stated FDR level (e.g., 3,185 unique molecules at 0% FDR)

## How to apply

Prepare query tandem MS spectra (e.g., the 3,964 negative-mode reference spectra from the GNPS spectral library) and choose the target structure database, preferring the non-redundant (unique) Dictionary of Natural Products to avoid inflated candidate lists. Run molDiscovery's database search with its pretrained probabilistic fragmentation model, noting that the default model was trained on combined CID/HCD positive-mode spectra and does not cover negative-ion behavior without retraining. Record the top-ranked candidate structure per query spectrum and compute top-1 accuracy against known identities (e.g., GNPS library annotations; the default model on negative-mode spectra yields ~36% against the unique DNP). At repository scale, the expected outcome is a FDR-controlled identification set — molDiscovery identified 3,185 unique small molecules at 0% FDR from over 8 million GNPS spectra, six times more unique molecules than previous methods such as MetFrag, CFM-ID, MAGMa+, and MSFinder. Use spectral library search as the most reliable but coverage-limited reference, falling back to structure-database search when the molecule is absent from libraries.

## Related tools

- **molDiscovery** (Primary database search engine: generates mass spectrometry fragmentations and applies a learned probabilistic model to score molecule-spectrum pairs) — github.com/mohimanilab/molDiscovery
- **MetFrag** (Rule-based baseline alternative that scores candidates by weighted peak count and bond dissociation energy)
- **CFM-ID** (Machine-learning baseline that predicts fragmentation spectra via a stochastic Markov process)
- **MAGMa+** (Baseline that scores both matched and missed peaks for molecule-spectrum pairs)

## Evaluation signals

- Top-1 accuracy against GNPS spectral library annotations: ~36% is the expected value when 3,964 negative-mode spectra are searched against the unique DNP with the unmodified default (positive-mode CID/HCD-trained) model
- At repository scale (8 million GNPS spectra), the search should recover ~3,185 unique small molecules at 0% FDR — a ~6-fold increase over prior methods; far fewer unique IDs indicates misconfiguration
- Redundancy check: accuracy computed against the non-redundant (unique) DNP, not the full DNP, so candidate lists are not inflated by duplicate structures
- Cross-check against baselines: results should be compared to rule-based (MetFrag, MSFinder, MassFrontier) and learned (CFM-ID, MAGMa+) competitors rather than evaluated in isolation

## Limitations

- The default probabilistic model was trained on combined CID/HCD positive-mode spectra; without retraining, negative-mode search accuracy drops to ~36% on GNPS negative spectra vs. unique DNP
- molDiscovery ignores local/global structural features (e.g., moiety) and covers only a limited number of bond types and rearrangement rules; benzene rings and large resonant structures are naively treated as alternating single/double bonds
- No p-value computation for general molecules — the MCMC p-value approach exists only for peptidic molecules
- Accuracy is lower in the low mass range (<600 Da), partly because fewer fragments are generated for small molecules
- Fragmentation is influenced by instrument type, dissociation technique, ionization mode, and collision energy, which are not fully modeled

## Evidence

- [intro] Spectral library coverage limits: "current reference spectral libraries are limited to tens of thousands of molecules, and the majority of known small molecules are not represented in any reference spectral library"
- [abstract] Repository-scale expected result: "molDiscovery correctly identify six times more unique small molecules than previous methods."
- [discussion] Negative-mode model caveat: "Mass spectrometry fragmentation is influenced by experimental factors such as instrument type, dissociation technique, ionization mode, collision energy, etc."
- [discussion] Structural feature coverage limits: "Currently, molDiscovery ignores these features and only covers a limited number of bond types and rearrangement rules"
- [readme] Access route for academic users: "To test molDiscovery for academic users, please visit : https://metabologenomic.cbd.cs.cmu.edu/"
