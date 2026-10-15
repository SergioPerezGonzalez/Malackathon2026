# Malackathon 2026 — Clustering the Star (Mercedes-Benz)

Segmentación de clientes de Mercedes-Benz con aprendizaje no supervisado (K-Means) para personalizar el marketing: ofertas de vehículos, planes de mantenimiento, fidelización y comunicaciones dirigidas.

El enunciado y el diccionario de datos están en [`Reto Malackathon _MercedesBenz_2026.pdf`](Reto%20Malackathon%20_MercedesBenz_2026.pdf). Para una lectura rápida de los retos básicos (resultados y decisiones), ver [`docs/resumen-retos-basicos.pdf`](docs/resumen-retos-basicos.pdf).

## Resultado

De 302.850 filas cliente–vehículo se obtienen **173.454 clientes** con 19 features. K-Means con **k = 5** (el codo y el máximo del Silhouette coinciden; resultado idéntico con 5 semillas) da cinco segmentos de negocio:

| Segmento | % clientes | Qué lo define | Acción de marketing |
|---|---|---|---|
| **Fieles Acomodados** | 28,6 | Mayor antigüedad y poder adquisitivo, leasing, Suiza | Fidelización y renovación anticipada |
| **Candidatos a Renovación** | 33,7 | Coche de combustión antiguo, sin financiación, más joven | Renovación con bono de entrega + financiación |
| **Pioneros Eléctricos** | 15,1 | 94 % enchufables, coche más nuevo | Servicios de carga y renovación a nuevos EQ |
| **Clientes Silenciosos** | 12,5 | Solo aceptan ePost | Recuperar el consentimiento |
| **Clientes Perdidos** | 10,1 | Sin vehículo actual | Campaña de recuperación (win-back) |

## Entregables

| Reto | Notebook |
|---|---|
| Básico 1 — EDA y calidad de datos | [`notebooks-básico/01_eda.ipynb`](notebooks-básico/01_eda.ipynb) |
| Básico 2 — Feature engineering y pipeline | [`notebooks-básico/02_feature_engineering.ipynb`](notebooks-básico/02_feature_engineering.ipynb) |
| Básico 3 — K-Means (codo y Silhouette) | [`notebooks-básico/03_kmeans.ipynb`](notebooks-básico/03_kmeans.ipynb) |
| Básico 4 — Interpretación de segmentos | [`notebooks-básico/04_segmentos.ipynb`](notebooks-básico/04_segmentos.ipynb) |
| Avanzado 1 — Comparativa con GMM y DBSCAN | [`notebooks-avanzado/05_comparativa.ipynb`](notebooks-avanzado/05_comparativa.ipynb) |
| Avanzado 2 — PCA y visualización | [`notebooks-avanzado/06_pca.ipynb`](notebooks-avanzado/06_pca.ipynb) |

Cada notebook empieza con un **resumen** (qué se hizo, resultados clave y decisiones) y explica cada tabla y gráfico con una celda de interpretación.

## Cómo ejecutarlo

Requisitos: Python (probado con 3.13) y [uv](https://docs.astral.sh/uv/) (o `pip`).

```bash
uv venv
uv pip install -r requirements.txt ipykernel
```

Abrir los notebooks con el kernel de `.venv` y ejecutarlos **en este orden**, porque cada uno genera ficheros que usan los siguientes:

1. `01_eda.ipynb` — independiente.
2. `02_feature_engineering.ipynb` — genera `data/processed/X_model.csv` y `client_features.csv`.
3. `03_kmeans.ipynb` — genera `data/processed/clusters.csv` (tarda ~10 minutos: 55 modelos).
4. `04_segmentos.ipynb`, `05_comparativa.ipynb` y `06_pca.ipynb` — leen los ficheros anteriores.

`data/processed/` no se sube al repositorio (`.gitignore`): hay que generarlo ejecutando el 02 y el 03.

## Datos

`datos_reto_mercedes_benz_malackathon.csv` (95,5 MB, 302.850 filas × 38 columnas) está incluido en el repositorio. SHA-256 del fichero correcto:

```
fb58cecbb924f358967fb5c95259b5f726901b6a0e0a3cc38b849665f48c1dde
```

Una descarga desde Drive llegó truncada sin avisar: el notebook 01 comprueba el SHA-256 al cargar el fichero.

## Estructura

```
├── datos_reto_mercedes_benz_malackathon.csv   datos del reto
├── notebooks-básico/                          retos básicos 1–4
├── notebooks-avanzado/                        retos avanzados
├── docs/resumen-retos-basicos.pdf             resumen para mentores
├── data/processed/                            ficheros generados (no versionados)
├── Apuntes Clustering/, Apuntes Python/, EDA.pdf   apuntes de la asignatura usados como referencia
└── requirements.txt
```
