# Trabajo Data Streams - Prediccion de lluvia y clustering online

Este proyecto implementa la actividad evaluable de **Data Streams** del Master en Investigacion en Inteligencia Artificial.  
Se trabaja con el dataset **weatherAUS** para resolver un problema de clasificacion binaria (*RainTomorrow*) y una parte de clustering online.

## Objetivo de la actividad

Segun el enunciado, se desarrollan 4 ejercicios:

1. Dos modelos de clasificacion online con evaluacion **holdout 70/30**.
2. Los mismos dos modelos con evaluacion **prequential** (interleaved test-then-train) y comparacion.
3. Deteccion de deriva de concepto en stream.
4. Dos modelos de clustering online y analisis de sus soluciones.

## Dataset

- **Fuente**: weatherAUS (Kaggle).
- **Tamano original**: 145,460 filas y 23 columnas.
- **Target**: `RainTomorrow` (No/Yes, convertido a 0/1).
- **Desbalanceo inicial**: aprox. 77.6% clase 0 y 22.4% clase 1.

### Limpieza inicial

- Se eliminan columnas con alto porcentaje de nulos: `Evaporation`, `Sunshine`, `Cloud9am`, `Cloud3pm`.
- Se eliminan filas con nulos en `RainTomorrow` y `RainToday`.
- Se ordena temporalmente por `Date`.

## Estructura del proyecto

```text
TrabajoDataStreams/
├── data/
│   └── weatherAUS.csv
├── modelos_stream.ipynb
├── Trabajo_Data_Streams.pdf
└── README.md
```

## Metodologia implementada

### 1) Clasificacion online (holdout 70/30)

Pipeline de preprocesamiento online con River:

- Imputacion dinamica (`StatImputer` con `Mean`/`Mode`).
- Estandarizacion para variables numericas (`StandardScaler`).
- Codificacion basada en target para variables categoricas relevantes (`TargetAgg` + `BayesianMean`).
- Balanceo de clases en flujo con `RandomUnderSampler` (dist. deseada 50/50).

Modelos:

- **Modelo 1**: `HoeffdingAdaptiveTreeClassifier`.
- **Modelo 2**: `ARFClassifier` (Adaptive Random Forest).

Metricas usadas:

- Accuracy, Balanced Accuracy, Cohen's Kappa, F1, ROC AUC, matriz de confusion y reporte por clase.

### 2) Clasificacion online (prequential)

Se repiten los mismos modelos y preprocesamiento, con estrategia:

1. Se predice en cada instancia.
2. Se actualizan metricas.
3. Se aprende con esa misma instancia.

Esto permite adaptacion continua al stream y comparacion directa con holdout.

### 3) Deteccion de deriva

- Se usa un clasificador base tipo Hoeffding adaptativo.
- Detector de deriva: **ADWIN** (`delta=1e-5`) sobre el error online.
- Se monitoriza accuracy en ventana movil para visualizar cambios y puntos de deriva detectados.

### 4) Clustering online

Sobre variables numericas preprocesadas online (imputacion + escalado):

- **Modelo cluster 1**: `cluster.KMeans` (online).
- **Modelo cluster 2**: `cluster.CluStream`.

Evaluacion externa mediante distribucion de etiquetas reales (`RainTomorrow`) dentro de cada cluster.

## Resultados principales observados

### Holdout (test 30%)

- **HoeffdingAdaptiveTree**: Accuracy **0.6048**, ROC AUC **0.7141**, Kappa **0.2143**.
- **Adaptive Random Forest**: Accuracy **0.7389**, ROC AUC **0.6907**, Kappa **0.2924**.

### Prequential

- **HoeffdingAdaptiveTree**: Accuracy **0.7057**, ROC AUC **0.7130**, Kappa **0.2885**.
- **Adaptive Random Forest**: Accuracy **0.7166**, ROC AUC **0.7239**, Kappa **0.3000**.

### Conclusiones del notebook

- El enfoque prequential muestra comportamiento mas estable en el tiempo.
- El entrenamiento holdout puede quedarse obsoleto antes en streams largos.
- ADWIN detecta multiples puntos de cambio, coherentes con no estacionariedad en datos meteorologicos.
- En clustering, KMeans y CluStream producen particiones distintas, con porcentajes de lluvia por cluster relativamente cercanos.

## Requisitos

El notebook utiliza principalmente:

- `python`
- `pandas`, `numpy`
- `river`
- `matplotlib`, `seaborn`
- `tqdm`

Instalacion sugerida:

```bash
pip install pandas numpy river matplotlib seaborn tqdm
```

## Ejecucion

1. Colocar el dataset en `data/weatherAUS.csv`.
2. Abrir `modelos_stream.ipynb`.
3. Ejecutar celdas en orden para reproducir:
   - preparacion del stream,
   - evaluacion holdout,
   - evaluacion prequential,
   - deteccion de deriva,
   - clustering online.
