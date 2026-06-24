# Predicción de la tasa de paro trimestral de la EPA en España

Trabajo de Fin de Máster — Máster Universitario en Business Analytics, Universidad Pontificia Comillas (ICADE).

Este repositorio contiene el pipeline completo de datos y modelado del trabajo: la limpieza y unificación de las series, el análisis exploratorio, la construcción del panel macroeconómico, los modelos de predicción y los resultados. El objetivo es predecir la **tasa de paro trimestral de la Encuesta de Población Activa (EPA)** en España, desde el nowcast del trimestre en curso hasta cuatro trimestres por delante.

## Pregunta de investigación

El trabajo responde a dos preguntas ligadas:

1. **¿Se puede predecir la tasa de paro trimestral de la EPA en España, y qué hace falta para ello?**
2. **¿Mejora el aprendizaje automático a la econometría clásica en esta tarea, y por qué sí o por qué no?**

## Resultado principal

La tasa de paro es una serie **fuertemente inercial**: su propio pasado reciente explica casi toda la capacidad de predicción. Sobre 198 observaciones trimestrales (1976Q3–2025Q4), evaluadas sin fuga de información mediante un backtest de ventana expansiva con 116 orígenes de predicción:

- Ningún modelo individual bate a un **ARIMA univariante**, que actúa como benchmark.
- El **panel macroeconómico** construido en el análisis exploratorio tiene valor descriptivo y causa en sentido de Granger (42 de 55 variables), pero **no aporta valor predictivo marginal fuera de muestra**. Es el hallazgo metodológico central: causalidad de Granger en muestra no equivale a valor predictivo.
- El **aprendizaje automático no rescata**: árboles que no extrapolan y sobreajustan, y el mejor lineal regularizado (Lasso) por debajo del paseo aleatorio.
- Solo una **combinación por stacking** supera al ARIMA, de forma modesta pero real (+3,1 % en RMSE global).
- La **COVID es la frontera de lo predecible**: en 2020–2021 el paseo aleatorio ingenuo bate a los modelos sofisticados, que intentan explicar un shock no anticipable.

| Modelo | RMSE global (fuera de muestra) | Papel |
|---|---|---|
| Ensamblaje (stacking) | 1.483 | Modelo final, +3,1 % sobre el benchmark |
| ARIMA | 1.531 | Benchmark, invicto entre los individuales |
| SARIMAX | 1.630 | Las exógenas no compensan el ruido que añaden |
| VAR | 1.694 | Mejor multivariante |
| Paseo aleatorio (random walk) | 1.883 | Suelo de referencia |
| Prophet (en diferencia) | 1.967 | Ancla cuando modela el cambio |
| Lasso | 2.047 | Mejor modelo de aprendizaje automático |

RMSE en puntos de la tasa de paro, agregando nowcast y horizontes 1 a 4. En el subperiodo COVID (2020–2021) el orden se invierte: el paseo aleatorio (1.50) es el más robusto y el ensamblaje hereda la debilidad del ARIMA (2.27). La tabla completa, con métricas por horizonte y skill scores, está en `Resultados/metricas_consolidado.csv`.

## Estructura del repositorio

| Carpeta | Contenido |
|---|---|
| `Limpieza y unión de series temporales/` | Doce cuadernos (0–11), uno por serie, con la banca de activo y pasivo en un mismo cuaderno: limpieza, homogeneización y unificación de cada serie cruda hasta su versión final. Vuelcan a `Datasets/`. |
| `Datasets/` | Series limpias y unificadas en CSV, en su frecuencia original y en su versión trimestral (`_trimestral`), más `panel_base.csv`. |
| `EDA/` | Catorce cuadernos de análisis exploratorio (0–12, con la banca desglosada en 9a activo y 9b pasivo): uno por cada serie principal y el del panel; caracterización, estacionariedad, heterocedasticidad y relación con el paro. |
| `Datasets Modelado/` | Conjuntos listos para modelar, en forma estacionaria, separados por familia y por canal económico (econométrico, ML con rezagos, Prophet, VECM y los siete canales). |
| `Modelos/` | Seis cuadernos de modelado (01–06), ficheros JSON de documentación metodológica y `verificacion_no_fuga.py`. |
| `Resultados/` | Predicciones por modelo (`preds_*.csv`), métricas (`metricas_*.json`), tablas consolidadas y las proyecciones a 2026 (`proyeccion_futura_*.csv`). |
| `Fuentes/` | Cuadernos preliminares y auxiliares (análisis exploratorio por subperiodos y experimentos de interpolación univariante). |

