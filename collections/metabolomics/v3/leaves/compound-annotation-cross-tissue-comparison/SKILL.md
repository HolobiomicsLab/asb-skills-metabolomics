---
name: compound-annotation-cross-tissue-comparison
description: Use when you have PyINETA-annotated compounds from INADEQUATE-13C-JRES
  spectra of multiple tissues (e.g., liver, spleen, pancreas, muscle, lung) from labeled-diet
  mouse studies and need to determine which metabolites are enriched, depleted, or
  co-enriched per organ.
when_to_use_negative:
- When only a single tissue or sample type is available and no cross-tissue enrichment
  comparison is intended
- When the spectra are unlabeled or the diet is not isotopically labeled, so there
  is no traceable metabolite flux from a defined precursor pool
- When the spectra are not INADEQUATE or 13C-JRES NMR data, but instead MS/MS or other
  non-NMR metabolomics formats
edam_operation: http://edamontology.org/operation_3767
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: PyINETA
  role: Python package that reads/references INADEQUATE spectra, performs peak picking,
    filters peaks into compound networks, matches them to a simulated INADEQUATE database,
    and integrates 13C-JRES information to annotate compounds; all parameters and
    outputs are tracked for reproducibility of cross-tissue comparison.
  repo: https://github.com/edisonomics/PyINETA
- name: NMRbox
  role: Platform where PyINETA is installed and publicly available, providing an accessible
    environment for running the annotation workflow on tissue spectra.
- name: 13C NMR
  role: Experimental modality providing the INADEQUATE and 13C-JRES spectra that encode
    structural and chemical-shift information used for compound annotation across
    tissues.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.4c03966
    title: 'PyINETA: Open-Source Platform for INADEQUATE-JRES Integration in NMR Metabolomics'
  source_content_hash: sha256:96d704c41d92f73eaaaf55314e53900224c63a5a4bb92188ddec2a0871859d8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edisonomics/PyINETA
  tools:
  - 13C NMR
  - NMRbox
  - PyINETA
  - Python
  edam_operation: http://edamontology.org/operation_3767
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# compound-annotation-cross-tissue-comparison

## Summary

Compares tissue-resolved 13C-JRES compound annotation results from PyINETA to classify metabolites derived from 13C-labeled amino acid diets into tissue-distribution categories (e.g., liver-enriched, liver-depleted, co-enriched) and track labeled metabolite flux across organs.

## When to use

Use when you have PyINETA-annotated compounds from INADEQUATE-13C-JRES spectra of multiple tissues (e.g., liver, spleen, pancreas, muscle, lung) from labeled-diet mouse studies and need to determine which metabolites are enriched, depleted, or co-enriched per organ.

## When NOT to use

- When only a single tissue or sample type is available and no cross-tissue enrichment comparison is intended
- When the spectra are unlabeled or the diet is not isotopically labeled, so there is no traceable metabolite flux from a defined precursor pool
- When the spectra are not INADEQUATE or 13C-JRES NMR data, but instead MS/MS or other non-NMR metabolomics formats

## Inputs

- INADEQUATE NMR spectra from multiple tissues (e.g., liver, spleen, pancreas, muscle, lung) of 13C-labeled amino-acid fed mice
- Complementary 13C-JRES spectra for the same tissue samples
- PyINETA config file specifying pipeline parameters
- Simulated INADEQUATE metabolite database for compound matching

## Outputs

- Per-tissue compound annotation lists with network-to-database matches
- Cross-tissue compound enrichment matrix (compound × tissue intensity table)
- Tissue-distribution category assignment per compound (e.g., liver-enriched, liver-depleted, co-enriched)

## How to apply

Run the PyINETA pipeline (load, pick, cluster, find, match) on INADEQUATE spectra from each tissue to produce compound annotations, then integrate these with complementary 13C-JRES data to quantify per-compound intensity across tissues. Normalize and compare per-tissue relative abundance of each annotated compound, grouping metabolites into categories such as liver-enriched, liver-depleted (enriched in other tissues), or co-enriched. The rationale is that PyINETA keeps all pipeline parameters and outputs, ensuring annotation transparency, so each compound's tissue assignment can be traced back to specific network matches. Cross-tissue comparison relies on the fact that PyINETA integrates INADEQUATE (structural) and 13C-JRES (chemical-shift/quantitative) information to identify metabolites that were transferred and transformed from the 13C-labeled amino acid diet.

## Related tools

- **PyINETA** (Python package that reads/references INADEQUATE spectra, performs peak picking, filters peaks into compound networks, matches them to a simulated INADEQUATE database, and integrates 13C-JRES information to annotate compounds; all parameters and outputs are tracked for reproducibility of cross-tissue comparison.) — https://github.com/edisonomics/PyINETA
- **NMRbox** (Platform where PyINETA is installed and publicly available, providing an accessible environment for running the annotation workflow on tissue spectra.)
- **13C NMR** (Experimental modality providing the INADEQUATE and 13C-JRES spectra that encode structural and chemical-shift information used for compound annotation across tissues.)

## Examples

```
python run_pyineta.py -c config.ini -o tissue_output
```

## Evaluation signals

- Check that PyINETA output logs retain all pipeline parameters and intermediate files for each tissue so that annotation decisions can be traced back to specific networks
- Confirm that every annotated compound is supported by a matched network in the simulated INADEQUATE database, not just peak picking alone
- Verify that the complementary 13C-JRES integration step is performed for each annotated compound, since INADEQUATE alone is insufficient for cross-tissue enrichment classification
- Compare per-tissue relative abundance profiles of annotated compounds to expected tissue-specific enrichment patterns (e.g., metabolites known to be liver-enriched should show higher abundance in liver than in spleen or pancreas)
- Sanity-check that compounds transformed from the labeled amino acid precursor are detected across multiple tissues, consistent with the finding that 13C-labeled amino acids fed to mice were transferred to different tissues and transformed to other metabolites

## Limitations

- PyINETA was evaluated only on a model mouse study; its performance on other organisms, tissues, or labeling schemes has not been validated in this work
- Peak-network filtering to identify same-compound peaks is probabilistic, so ambiguous or overlapping metabolite networks may yield incomplete or mixed annotations
- Cross-tissue enrichment classification requires comparable spectral quality and acquisition conditions across tissues; variability in SNR or acquisition parameters may bias intensity-based enrichment calls

## Evidence

- [intro] PyINETA automatically performs compound annotation by matching peak networks to a simulated INADEQUATE metabolite database, integrating complementary 13C-JRES information.: "It is designed to filter these picked peaks to identify networks of peaks (ideally) coming from the same compound which it then matches to a simulated INADEQUATE database of metabolites to identify"
- [abstract] PyINETA integrates INADEQUATE and 13C-JRES so tissue-specific compound distributions can be evaluated.: "integrates information of INADEQUATE and a complementary NMR experiment 13 C J -resolved experiment ( 13 C-JRES)"
- [abstract] PyINETA tracks all pipeline parameters and outputs, enabling reproducible annotation.: "PyINETA keeps track of all the pipeline parameters and outputs, ensuring the transparency of annotation in metabolomics."
- [readme] PyINETA can be installed via the provided environment.yml and run through the run_pyineta.py CLI with a config file and step selection.: [evidence span withheld — anchor status: not_found]
- [abstract] 13C-labeled amino acids were transferred to different tissues and transformed to other metabolites.: "The results showed that 13 C-labeled amino acids that were fed to mice were transferred to different tissues, and, also, they were transformed to other metabolites."
