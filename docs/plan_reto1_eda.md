# Plan — Reto 1: Exploración y calidad de datos (EDA)

Objetivo: **describir, diagnosticar y proponer**. Este reto no limpia ni selecciona features para el modelo (eso es el Reto 2); documenta qué hay, qué está mal y qué haremos con ello.

**Entregables**

1. `notebooks/01_eda.ipynb` ejecutado, con una celda markdown de interpretación después de cada tabla o gráfico.
2. Resumen final que responda literalmente:
   - ¿Cuáles son las principales características de los clientes?
   - ¿Qué problemas de calidad habéis encontrado y cómo los habéis resuelto?

---

## 0. Datos de partida

| Hecho | Valor |
|---|---|
| Fichero | `datos_reto_mercedes_benz_malackathon.csv` (95,5 MB) |
| SHA-256 válido | `fb58cecbb924f358967fb5c95259b5f726901b6a0e0a3cc38b849665f48c1dde` |
| Filas / columnas | 302.850 × 38 (+ índice sin nombre) |
| Clientes únicos | 173.456 |
| Vehículos únicos | 244.519 |
| Granularidad | **cliente–vehículo** (media 1,75 filas por cliente, máx. 561) |
| Mercados | CH (~64 %) y AT (~36 %) |

Lectura: `pd.read_csv(path, index_col=0)`.

---

## 1. Estructura del notebook

### 1.1 Carga y visión general
- [ ] Carga, `shape`, `dtypes`, `head()`.
- [ ] Comprobar integridad: índice continuo 0…302849 y 39 campos por línea (ver problema Q0).
- [ ] Explicar la granularidad: filas vs `client_id` únicos vs `vehicle_id` únicos; vehículos compartidos por varios clientes (owner + driver).
- [ ] Verificar qué columnas son de nivel cliente (constantes por `client_id`): `age_range`, `gender_derived`, `months_as_mb_customer`, `currently_connected_vehicles`, `finance_leasing_status`, consentimientos, `zipcode`, `country_code` → todas constantes. `role_type` varía en 545 clientes.

### 1.2 Estadísticas descriptivas
- [ ] Numéricas: `describe(percentiles=[.01, .25, .5, .75, .99])`.
- [ ] Categóricas: `value_counts(normalize=True)` de `age_range`, `gender_derived`, `model_class`, `product_group`, `engine_concept`, `fuel_type`, `country_code`, `role`, `role_type`, `finance_leasing_status`, `is_current_vehicle` y los tres consentimientos.
- [ ] Cardinalidad (`nunique`) de todas las columnas.

### 1.3 Valores nulos
- [ ] Tabla y gráfico de barras de % de nulos por columna.
- [ ] Mapa de nulos (`missingno` o heatmap de `isna()`) para mostrar que la telemetría falta **en bloque**.
- [ ] Comprobar si los nulos son aleatorios: % de nulos de telemetría por `is_current_vehicle` (74 % en no actuales vs 53 % en actuales) → **no aleatorio**.

### 1.4 Inconsistencias
- [ ] Crosstab `engine_concept` × `fuel_type` (coherencia y posibilidad de imputación cruzada).
- [ ] Columnas `*_mode` de tipo texto con listas de intervalos.
- [ ] Outliers con IQR y Z-score (apuntes): `speed_average`, `fuel_consumption_average`, `trip_distance_average`, `operating_hours_total`, `workshop_visits_warranty_goodwill`, `speed_all_counter`.
- [ ] Ceros sospechosos: `population` / `number_of_households` = 0.
- [ ] Fechas: rango de `date_of_first_registration` (1987–2026, ninguna futura).
- [ ] Coherencia `currently_connected_vehicles` vs nº de `is_current_vehicle=True` por cliente (100 % de coincidencia).
- [ ] Clientes con muchos vehículos (>10: ~1.600, casi todos B2C) → posibles flotas mal etiquetadas.

### 1.5 Distribuciones de variables clave
- [ ] Histogramas + boxplots: `months_as_mb_customer`, `purchasing_power_per_household_eur`, `population`, `workshop_visits_warranty_goodwill`, `currently_connected_vehicles`, `operating_hours_total`, `trip_distance_average`, `speed_average`. Comentar asimetría (justifica log en Reto 2).
- [ ] Barras: `age_range` (ordenada), `model_class` (top 15), `engine_concept`, `finance_leasing_status`, `role`.
- [ ] Años de matriculación (tendencia de compra; crecimiento hasta 2023).

### 1.6 Redundancia
- [ ] Matriz de correlación de Spearman de las numéricas.
- [ ] Señalar pares |ρ| > 0,8 (criterio de los apuntes):

| Par | ρ |
|---|---|
| `population` – `number_of_households` | 1,00 |
| `population` – `purchasing_power_millions_eur` | 0,97 |
| `purchasing_power_per_inhabitant_eur` – `purchasing_power_per_household_eur` | 0,96 |
| `number_of_households` – `purchasing_power_millions_eur` | 0,96 |
| `speed_average` – `speed_median` | 0,85 |

- [ ] Demostrar que `purchasing_power_millions_eur` = `per_household × number_of_households / 1e6` (error relativo mediano ≈ 0).
- [ ] Mostrar que `role_type = B2B` ⇔ `role = DRIVER` (5.231 filas en el fichero truncado; recalcular).

### 1.7 Exclusión de identificadores y metadatos
- [ ] Eliminar y justificar (solo lo que **no aporta valor analítico**; la selección por criterio de negocio va al Reto 2):

