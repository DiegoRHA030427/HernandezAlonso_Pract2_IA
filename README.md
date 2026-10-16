# Práctica 2. Análisis Exploratorio de Datos (EDA) y preparación de datasets

**Alumno:** Diego Raymundo Hernández Alonso
**Materia:** Inteligencia Artificial, ESCOM-IPN

En esta práctica hice el análisis exploratorio y la preparación de dos datasets de clasificación binaria: **Titanic** y **Adult Income**. El trabajo cubre el perfilado de los datos, el EDA visual, la imputación de valores perdidos, el tratamiento de outliers con winsorización y el balanceo de clases.

## Datasets

| Dataset | Fuente | Registros | Variables | Objetivo |
|---|---|---|---|---|
| Titanic | Kaggle, *Titanic - Machine Learning from Disaster* (`train.csv`) | 891 | 12 | `Survived` (0 / 1) |
| Adult Income | UCI Machine Learning Repository, *Adult* (`adult.data`) | 32,561 | 15 | `income` (`<=50K` / `>50K`) |

El archivo de UCI no trae encabezados. En `datos/adult.csv` se agregaron los nombres de columna oficiales y se quitaron los espacios al inicio de cada valor. Los valores faltantes se dejaron como `?`, igual que en el original.

## Estructura del repositorio

```
HernandezAlonso_Pract2_IA/
├── Practica2_EDA.ipynb      Notebook con todo el desarrollo (pasos 2 a 6)
├── datos/
│   ├── titanic.csv
│   └── adult.csv
├── graficas/                Gráficas generadas por el notebook (PNG)
├── docs/
│   ├── ficha_tecnica.md     Ficha técnica de los datasets
│   └── informe_ejecutivo.md Hallazgos, decisiones y resultados
├── requirements.txt
└── README.md
```

## Cómo ejecutarlo

1. Instalar las dependencias:

   ```
   pip install -r requirements.txt
   ```

2. Abrir el notebook:

   ```
   jupyter notebook Practica2_EDA.ipynb
   ```

3. La variable `ruta` de la primera celda apunta a `C:\Inteligencia Artificial ESCOM\Pract2_InteligenciaArtificial`. Si el repositorio se clona en otra ubicación, hay que cambiarla por la carpeta correspondiente.

4. Ejecutar todas las celdas en orden (*Run All*). Las gráficas se guardan en `graficas/`.

## Contenido del notebook

| Paso | Contenido |
|---|---|
| 2. Data profiling | Tipos de dato, valores válidos, nulos, estadísticos, variable objetivo, identificadores, fugas y sesgos |
| 3. EDA | Distribuciones, boxplots, variables categóricas, distribución de clases, matriz de correlación y relación atributo-objetivo |
| 4. Imputación | `Age`: media vs mediana. `occupation`: moda vs "Desconocido". Comparación de varianza |
| 5. Winsorización | `Fare` y `hours-per-week`: IQR vs percentiles 1-99 |
| 6. Desbalance | Razón de desbalance (IR), sobremuestreo aleatorio y comparación con regresión logística |

## Resultados principales

- **Imputación:** en `Age`, la media y la mediana redujeron la varianza casi igual (-19.89% y -19.67%). Se eligió la mediana. En `occupation` se eligió "Desconocido", porque los registros sin ocupación tienen un ingreso alto mucho menos frecuente (10.36%) que la moda `Prof-specialty` (44.9%).
- **Winsorización:** el IQR recortó 116 valores de `Fare` y 9,008 de `hours-per-week`. Los percentiles 1-99 recortaron 9 y 503, respectivamente. Se aplicaron percentiles, porque el IQR generaba picos artificiales.
- **Balanceo:** IR de 1.61 en Titanic y 3.15 en Adult. Con sobremuestreo, el recall de la clase minoritaria en Adult subió de 0.613 a 0.856 y el F1 de 0.670 a 0.681.

El detalle está en `docs/informe_ejecutivo.md`.
