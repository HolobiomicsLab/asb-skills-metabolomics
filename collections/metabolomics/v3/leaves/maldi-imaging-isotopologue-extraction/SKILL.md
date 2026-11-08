---
name: maldi-imaging-isotopologue-extraction
description: Use when you have MALDI-AIF imzML data (e.g. 600-1600 m/z, negative mode)
  from 13C-labelled tissue and need spatial isotopologue ion images for fatty acids
  rather than METASPACE's natural-abundance-only annotations.
when_to_use_negative:
- Your annotations already come from METASPACE and you only need natural-abundance
  isotopologue distributions - METASPACE output suffices and the pyimzML extraction
  step is unnecessary.
- 'Overall substrate usage is very low (e.g. hypoxia/normoxia single-cell data): fitting
  a binomial distribution to estimate uptake fraction becomes unreliable and was only
  used for ACLYkd and brain data.'
- You need absolute quantification of substrate contribution to the lipogenic acetyl-CoA
  pool - the method provides only relative contributions since pool size cannot be
  quantified.
edam_operation: http://edamontology.org/operation_3694
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: pyimzML
  role: Parses the imzML file to extract ion images for all isotopologues of ions
    from the molecule list, in the format expected by SpaceM; used because METASPACE
    only generates natural-abundance isotopologue distributions
  repo: https://github.com/alexandrovteam/pyimzML
- name: 13C-SpaceM
  role: Provides the analysis scripts (extract_isotopologue_ion_images.py, analysis/brain/00_extract_ion_images.py)
    and per-pixel isotopologue extraction/correction workflow for mouse-brain tissue
    imaging data
  repo: https://github.com/Buglakova/13C-SpaceM
- name: 13C_binomial
  role: Fits the binomial model (C16/C18) to corrected, normalized isotopologue distributions
    of fatty acids to estimate fractional labelling and uptake
  repo: https://github.com/Buglakova/13C_binomial
- name: METASPACE
  role: Hosts and serves the mouse-brain MALDI-imaging datasets for download (metaspace2020.eu/project/buglakova-2024);
    annotations from METASPACE are superseded for isotopologue extraction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42255-024-01118-4
    title: Spatial single-cell isotope tracing reveals heterogeneity of de novo fatty
      acid synthesis in cancer
  source_content_hash: sha256:3dc56d9771f23ca2d3bcf8c2d3a1c4bd0812dad376e48a4c6bb674473ba23e7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexandrovteam/pyimzML
  tools:
  - 13C-SpaceM
  - 13C_binomial
  - METASPACE
  - pyimzML
  edam_operation: http://edamontology.org/operation_3694
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# maldi-imaging-isotopologue-extraction

## Summary

Parse MALDI-imaging imzML files with pyimzML and extract per-pixel ion images for all isotopologues (M+0 to M+16+) of labelled fatty acids, so that natural-abundance correction and binomial fitting can quantify de novo synthesis versus uptake spatially, as done in 13C-SpaceM for U-13C-glucose-labelled glioma tissue.

## When to use

Use when you have MALDI-AIF imzML data (e.g. 600-1600 m/z, negative mode) from 13C-labelled tissue and need spatial isotopologue ion images for fatty acids rather than METASPACE's natural-abundance-only annotations.

## When NOT to use

- Your annotations already come from METASPACE and you only need natural-abundance isotopologue distributions - METASPACE output suffices and the pyimzML extraction step is unnecessary.
- Overall substrate usage is very low (e.g. hypoxia/normoxia single-cell data): fitting a binomial distribution to estimate uptake fraction becomes unreliable and was only used for ACLYkd and brain data.
- You need absolute quantification of substrate contribution to the lipogenic acetyl-CoA pool - the method provides only relative contributions since pool size cannot be quantified.

## Inputs

- imzML file (MALDI-AIF, e.g. 600-1600 m/z negative mode, 10 um pixel size) downloaded from the METASPACE project buglakova-2024
- Ion/molecule list CSV (e.g. ion_list_fa.csv) enumerating fatty acids and their isotopologues M+0 to M+16+
- Environment: pyimzML, numpy-based binomial fitting code from Buglakova/13C_binomial

## Outputs

- Per-pixel ion images for all isotopologues of each fatty acid in the ion list
- Corrected, normalized isotopologue distributions per pixel
- Binomial fit results (model type, fit_success, uptake, p, mean) and de novo synthesis fraction tables (e.g. brain_denovo_fractions.csv)
- Spatial maps of de novo synthesis fractions for figures such as Figure 5 panels c-d and Figure 6 panel a

## How to apply