## Datos y fuentes

Las series proceden de fuentes oficiales primarias:

- **EPA** (parados, ocupados, inactivos y población activa por sexo y edad), **Contabilidad Nacional Trimestral (PIB)**, **IPC**, **IPRI** e **IPI**: Instituto Nacional de Estadística (INE).
- **Tipos de interés** (interés legal del dinero e interés de demora tributaria) y **entidades de depósito** (activo y pasivo): Banco de España.
- **Tipo de cambio** (peseta y euro): Banco de España, para las cotizaciones históricas de la peseta, y Banco Central Europeo, para las del euro.
- **Precio del petróleo** (WTI y Brent): Energy Information Administration (EIA) de Estados Unidos, a través del repositorio FRED de la Reserva Federal de St. Louis.

La variable objetivo es la tasa de paro de la EPA, ambos sexos, total.

## Metodología

El pipeline es secuencial y trazable:

1. **Limpieza y unificación** (`Limpieza y unión de series temporales/` → `Datasets/`): cada serie se trata de forma individual, con su empalme de versiones, su homogeneización de unidades y su paso a frecuencia trimestral.
2. **Análisis exploratorio** (`EDA/`): caracterización de cada serie y diagnóstico de su forma estacionaria, sin tomar decisiones de modelización.
3. **Construcción del panel** (`Datasets Modelado/`): estacionarización de las variables y organización en pools por familia de modelo y por canal económico.
4. **Modelado** (`Modelos/`): paseo aleatorio, ARIMA y SARIMAX, VAR y VECM, Prophet, modelos de aprendizaje automático (Random Forest, XGBoost, Ridge, Lasso), análisis de contribución por canal y causalidad de Granger, y ensamblaje.
5. **Validación**: ventana expansiva (walk-forward) con 116 orígenes de predicción desde 1996Q1 hasta 2024Q4, nowcast y horizontes 1 a 4. Toda la selección de modelos e hiperparámetros se decide solo con información anterior al test; `verificacion_no_fuga.py` comprueba la ausencia de fuga.
6. **Resultados** (`Resultados/`): backtest, métricas, comparación con previsiones oficiales como prueba de estrés y proyección a 2026.

## Reproducibilidad

- **Python 3.13.5** (CPython) sobre Windows.
- Cálculo y datos: `pandas`, `numpy`, `scipy`. Econometría: `statsmodels`. Aprendizaje automático: `scikit-learn`, `xgboost`, `optuna`. Modelo estacional: `prophet`. Interpretabilidad: `shap`. Visualización: `matplotlib`, `seaborn`, `plotly`.
- **Semilla única 42** en todos los componentes con azar (Optuna, Random Forest, XGBoost); los modelos econométricos y los ensamblajes lineales son deterministas.
- Las versiones exactas están fijadas en `requirements.txt`.
- La reproducibilidad de los resultados se sostiene en la fijación de semillas, el esquema de validación por ventana expansiva y la verificación de no fuga.

## Orden de lectura sugerido

`Limpieza y unión de series temporales/` → `Datasets/` → `EDA/` → `Datasets Modelado/` → `Modelos/` (01 a 06) → `Resultados/`.

## Autor

Marcos Fernández Arévalo — Máster Universitario en Business Analytics, Universidad Pontificia Comillas (ICADE).
