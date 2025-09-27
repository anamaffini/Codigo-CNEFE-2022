
---
# Title: 'Código CNEFE 2022: reproducible pipelines for classifying and aggregating Brazil’s CNEFE 2022 address points'
tags:
  - geospatial
  - GIS
  - Python
  - urban planning
  - Brazil
  - census

authors:
  - name: Ana Luisa Maffini
  - orcid: 0000-0001-5334-7073
  - affiliation: 1

affiliations:
  - name: Universidade Federal do Rio Grande do Sul (UFRGS), PROPUR — Programa de Pós-Graduação em Planejamento Urbano e Regional

date: 2025-09-27

bibliography: paper.bib
---

# Summary

Brazil’s **Cadastro Nacional de Endereços para Fins Estatísticos (CNEFE)** is the national address register curated by the **Instituto Brasileiro de Geografia e Estatística (IBGE)** to support census operations, surveys, and sampling. The **2022** release provides **spatial points (GeoJSON)** with an updated category schema (`COD_ESPECIE` = 1..8), in contrast to **2010**, which is distributed as **fixed-width text without geometry**. This change improves usability but complicates **continuity** for analysts whose pipelines, indicators, and maps were built around the 2010 “wide” format.

**Código CNEFE 2022** is a small, well-documented pipeline that bridges this gap. It (i) maps `COD_ESPECIE` codes **1–8** directly into one-hot (**“wide”**) columns (`cat_*`), (ii) derives **optional subtypes** (`sub_*`) exclusively for **“Outras Finalidades”** (code 6) via a transparent, editable keyword dictionary, (iii) preserves **zero-padding** in identifiers (UF, municipality, CEP) and handles **CRS** coherently (input **EPSG:4674**, GeoJSON export **EPSG:4326**), and (iv) exports **GeoPackage/GeoJSON** layers plus **CSV aggregations** by **block face (`NUM_FACE`)** and **census sector (`COD_SETOR`)**. The result is a **reproducible, auditable** baseline compatible with common research workflows (GeoPandas/QGIS) and directly comparable to legacy 2010 outputs.

