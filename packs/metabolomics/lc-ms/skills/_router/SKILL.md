---
name: metabolomics-lc-ms-router
description: Use for any metabolomics — LC-MS how-to question (which method, tool, parameters or pipeline to use), before answering from memory. Searches 2,617 evidence-grounded metabolomics — LC-MS skills distilled from method papers and their code, then reads the one that fits.
license: CC-BY-4.0
metadata:
  skills_count: 2617
  leaf_dir: leaves
  retrieval: bin/search_skills.py
  indexes:
  - skills_index.json
schema_version: 0.2.0
---

# ASB Metabolomics — LC-MS router

**2,617 evidence-grounded skills**, each derived from a
method paper (journal article or preprint) and its code.
They ship beside this router as data, in `leaves/<slug>/SKILL.md`. **Do not
list `leaves/` and do not read the indexes whole.** Search, then read the one
file you need.

## 1. Search

Run the commands as written, from any working directory; do not `cd`.
If a command still shows a `${...}` variable instead of an absolute path, the
host did not fill it in: replace the variable with the absolute directory of
this file, `<collection root>/skills/_router`.

```bash
# one atomic skill
python3 "${CLAUDE_SKILL_DIR}/../../bin/search_skills.py" --query "<the user's task>" -k 10
```

Each hit prints the absolute `read:` path; a skill hit also its licence tier and source DOI. Narrow with `--tool <name>`, `--technique <tag>` or `--edam <iri-substring>`.
Python 3.9 or later, standard library only, no network; where `python3` is missing, use `python` or `py -3`.

## 2. Apply

Read the `read:` path of the chosen hit, `leaves/<slug>/SKILL.md`. Its
frontmatter carries `metadata.tools` (what to install or invoke), `derived_from`
(source paper DOIs) and `evidence_spans` (verbatim anchors from the paper or
repo). Follow the body.
In the answer, name the skill and give its source DOI and its licence tier.

## 3. Ground (recommended)

Before trusting a parameter, threshold or default, verify it against the paper
the skill was built from. `kb_bundle.json` maps each skill to its source KBs:

```bash
python3 "${CLAUDE_SKILL_DIR}/../../bin/perspicacite_kb_bind.py" query --collection "${CLAUDE_SKILL_DIR}/../.." \
  --skill <slug> --question "<what you need to verify>"
```

See `GROUNDING.md` for the backends and tiers.

## Licence tiers

Each skill hit prints `tier`, the licence tier of the tool it relies on (the
skill text itself is CC-BY-4.0). Check it before presenting a candidate.
`open`: commercial use allowed. `noncommercial`: academic or non-commercial use
only; get explicit confirmation of a non-commercial purpose before applying it.
`restricted`: a licence was established and it constrains reuse (for example a
vendor licence); say so before commercial use or redistribution. `unknown`: no
licence for the tool was established; say so, and treat the tool as all rights
reserved until the user has checked it.

## Report and contribute

`metabolomics-lc-ms` improves from use. Offer a report yourself, once per
problem, when a procedure failed or no longer matches the tool, the search
returned nothing usable and the work was done unaided, the user names a
method paper whose DOI the search does not find, the leaves had to be
composed by hand, or a command of this collection failed. Name the route:

1. this skill is wrong or could be better;
2. a skill is missing: request it, or propose yours;
3. a paper is not referenced: propose it;
4. something technical failed: report it.

After a no, do not raise it again. Draft the report, show the user its exact
title, labels and body, and post only on an explicit yes. Never paste paths,
credentials or sample names.

Write the title and body: the route, the skill slug or task, what happened.
With `gh` and the user's yes, run `gh issue create --repo HolobiomicsLab/asb-skill-collections`;
otherwise give the user the address
`https://github.com/HolobiomicsLab/asb-skill-collections/issues/new?title=<title>&body=<body>`, URL-encoded.

Reports go to the public registry; a maintainer carries them into the
domain repository.
