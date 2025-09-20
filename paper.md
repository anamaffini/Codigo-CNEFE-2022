
---
title: 
# Código CNEFE: reproducible pipelines for classifying and aggregating Brazil’s CNEFE 2022 address data
tags:
  - geospatial
  - GIS
  - Python
  - urban planning
  - Brazil
  - census

author: Ana Luisa Maffini

    orcid: 0000-0001-5334-7073

date: 2025-09-20

bibliography: paper.bib
---

# Summary

Brazil’s **Cadastro Nacional de Endereços para Fins Estatísticos (CNEFE)** is a foundational address register used by the Brazilian Institute of Geography and Statistics (IBGE) to support census operations and household sampling. Despite its wide relevance for urban research and public policy, working with CNEFE across releases can be cumbersome: the **2010** distribution arrives as fixed‑width text without geometry, whereas **2022** is provided as **GeoJSON points**, with different category codifications. **Código CNEFE** delivers **reproducible, open workflows** that harmonize these two eras, producing standardized, analysis‑ready layers for urban morphology and planning.

Our contribution is a **transparent classification and aggregation pipeline** that: (a) maps **CNEFE 2022** `COD_ESPECIE` (codes 1–8) into **one‑hot (“wide”) columns** (`cat_*`), (b) derives **subtype dummies** (`sub_*`) for **“Outras Finalidades”** via editable keyword rules, (c) mirrors the **2010** notebook’s logic so downstream scripts expecting the 2010 “wide” shape keep working, and (d) exports **GeoPackage**, **GeoJSON**, and **CSV** summaries **by block face (`NUM_FACE`) and by census sector (`COD_SETOR`)**. The project emphasizes **zero‑padding preservation** of IDs, CRS handling, and auditability.

# Statement of need

Urban analysts, planners, and public institutions frequently need to **join CNEFE** with block‑face or tract‑level geometries, **classify** address records into policy‑relevant groups, and **aggregate** results for mapping and modeling. However, existing public workflows are either **ad‑hoc**, **underdocumented**, or tailored to a single vintage of the data. The shift from non‑spatial 2010 records to spatial 2022 points — with **changed category semantics** — complicates reuse.

**Código CNEFE** addresses this gap with a **minimal, well‑documented pipeline** that standardizes classification across releases, keeps outputs **backward‑compatible** with 2010 “wide” conventions, and **modularizes** the only ambiguous part (keyword‑based subtyping of “Outras Finalidades”). Researchers can readily integrate the outputs into **GeoPandas**, **QGIS**, or statistical workflows (e.g., inequality metrics, accessibility, or urban morphology).

# Functionality

- **Direct 2022 mapping**: `COD_ESPECIE` ∈ {1..8} → `cat_DOMICILIO_PARTICULAR`, `cat_DOMICILIO_COLETIVO`, `cat_AGROPECUARIO`, `cat_ENSINO`, `cat_SAUDE`, `cat_OUTRAS_FINALIDADES`, `cat_CONSTRUCAO_REFORMA`, `cat_RELIGIOSO`; plus `cat_NAO_CLASSIFICADO` as a safety net.
- **Subtypes for code 6** (*Outras Finalidades*): `sub_MERCADO`, `sub_PADARIA`, `sub_BANCO`, `sub_BAR`, `sub_RESTAURANTE`, `sub_COMERCIO`, `sub_SERVICOS`, `sub_HOSPEDAGEM`, `sub_CULTURA_LAZER`, `sub_INDUSTRIA`, `sub_TRANSPORTE`, `sub_SEGURANCA`, `sub_ADMIN_PUBLICA`, `sub_OUTROS`. The dictionary is **user‑editable**.
- **Outputs**: per‑point layers with dummies (**GPKG**, **GeoJSON**), and **CSV aggregates** by `NUM_FACE` and `COD_SETOR`.
- **2010 compatibility**: a companion notebook reproduces the **wide** format from 2010, ensuring continuity for existing analyses.
- **Robustness features**: zero‑padding for codes (UF, municipality, CEP); explicit boolean logic (no brittle text heuristics for the main categories); consistent CRS (input EPSG:4674, GeoJSON in EPSG:4326).

# Design/implementation

The pipeline is implemented in **Python** using **GeoPandas** [@geopandas], **Pandas** [@mckinney2010], **Shapely** [@shapely], **PyProj** [@pyproj], and **Pyogrio/Fiona** [@fiona; @pyogrio]. The classification is **vectorized** over columns (no Python loops over features), which keeps runtimes practical even for **~10⁵–10⁶** records on commodity hardware. The only optional text processing appears in the **subtype** stage, which is intentionally isolated and easily modified or localized (e.g., to refine “farmácia” handling).

# Quality control

We include lightweight but useful checks in the notebooks/scripts:

1. **Schema checks**: verify that expected input columns exist; warn (or fail) otherwise.  
2. **Sanity totals**: confirm that exactly **one** `cat_*` flag is set per record (except `cat_NAO_CLASSIFICADO`).  
3. **CRS audit**: set EPSG:4674 if missing; reproject to 4326 for GeoJSON.  
4. **Deterministic aggregations**: groupby‑sums over `NUM_FACE` and `COD_SETOR`, stored as CSV for external validation.

For continuous integration, the repository includes **scriptable entry points** (e.g., `cnefe2022_wide_v3.py`) so that maintainers can run sample jobs in CI with hash checks on outputs (practical for synthetic subsets).

# Installation

The notebooks are environment‑light. A minimal install is:

```bash
pip install geopandas pyogrio shapely fiona pyproj pandas
```

QGIS users can also run the `.py` script via OSGeo4W shells. We recommend **Python ≥ 3.10**.

# Usage

**Notebook**: open `CNEFE_2022_wide_v3.ipynb`, set `INPUT_JSON`, run all cells.  
**Script**: place `qg_*.json` next to `cnefe2022_wide_v3.py` and run:

```bash
python cnefe2022_wide_v3.py
```

Outputs are written to `outputs/` (GPKG/GeoJSON + `*_aggr_face.csv` and `*_aggr_setor.csv`).

# State of the field

Several excellent Python geospatial libraries exist (GeoPandas, Shapely, PyProj), but **domain‑specific** tooling for **CNEFE harmonization** across 2010 and 2022 is limited and scattered across private scripts or institutional notes. **Código CNEFE** focuses on this niche by publishing a **documented, minimal** baseline that researchers can extend (e.g., to add quality labels, merge with block‑face polygons, or compute urban accessibility metrics).

# Example

Processing a municipal CNEFE 2022 GeoJSON (~12k records) on a laptop completes in a few minutes, yielding non‑zero totals across all categories (e.g., domiciles, agropecuário, construção/reforma, ensino, saúde, religioso), and practical aggregates by sector and block face that can be joined to cartographic layers for immediate mapping.

# Acknowledgements

We thank **IBGE** for making CNEFE data available and the maintainers of **GeoPandas**, **Shapely**, **PyProj**, **Pyogrio**, **Fiona**, and **Pandas**. The authors acknowledge colleagues who provided feedback on classification vocabularies. Any errors are our own.

# References