**Repository:** [[REPO_URL](https://github.com/anamaffini/Codigo-CNEFE-2022)]  
**Archive (DOI):** [[ZENODO_DOI](https://doi.org/10.5281/zenodo.17184232)]

# Statement of need

Urban and regional planning studies routinely require **classifying** CNEFE records into policy-relevant categories (domicílio particular/coletivo, ensino, saúde, etc.) and **aggregating** them to operational units (faces/sectors) for mapping and modeling (e.g., access to services, urban morphology, inequality diagnostics). Publicly available examples tend to be **ad‑hoc** or version‑specific, and the shift from **2010 (non‑spatial)** to **2022 (spatial)**, with **changed codes and semantics**, introduces friction that **breaks backward compatibility**. Researchers lose time replicating basic steps — type coercion, zero‑padding, deterministic category flags, and consistent exports.

**Código CNEFE 2022** addresses these pain points with a **minimal, explicit** pipeline that standardizes outputs across vintages while respecting the official 2022 categories. It emphasizes: (a) **deterministic boolean logic** for the main `cat_*` flags (no brittle text parsing), (b) **scoped** text matching only for `sub_*` within `cat_OUTRAS_FINALIDADES`, (c) **robust I/O** (GeoPackage/GeoJSON + CSV aggregates), and (d) easy **inspectability** for audit and review.

# State of the field

Brazil’s CNEFE is widely used in urban studies, socio‑spatial diagnostics, and public policy analyses. The Python geospatial ecosystem (GeoPandas, Shapely, PyProj, Fiona/Pyogrio, Pandas) is mature and well supported, but **domain‑specific, open, and documented pipelines** targeting **CNEFE 2022** and ensuring **continuity** with **2010** remain scattered across private scripts or institutional notebooks. The contribution of **Código CNEFE 2022** is not a novel algorithm, but a **reference implementation** with **clear contracts** (columns, CRS, exports) that other researchers can adopt, validate, and extend — particularly when combining CNEFE with **census tables**, **street networks**, or **land‑use layers** in reproducible studies.

# Software description

## Data model and categories

The pipeline reads a municipal **GeoJSON (EPSG:4674)** where each feature corresponds to a CNEFE address point. The field `COD_ESPECIE` maps to the following **primary categories**, each generating a binary column `cat_*`:

1. `cat_DOMICILIO_PARTICULAR` — Domicílio particular  
2. `cat_DOMICILIO_COLETIVO` — Domicílio coletivo  
3. `cat_AGROPECUARIO` — Estabelecimento agropecuário  
4. `cat_ENSINO` — Estabelecimento de ensino  
5. `cat_SAUDE` — Estabelecimento de saúde  
6. `cat_OUTRAS_FINALIDADES` — Estabelecimento de outras finalidades  
7. `cat_CONSTRUCAO_REFORMA` — Edificação em construção ou reforma  
8. `cat_RELIGIOSO` — Estabelecimento religioso  

A safety net `cat_NAO_CLASSIFICADO` is included for any out‑of‑range values. **Subtype** flags (`sub_*`) are created **only** where `cat_OUTRAS_FINALIDADES == 1`, using an **editable** keyword dictionary (e.g., `sub_MERCADO`, `sub_PADARIA`, `sub_BANCO`, `sub_BAR`, `sub_RESTAURANTE`, `sub_COMERCIO`, `sub_SERVICOS`, `sub_HOSPEDAGEM`, `sub_CULTURA_LAZER`, `sub_INDUSTRIA`, `sub_TRANSPORTE`, `sub_SEGURANCA`, `sub_ADMIN_PUBLICA`, `sub_OUTROS`).

## Implementation and exports

The pipeline is implemented in **Python** with **GeoPandas** [@geopandas], **Pandas** [@mckinney2010; @pandas], **Shapely** [@shapely], **PyProj** [@pyproj], and **Fiona/Pyogrio** [@fiona; @pyogrio]. Category columns are computed with **vectorized boolean masks** from `COD_ESPECIE`. Subtypes are derived through normalized text matching **only** inside `cat_OUTRAS_FINALIDADES`, isolating ambiguity and enabling local customization without changing the primary logic.

Outputs include:  
- **Per‑point layers** with `cat_*` and `sub_*` in **GeoPackage (GPKG)** and **GeoJSON (EPSG:4326)**;  
- **Aggregations** by `NUM_FACE` and `COD_SETOR` saved as **CSV** (groupby‑sum of dummy columns).

The repository ships both a **Jupyter notebook** (`CNEFE_2022_wide_v3.ipynb`) and a **script** (`cnefe2022_wide_v3.py`) for batch runs.

# Quality control and validation

We include lightweight but effective checks:

- **Schema & typing**: verify the presence of `COD_ESPECIE`, `NUM_FACE`, `COD_SETOR`; coerce identifiers to string and **zero‑pad** when numeric.  
- **One‑of‑N**: confirm that exactly one primary `cat_*` is set per record (except for `cat_NAO_CLASSIFICADO`).  
- **CRS handling**: set **EPSG:4674** if missing; reproject to **EPSG:4326** for GeoJSON.  
- **Sanity totals**: emit quick sums of `cat_*`/`sub_*` and non‑zero counts by block face / sector after aggregation.  
- **Determinism**: avoid random seeds or data‑dependent branching; the same input yields the same outputs.

# Performance notes

Processing **~10⁴–10⁵** points runs in minutes on commodity laptops (vectorized operations, fast I/O via Pyogrio/Fiona). Memory use scales linearly with the number of features; the “wide” schema adds dozens of boolean columns but remains compact (ints 0/1). For very large municipalities, chunked reading or column pruning can be added without altering the data model.

# Reuse and extensibility

The design intentionally **modularizes** the only “open‑world” component — the **subtype dictionary**. Users can localize or extend subtypes (e.g., separar `FARMÁCIA` de `SAÚDE` quando cadastrada em “outras finalidades”) while keeping the **primary** category flags and outputs stable. The aggregated CSVs are ready for **joins** with faces/sectors in GIS or for statistical modeling (e.g., accessibility, exposure, morphology).

# Example

A municipal CNEFE 2022 GeoJSON with ~**12k** points yields non‑zero totals across **domicílios**, **agropecuário**, **construção/reforma**, **ensino**, **saúde**, **religioso**, and a meaningful share of **outras finalidades** entries with informative `sub_*`. Aggregations by `NUM_FACE`/`COD_SETOR` join directly to cartographic layers for mapping and exploratory analysis.

# Installation

```bash
pip install geopandas pyogrio shapely fiona pyproj pandas
```

# Usage

**Notebook**: open `CNEFE_2022_wide_v3.ipynb`, set `INPUT_JSON`, run all cells.  
**Script**: place a municipal GeoJSON next to `cnefe2022_wide_v3.py` and run:

```bash
python cnefe_2022_v03.py
```

Outputs are written to `outputs/` (GPKG/GeoJSON; `*_aggr_face.csv`, `*_aggr_setor.csv`).

# Limitations

- The choice of **subtype keywords** reflects common Brazilian terms; local vocabularies may require adjustment.  
- Accuracy depends on the **quality and completeness** of `DSC_ESTABELECIMENTO` when deriving subtypes.  
- The pipeline does **not** infer geometry for 2010; spatial joins must be provided by the user for legacy data.

# Acknowledgements

We thank **IBGE** for providing CNEFE data and the maintainers of **GeoPandas**, **Shapely**, **PyProj**, **Pyogrio**, **Fiona**, and **Pandas**. Feedback from colleagues on classification vocabularies is gratefully acknowledged. Any errors are the author’s own.

# References
