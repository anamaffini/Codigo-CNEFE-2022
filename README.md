# Codigo-CNEFE-2022
Reproducible pipeline for Brazil’s CNEFE data: reads 2022 GeoJSON points, maps COD_ESPECIE (1–8) to one-hot cat_*, derives sub_* for “Outras Finalidades”, preserves zero-padding, and exports per-point + aggregated (NUM_FACE, COD_SETOR) outputs to GPKG/GeoJSON/CSV.
