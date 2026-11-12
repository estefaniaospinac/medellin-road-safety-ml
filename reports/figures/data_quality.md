# Data quality log

Dataset: Medellín road traffic incidents 2008–2025 (Mendeley Data)
Raw records: 702,540

| # | Column | Issue | Affected rows | % | Decision | Impact on model |
|---|---|---|---|---|---|---|
| 1 | Codigo Comuna | Mixed types: numeric codes + "N/D" | 28.154 | 4.01 % | Converted "N/D" to null, cast to Int64 | Rows without comuna excluded from zone-level analysis |
| 2 | HORA_INCIDENTE | Mixed types: 595,305 Excel time objects + 107,235 text values | 107,235 | 15.26 % | Cast to string; hour (0–23) extracted in cleaning | Unparseable hours excluded from time features |