Run the project's ion-image extraction script (analysis/brain/00_extract_ion_images.py with an ion list CSV such as ion_list_fa.csv) to parse the imzML file via pyimzML and produce ion images for every isotopologue (M+0 through M+16+) of each fatty acid in the list; this is needed because METASPACE only generates isotopologue distributions based on natural isotope abundance. In the mouse-brain workflow this was applied to the AIF dataset 2023-09-25_ME_PattiLab_MouseBrain_4-4_010G_AIF_600-1600_30NCE_100-600_10umss_285x230_32at (10 um pixel size, 285x230 pixels). After extraction, apply natural isotope abundance correction to each fatty acid's isotopologue distribution per pixel, then fit a binomial distribution (model type C16 or C18; the C18 model accounts for the M+2 peak arising from unlabeled palmitate + 2 x 13C) to estimate fractional labelling of the lipogenic acetyl-CoA pool and compute de novo fractions such as 1-(M+0)/sum(M+n). Validation relies on qualitative reproduction of expected spatial patterns rather than absolute values, since isotopic steady state was likely not reached within the 48-h labelling period.

## Related tools

- **pyimzML** (Parses the imzML file to extract ion images for all isotopologues of ions from the molecule list, in the format expected by SpaceM; used because METASPACE only generates natural-abundance isotopologue distributions) — https://github.com/alexandrovteam/pyimzML
- **13C-SpaceM** (Provides the analysis scripts (extract_isotopologue_ion_images.py, analysis/brain/00_extract_ion_images.py) and per-pixel isotopologue extraction/correction workflow for mouse-brain tissue imaging data) — https://github.com/Buglakova/13C-SpaceM
- **13C_binomial** (Fits the binomial model (C16/C18) to corrected, normalized isotopologue distributions of fatty acids to estimate fractional labelling and uptake) — https://github.com/Buglakova/13C_binomial
- **METASPACE** (Hosts and serves the mouse-brain MALDI-imaging datasets for download (metaspace2020.eu/project/buglakova-2024); annotations from METASPACE are superseded for isotopologue extraction)

## Examples

```
python analysis/brain/00_extract_ion_images.py analysis/brain/ion_list_fa.csv
```

## Evaluation signals

- Reproduced panels show strong induction of de novo synthesis of SFAs (palmitate, stearate) in tumour tissue versus surrounding brain
- MUFAs palmitoleate and oleate show a higher M+0 (uptake) proportion than palmitate, consistent with high MUFA demand
- M+2 enrichment in stearate and oleate is consistent with elongation of palmitate/palmitoleate chains (C18 model fits distinguish elongation-derived M+2)
- Binomial fits report fit_success = 1 (optimization converged) and the fitted p matches the distribution mean via mean = pn
- Natural isotope abundance correction removes natural 13C signal so corrected M+0 fractions in unlabelled control tissue approach expected natural abundance

## Limitations

- MALDI-AIF restricts analysis to fatty acids from lipid classes effectively ionized in negative mode, primarily acidic glycerophospholipids; abundant species like phosphatidylcholines, triglycerides, and sphingomyelins are underrepresented
- Isotopic steady state was likely not reached within the 48-h labelling period, so labelling data must be interpreted alongside pool size data; de novo vs uptake proportions may differ by lipid class
- The binomial model cannot reliably estimate uptake fraction when overall substrate usage is very low (e.g. hypoxia/normoxia experiment); the spatial-resolution MALDI imaging used here analyses per pixel, not single cells, in tissue sections

## Evidence

- [readme] METASPACE only generates isotopologue distributions based on natural isotope abundance, therefore here after generating all possible isotopologues for a given molecule, pyimzML is used to extract ion images: "Metaspace only generates isotopologue distributions based on natural isotope abundance, therefore here after generating all possible isotopologues for a given molecule,"
- [readme] In each pixel we do the same analysis steps: natural isotope abundance correction and binomial fitting: "In each pixel we do the same analysis steps as for the single-cell data:
- Perform natural isotope abundance correction for each ion's isotopologue distribution
- Fit binomial distribution to each"
- [readme] Function fit_binomial takes as input numpy array of normalized and corrected intensities; model can be either C16 or C18; C18 takes into account that M+2 peak can come from unlabeled palmitate + 2 13C: [evidence span withheld — anchor status: external_doc]
- [discussion] Isotopologue analysis revealed a strong induction of synthesis of SFAs, myristate, palmitate and stearate, in tumour tissue compared with the surrounding brain: "Isotopologue analysis revealed a strong induction of synthesis of SFAs, myristate, palmitate and stearate, in tumour tissue compared with the surrounding brain"
- [discussion] Isotopic steady state was likely not reached for fatty acids within the 48-h labelling period; labelling data need to be interpreted under consideration of pool size data: "isotopic steady state was likely not reached for fatty acids within the 48-h labelling period38. Thus, labelling data need to be interpreted under consideration of pool size data."
