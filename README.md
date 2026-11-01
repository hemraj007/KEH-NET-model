Code and data for "KEH-NET: An Explainable Knowledge-Enhanced Hybrid Neural
Network for Multi-label Text Classification of Biomedical Literature"
(Computational Intelligence, under review).

## Datasets

Source corpora are publicly available and are not redistributed here:
- HoC: https://github.com/sb895/Hallmarks-of-Cancer (Baker et al., 2016)
- CEI: https://github.com/sb895/chemical-exposure-information-corpus
  (Larsson et al., 2017)

`data/*/\*_umls_sortlist_tui.csv` are our pre-processed versions, with
entity linking applied. Row order in these files defines the
cross-validation folds (see below).

## Knowledge graphs

For each dataset:

| File | Contents |
|---|---|
| `triples_ontology.json` | ScispaCy entity linking, GO concepts, UMLS relations |
| `triples_llm_raw.json` | Gemini output before filtering |
| `triples_llm_filtered.json` | After structural and textual-presence filtering |
| `triples_llm_rejected.json` | Removed triples with per-triple reason |
| `triples_combined.json` | Union of the two sources — the graph used by KEH-NET |
| `filter_report.json` | Retention rate and rejection counts by category |

Triples are grouped per document, in the same order as the rows of the
pre-processed CSV. Each triple is `{"source": ..., "relationship": ...,
"target": ...}`.

## Extraction settings

- Model: `gemini-1.5-flash-<VERSION>`  ← replace with the pinned version
- Decoding: default parameters; safety thresholds set to BLOCK_ONLY_HIGH
- Prompt: `prompts/gemini_extraction_prompt.txt`, used verbatim
- Entity linking: ScispaCy `<VERSION>`, model `<en_core_sci_sm>`,
  linker `<kb>`, similarity threshold `<t>`, abbreviation resolution enabled
- UMLS access: UMLS REST API, requires a UTS account and licence agreement

## Filtering

`code/02_triple_filtering.py` applies structural validation (empty fields,
self-loops, entity spans over six tokens, pronoun and stopword entities,
within-document duplicates) and requires at least one entity of each triple
to occur in the source abstract. Retention: 95.8% (HoC), 94.1% (CEI).

## Cross-validation splits

All experiments use `KFold(n_splits=10, shuffle=True, random_state=42)`
applied to the pre-processed CSV in its published row order. The folds are
therefore fully determined by the released data and this seed; no separate
index files are required.

## Reproducing

```bash
pip install -r requirements.txt
python code/02_triple_filtering.py --triples data/hoc/triples_llm_raw.json \
    --csv data/hoc/hoc_data_umls_sortlist_tui.csv --require either \
    --min-per-doc 3 --outdir data/hoc/
jupyter notebook code/04_data_prep_hoc.ipynb
```

## Citation

<BibTeX entry, added on acceptance>

## License

Code: MIT. Data derived from the HoC and CEI corpora remains subject to the
licences of those corpora.
