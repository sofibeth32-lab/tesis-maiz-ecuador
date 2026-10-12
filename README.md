# Pronóstico de Producción, Demanda e Importaciones de Maíz Duro Amarillo en Ecuador

Tesis de Maestría en Big Data. Todo el análisis está en un único notebook,
[`TESIS_COMPLETA.ipynb`](TESIS_COMPLETA.ipynb), que se ejecuta de principio a fin desde los datos
crudos incluidos en `datos/`.

## Resultados principales

| Etapa | Resultado |
|---|---|
| 1. Producción — R² LOYO (provincia-año) | 0.967 |
| 1. Producción — error nacional por año (LOYO) | MAPE 10.9% · R² nacional 0.134 |
| 1. Producción 2026 | 1,493,945 Tm |
| 2. Demanda estimada (promedio USDA-PSD 2023-2025) | 1,508,333 Tm |
| 3. Importaciones — modelo final | Producción + Precio, log-target (MAE 38,603 Tm) |
| 3. Importaciones Necesarias 2026 | 65,404 Tm (intervalo de predicción 80%: 30,316 – 141,106 Tm) |
| 3. Reducción del MAE frente a la identidad contable | 78% (Wilcoxon p = 0.002) |

## Metodología

1. **Producción (Etapa 1):** panel provincia-año 2014-2025 construido desde los microdatos y
   Tabulados de la ESPAC (INEC), con variables climáticas de Copernicus ERA5 interactuadas por
   región (Costa / Sierra). Modelo XGBoost con búsqueda de 192 combinaciones de hiperparámetros,
   validado con Leave-One-Year-Out y Group K-Fold por provincia.
2. **Demanda (Etapa 2):** comparación de seis métodos de series de tiempo (promedio histórico,
   Random Walk, regresión lineal, AR(1), Holt, Auto-ARIMA) sobre tres fuentes (USDA-PSD, cálculo
   propio ESPAC + BCE, FAOSTAT).
3. **Importaciones (Etapa 3):** seis variantes, desde la identidad contable de balance hasta
   regresiones con Producción y Precio internacional, evaluadas con Leave-One-Out. El modelo final
   se elige por menor MAE.

La **Parte 5B** reproduce los capítulos de resultados y discusión: réplica de las soluciones
alternativas de la literatura (Random Forest, regresión lineal múltiple, tendencia lineal y ARIMA, con
los mismos pliegues y métricas, y prueba de Wilcoxon), evaluación con origen móvil de la demanda,
inferencia del modelo final de importaciones (coeficientes, intervalos de confianza, Shapiro-Wilk,
Breusch-Pagan, Durbin-Watson), intervalo de predicción y escenarios 2026, y traducción monetaria del
error. Sus figuras se guardan en `resultados/figuras/`.

Antes de las conclusiones, el notebook incluye una sección de **análisis exploratorio y pruebas sobre la naturaleza de los datos** (normalidad, linealidad, estacionariedad ADF/KPSS, multicolinealidad y matrices de correlación), que reproduce las cifras y figuras del capítulo de metodología de la tesis; las figuras se guardan en `resultados/figuras/`.

### Corrección de la producción 2016-2017

En los `.csv` de la ESPAC 2016 y 2017, la variable de producción derivada (`ct_prod`) solo está
presente en el 2.7% de los registros de maíz duro seco. El notebook la reconstruye con la misma
regla que el INEC aplica en 2014-2015 (`ct_k516 × ct_k518 / 2200`: cantidad cosechada × libras
por unidad / 2200). Luego valida el resultado contra los Informes Ejecutivos ESPAC 2016-2017: el
total nacional difiere en +1.6% y +0.1%, y Los Ríos, Manabí y Guayas en menos del 2%. El
diagnóstico completo está en los Pasos 1.11 a 1.14 del notebook.

## Estructura del repositorio

```
├── TESIS_COMPLETA.ipynb     notebook principal (con todas las salidas)
├── requirements.txt         versiones de las librerías usadas
├── datos/
│   ├── ESPAC/               microdatos 2014-2017 y Tabulados 2018-2025 (INEC)
│   ├── Copernicus/          clima mensual ERA5 (.nc)
│   ├── BCE/                 importaciones por subpartida (Banco Central del Ecuador)
│   ├── FRED/                precio internacional del maíz (FMI, vía FRED)
│   ├── FAOSTAT/             balance de alimentos de la FAO
│   ├── USDA/                balance USDA-PSD (copia local de la API)
│   └── Mapa/                límites provinciales de Ecuador (GeoJSON)
└── resultados/              tablas exportadas a Excel y figuras del capítulo de metodología
```

## Cómo ejecutarlo

**Local:**

```bash
git clone <url-del-repositorio>
cd tesis-maiz-ecuador
pip install -r requirements.txt
jupyter notebook TESIS_COMPLETA.ipynb
```

**Google Colab:** subir la carpeta `datos/` a `Mi unidad/Colab Notebooks/Tesis/`, abrir el
notebook y ejecutar todas las celdas. El notebook detecta que está en Colab, instala las
librerías y monta Google Drive.

La ejecución completa tarda unos 20-25 minutos.

### API del USDA (opcional)

Por defecto el notebook usa la copia local `datos/USDA/usda_psd_maiz_ecuador.csv`, que reproduce
exactamente los resultados publicados. Para consultar la API en vivo, se puede definir la clave
como variable de entorno (o como secreto `USDA_API_KEY` en Colab):

```bash
export USDA_API_KEY="tu_clave"
```

La clave nunca se guarda en el notebook. Si la API trae cifras revisadas, los resultados de las
Etapas 2 y 3 pueden cambiar ligeramente.

## Reproducibilidad

- Semillas fijas: XGBoost usa `random_state=42` y el reparto de provincias en Group K-Fold usa
  `semilla=42`. El reparto se hace en el propio notebook para que no dependa de la versión de
  scikit-learn.
- Las versiones exactas usadas están en `requirements.txt` (Python 3.13).

## Fuentes de datos

- INEC — Encuesta de Superficie y Producción Agropecuaria Continua (ESPAC), 2014-2025.
- Copernicus Climate Change Service — ERA5 monthly averaged data on single levels.
- Banco Central del Ecuador — estadísticas de comercio exterior por subpartida (1005901100).
- Fondo Monetario Internacional — precio global del maíz (PMAIZMTUSDA), vía FRED.
- FAO — FAOSTAT, Food Balances.
- USDA Foreign Agricultural Service — Production, Supply and Distribution (PSD).
