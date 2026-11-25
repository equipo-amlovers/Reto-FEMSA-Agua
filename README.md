# Reto-FEMSA-Agua

Auditoría de descargas de agua por planta (enero-octubre 2026): conciliación entre la bitácora de las plantas y Reporting Factory, con KPIs y semáforo de cumplimiento para Auditoría Interna.

## Código (etapa 2 — SQL/SQLite)

Pipeline en capas bronce/plata/oro, construido sobre `sqlite3`:

| Archivo | Capa | Qué hace |
| --- | --- | --- |
| `ingesta.py` | Bronce | Lee los Excel originales (bitácora + Reporting Factory) y los vuelca sin transformar a tablas `raw_*` en SQLite |
| `exploracion.sql` | — | Perfilado y hallazgos (E1-E19): huecos, fórmulas rotas, negativos, picos, ceros, comparación bitácora vs RF |
| `limpieza.sql` | Plata/Oro | Reglas R1-R9: catálogo de plantas, calendario real, interpolación de lecturas faltantes, volumen diario, detección de error de unidad en RF, KPIs y semáforo |
| `resultados.sql` / `v1.sql` | Consumo | Consultas de resumen por planta, evolución mensual y validaciones de control |
| `runsql.py` | Utilidad | Ejecuta cualquier `.sql` contra la base e imprime el resultado |
| `modelo_estrella.sql` | Etapa 4 | Materializa el esquema en estrella de Etapa 3: `dim_tiempo_mes`, `dim_diagnostico` (dimensión basura) y `fact_conciliacion_mensual` (hecho limpio, solo llaves + métricas) |
| `exportar_modelo_estrella.py` | Etapa 4 | Exporta las 4 tablas del esquema en estrella a un Excel para conectar Power BI sin driver ODBC |

### Cómo correrlo

Los Excel originales y la base generada (`femsa.db`) **no se suben al repo** por confidencialidad (ver `.gitignore`). Para reproducir:

```bash
pip install openpyxl pandas
python ingesta.py "Bitacora descargas 2026.xlsx" "ReportingFactory Ene26 - Oct26.xlsx" femsa.db
python runsql.py femsa.db limpieza.sql
python runsql.py femsa.db exploracion.sql
python runsql.py femsa.db resultados.sql
python runsql.py femsa.db v1.sql
python runsql.py femsa.db modelo_estrella.sql
python exportar_modelo_estrella.py
```
