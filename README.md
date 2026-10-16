# Práctica 2. Análisis Exploratorio de Datos (EDA) y preparación de datasets

**Alumno:** Diego Raymundo Hernández Alonso
**Materia:** Inteligencia Artificial, ESCOM-IPN

En esta práctica hice el análisis exploratorio y la preparación de dos datasets de clasificación binaria: **Titanic** y **Adult Income**. El trabajo cubre la comprensión del dominio, el perfilado de los datos, el EDA visual, la imputación de valores perdidos, el tratamiento de outliers con winsorización y el balanceo de clases. El efecto de cada decisión se mide con una regresión logística.

## Datasets

| Dataset | Fuente | Licencia | Registros | Variables | Objetivo |
|---|---|---|---|---|---|
| Titanic | Kaggle, *Titanic - Machine Learning from Disaster* (`train.csv`) | Reglas de la competencia de Kaggle | 891 | 12 | `Survived` (0 / 1) |
| Adult Income | UCI Machine Learning Repository, *Adult* (`adult.data`) | CC BY 4.0 | 32,561 | 15 | `income` (`<=50K` / `>50K`) |

El archivo de UCI no trae encabezados. En `datos/adult.csv` se agregaron los nombres de columna oficiales y se quitó el espacio al inicio de cada valor. Los valores faltantes se dejaron como `?`, igual que en el original.

## Estructura del repositorio

```
HernandezAlonso_Pract2_IA/
├── Practica2_EDA.ipynb      Notebook con todo el desarrollo
├── datos/
│   ├── titanic.csv
│   └── adult.csv
├── graficas/                Gráficas generadas por el notebook (PNG)
├── docs/
│   ├── ficha_tecnica.md     Ficha técnica de los datasets
│   └── informe_ejecutivo.md Hallazgos, decisiones, resultados y recomendación
├── requirements.txt
└── README.md
```

## Cómo ejecutarlo

Se necesita Python 3.11 o superior.

1. Clonar el repositorio y entrar a la carpeta:

   ```
   git clone https://github.com/DiegoRHA030427/HernandezAlonso_Pract2_IA.git
   cd HernandezAlonso_Pract2_IA
   ```

2. Instalar las dependencias:

   ```
   pip install -r requirements.txt
   ```

3. Abrir el notebook desde esa misma carpeta:

   ```
   jupyter notebook Practica2_EDA.ipynb
   ```

4. Ejecutar todas las celdas en orden (*Run All*). Las rutas son relativas a la carpeta del repositorio, así que no hay que modificar nada. Las gráficas se guardan en `graficas/`.

## Contenido del notebook

| Paso | Contenido |
|---|---|
| 2. Dominio y data profiling | Fuente, licencia, unidad de análisis, tipos de dato, nulos, estadísticos, percentiles, asimetría, columnas casi constantes, categorías inconsistentes, identificadores, fugas y sesgos |
| 3. EDA | Distribuciones, boxplots, variables categóricas, distribución de clases, matriz de correlación, relación atributo-objetivo y relaciones no lineales |
| 4. Imputación | `Age`: media vs mediana e indicador `Age_faltante`. `occupation`: moda vs "Desconocido". Comparación de varianza, concentración y desempeño del modelo |
| 5. Winsorización | `Fare` y `hours-per-week`: IQR vs percentiles 1-99, con estadísticos, gráficas y desempeño del modelo |
| 6. Desbalance | Conteo, porcentaje y razón de desbalance (IR). Comparación de sobremuestreo, submuestreo y SMOTE |
| Conclusiones | Conclusiones y recomendación final |

## Resultados principales

- **Imputación:** en `Age`, la media y la mediana redujeron la varianza casi igual (-19.89% y -19.67%) y el modelo casi no las distingue. Se eligió la mediana y se agregó `Age_faltante`, porque los pasajeros sin edad sobreviven menos (29.4% contra 40.6%). En `occupation` se eligió "Desconocido", porque los registros sin ocupación tienen un ingreso alto mucho menos frecuente (10.36%) que la moda `Prof-specialty` (44.9%).
- **Winsorización:** el IQR recortó 116 valores de `Fare` y 9,008 de `hours-per-week`, y en Titanic bajó el F1 de 0.737 a 0.723. Los percentiles 1-99 recortaron 9 y 503 valores y no empeoraron el modelo. Se aplicaron percentiles.
- **Balanceo:** IR de 1.61 en Titanic y 3.15 en Adult. En Adult, cualquiera de las tres técnicas sube el recall de la clase minoritaria de 0.613 a cerca de 0.856. Se recomienda el sobremuestreo aleatorio, porque da el mismo resultado sin perder registros ni crear casos artificiales.

El detalle está en `docs/informe_ejecutivo.md`.
