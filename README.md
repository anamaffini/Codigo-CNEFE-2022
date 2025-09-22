# Código CNEFE — Reproducible pipelines for CNEFE 2022

## By Ana Luisa Maffini

Tools to parse, classify, and aggregate Brazil’s **CNEFE** address data. The repo harmonizes the non-spatial **2010** release and the **2022** GeoJSON points into a common **“wide”** format (one-hot columns), plus block-face and census-sector summaries.

---

## What’s inside 

- **2022 workflow** (GeoJSON points → wide categories + subtypes)
- Consistent outputs: **GPKG**, **GeoJSON**, **CSV** (per-point + aggregated by `NUM_FACE` and `COD_SETOR`)
- Explicit, auditable logic with preserved zero-padding for codes (UF, municipality, CEP)

---

## Category mapping (CNEFE 2022)

`COD_ESPECIE` → `cat_*` columns:

| Code | Meaning                                   | Column                  |
|-----:|-------------------------------------------|-------------------------|
| 1    | Domicílio particular                      | `cat_DOMICILIO_PARTICULAR` |
| 2    | Domicílio coletivo                        | `cat_DOMICILIO_COLETIVO`   |
| 3    | Estabelecimento agropecuário              | `cat_AGROPECUARIO`         |
| 4    | Estabelecimento de ensino                 | `cat_ENSINO`               |
| 5    | Estabelecimento de saúde                  | `cat_SAUDE`                |
| 6    | Estabelecimento de outras finalidades     | `cat_OUTRAS_FINALIDADES`   |
| 7    | Edificação em construção ou reforma       | `cat_CONSTRUCAO_REFORMA`   |
| 8    | Estabelecimento religioso                 | `cat_RELIGIOSO`            |

**Subtypes (`sub_*`)** are created **only** when `cat_OUTRAS_FINALIDADES == 1` using an editable keyword dictionary (e.g., `sub_MERCADO`, `sub_PADARIA`, `sub_BANCO`, `sub_BAR`, `sub_RESTAURANTE`, `sub_COMERCIO`, `sub_SERVICOS`, `sub_HOSPEDAGEM`, `sub_CULTURA_LAZER`, `sub_INDUSTRIA`, `sub_TRANSPORTE`, `sub_SEGURANCA`, `sub_ADMIN_PUBLICA`, `sub_OUTROS`).  
A safety column `cat_NAO_CLASSIFICADO` is kept for out-of-range values.

---

## Files you’ll use most

- **`CNEFE_2022_wide_v3.ipynb`** – Notebook for 2022 points → wide categories/subtypes → exports/aggregations.  
- **`cnefe2022_wide_v3.py`** – Script with the same pipeline (batch-friendly).  
- (Optional) 2010 notebooks/scripts for fixed-width parsing and harmonization.

---

## Installation

```bash
# Python ≥ 3.10 recommended
pip install geopandas pyogrio shapely fiona pyproj pandas
```

QGIS/OSGeo environments also work.

---

## Quick start (2022)

**Notebook**

1. Open `CNEFE_2022_wide_v3.ipynb`.
2. Set `INPUT_JSON` to your municipal GeoJSON file.
3. Run all cells.

**Script**

```bash
python cnefe2022_wide_v3.py
```

**Outputs** (in `outputs/`):
- Per-point with dummies: `cnefe2022_wide_v3.gpkg` (layer `cnefe2022_wide_v3`), `cnefe2022_wide_v3.geojson`
- Aggregations: `cnefe2022_aggr_face_v3.csv` (by `NUM_FACE`), `cnefe2022_aggr_setor_v3.csv` (by `COD_SETOR`)

---

## Quick start (2010)

1. Use the 2010 parsing notebook to read **fixed-width TXT** and **preserve zero-padding**.
2. (Optional) Spatialize by joining to faces/sectors using the appropriate keys.
3. Export wide outputs and aggregated tables, mirroring the 2022 structure.

---

## Customization

- **Subtypes**: edit `SUBTIPO_KEYWORDS` in the notebook/script to add/remove terms or new `sub_*` columns.
- **Column expectations**: 2022 flow uses `COD_ESPECIE`, `DSC_ESTABELECIMENTO`, `NUM_FACE`, `COD_SETOR`.  
- **CRS**: inputs assumed EPSG:4674; GeoJSON exports in EPSG:4326.

---

## Troubleshooting

- **Only one category has 1s** → verify `COD_ESPECIE` exists and is mapped as **string** in {1..8}.  
- **Aggregations empty** → ensure `NUM_FACE` / `COD_SETOR` are present.  
- **CRS warnings** → the pipeline sets EPSG:4674 if missing; adjust if your source differs.  
- **Write errors** → confirm `outputs/` exists (created automatically) and you have permissions.

---

## Reproducibility & design notes

- Vectorized boolean logic for categories (no brittle text parsing).  
- Subtype matching isolated in one step for easy review and localization.  
- Consistent schema across 2010/2022 to keep downstream analytics stable.

---

## Data & licensing

- **Data**: CNEFE is published by IBGE. Respect the IBGE terms of use for datasets you process.  
- **Citation**: see `paper.md'.

---

## Contributing

Issues and pull requests are welcome: bug reports, new subtype vocabularies, tests, or performance tweaks. Please add concise examples and expected outputs.