| Columna | Motivo |
|---|---|
| índice sin nombre | Artefacto de exportación |
| `client_id` | Identificador hash (se conserva solo como clave de agregación) |
| `vehicle_id` | Identificador hash |
| `zipcode` | Identificador geográfico; su información ya está en las variables socioeconómicas |
| `trip_distance_all_counter`, `speed_all_counter` | Metadatos de cálculo de las métricas, no comportamiento |
| `trip_distance_mode`, `speed_mode` | Texto con listas de intervalos; inutilizables |

### 1.8 Resumen y conclusiones
- [ ] Perfil del cliente (ver §3).
- [ ] Tabla de problemas de calidad y tratamiento (ver §2).
- [ ] Decisiones que se trasladan al Reto 2 (ver §4).

---

## 2. Problemas de calidad y tratamiento propuesto

| # | Problema | Evidencia | Tratamiento propuesto |
|---|---|---|---|
| Q0 | Fichero inicial truncado al descargar de Drive | Faltaban 17.925 filas (índice 23256–41180) y una línea empalmada con 52 campos | Re-descarga y verificación por SHA-256, índice continuo y 39 campos por línea. **Resuelto** |
| Q1 | Granularidad cliente–vehículo | 302.850 filas para 173.456 clientes | Agregar a una fila por cliente antes de clusterizar (Reto 2) |
| Q2 | Telemetría ausente ~60 % | Nulos en bloque; dependen de `is_current_vehicle` (no aleatorios) | No imputar a ciegas: flag `has_telemetry` + agregación por cliente; mediana solo si se usa en el modelo |
| Q3 | `engine_concept` 44 % nulos, `fuel_type` 59 % | Crosstab: son coherentes entre sí | Imputación cruzada (`ELECTRIC`↔`ELECTRIC`, `*,ELECTRIC`→`PLUGIN_HYBRID`), luego por `vehicle_type` (EQ*, ` d`, ` e`/`de`); resto `UNKNOWN` |
| Q4 | `gender_derived` 11 % nulos y es inferido | Diccionario: "no es un dato declarado" | Categoría `unknown`; usar solo para perfilado |
| Q5 | Socioeconómicas 6,3 % nulos; `zipcode` 5,3 % | Hay filas con zipcode pero sin socioeconómicas | Mediana por `country_code` |
| Q6 | `population` / `number_of_households` = 0 | 73 filas | Tratar como NaN e imputar como Q5 |
| Q7 | Columnas `*_mode` de texto con listas | Valores como `"2.5,7.5,15,30,60,100,180"` | Eliminar (ya tenemos media y mediana) |
| Q8 | Outliers | `speed_average` > 150 (~25), `fuel_consumption_average` > 25 l (~600), `speed_all_counter` máx. 1,45·10⁹ | Winsorizar p1–p99 y log en contadores/horas (Reto 2) |
| Q9 | Redundancias | Tabla §1.6 | Conservar una por grupo (decisión en Reto 2) |
| Q10 | Posibles flotas etiquetadas B2C | ~1.600 clientes con >10 vehículos | Flag `is_fleet`; analizar aparte o excluir para no distorsionar K-Means |
| Q11 | `date_of_first_registration` 1,4 % nulos | — | Imputar `vehicle_age` con la mediana por `model_series` |
| Q12 | `months_as_mb_customer` = 0 | ~8.600 filas | Válido (clientes nuevos); no se trata |

---

## 3. Perfil del cliente (borrador para el resumen)

Cifras definitivas a nivel **cliente** (fichero completo) en la sección 7 de `notebooks/01_eda.ipynb`. Resumen:

- **Demografía:** 65 % hombres, 18 % mujeres, 17 % sin género inferido; el 61 % tiene 45 años o más.
- **Tipo de cliente:** B2B < 1 % de los clientes; 87 % de las relaciones son de propietario (OWNER).
- **Vehículos:** 73 % de los clientes con un único vehículo; predominan GLC, C-Class y A-Class; ~41 % electrificados entre los que tienen dato de motorización.
- **Relación con la marca:** antigüedad mediana 36 meses (con un pico sospechoso en 36, ver Q11 del notebook); 71 % nunca ha usado financiación o leasing.
- **Comunicación:** ePost 98 %, teléfono 86 %, correo postal 85 %.
- **Zona:** poder adquisitivo por hogar mediano ≈ 104.000 € (Suiza ≈ 116.000 €, Austria ≈ 67.000 €).

---

## 4. Decisiones que pasan al Reto 2

- Agregación a nivel cliente y features agregadas (`n_vehicles`, `n_current`, `pct_electrified`, `main_model_class`, `has_van`, `is_b2b`, `has_telemetry`, medias de telemetría).
- Feature derivada obligatoria: `vehicle_age` desde `date_of_first_registration`.
- Selección final por criterio de negocio citando la evidencia del EDA (correlaciones, cardinalidad de `vehicle_type`, ~98 % True en `consent_epost`).
- **Decisión abierta:** con ~60 % de nulos en telemetría, ¿modelo base sin telemetría + modelo de clientes conectados, o un único modelo con `has_telemetry`?

---

## 5. Reparto sugerido (equipo de 4)

| Persona | Secciones |
|---|---|
| A | 1.1, 1.2, 1.7 |
| B | 1.3, 1.4 (nulos e inconsistencias) |
| C | 1.5, 1.6 (distribuciones y redundancia) |
| D | 1.8 + §2 y §3 (resumen y conclusiones) |
