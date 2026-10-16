# Informe ejecutivo

**Práctica 2. Análisis Exploratorio de Datos (EDA) y preparación de datasets**
Diego Raymundo Hernández Alonso

## 1. Objetivo

Analizar y preparar dos datasets de clasificación binaria, Titanic y Adult Income, para dejarlos listos para entrenar modelos. Se revisó la calidad de los datos, se exploraron sus patrones y se trataron los valores faltantes, los valores atípicos y el desbalance de clases. Cada decisión se justifica con los datos y con su efecto en un clasificador: una regresión logística entrenada con el 70% de los datos y evaluada con el 30% restante.

## 2. Hallazgos principales

**Titanic**

- El sexo es el factor que más separa a los sobrevivientes: sobrevivió el 74% de las mujeres contra el 19% de los hombres.
- La clase también influye mucho: sobrevivió el 63% de primera clase contra el 24% de tercera. `Pclass` es la variable numérica más correlacionada con `Survived` (-0.34), seguida de `Fare` (0.26).
- La edad casi no se correlaciona con la supervivencia (-0.08), pero la relación no es lineal: los niños de 12 años o menos sobreviven 58% y los mayores de 60 solo 23%.
- `Cabin` tiene 77% de nulos y no se puede recuperar de forma confiable.
- La edad falta en el 27.7% de tercera clase, contra el 13.9% en primera y el 6% en segunda. Además, los pasajeros sin edad sobreviven menos (29.4% contra 40.6%), así que los nulos no son aleatorios.
- `Fare` está muy sesgada (asimetría de 4.79): la mediana es 14.45 y el máximo 512.33.

**Adult Income**

- La variable numérica más relacionada con el ingreso es `education-num` (0.34), seguida de `age` y `hours-per-week` (0.23 cada una).
- La edad tiene forma de U invertida: gana más de 50K el 1.8% de las personas de 17 a 25 años, el 39.8% de 46 a 55 y el 20.1% de los mayores de 65.
- Los datos tienen sesgos marcados: 67% son hombres y el 30.6% de ellos gana más de 50K, contra el 10.9% de las mujeres. `relationship` revela el sexo de forma indirecta.
- Los nulos están solo en variables categóricas y no son aleatorios: las personas sin ocupación registrada ganan más de 50K en el 10.36% de los casos, contra el 24.08% general.
- `capital-gain` y `capital-loss` son casi constantes (91.7% y 95.3% en 0), y el 46.7% de las personas trabaja exactamente 40 horas a la semana.
- `fnlwgt` no aporta información para predecir (correlación de -0.01), por lo que se descartó.

## 3. Decisiones y justificación

### 3.1 Imputación

| Variable | Estrategia A | Estrategia B | F1 del modelo (A / B) | Decisión |
|---|---|---|---|---|
| `Age` (varianza original 211.02) | Media: 169.05 (-19.89%) | Mediana: 169.51 (-19.67%) | 0.743 / 0.735 | **Mediana + `Age_faltante`** |
| `occupation` (Gini original 0.8945) | Moda: 0.8885 | "Desconocido": 0.9029 | 0.667 / 0.668 | **"Desconocido"** |

- En `Age`, las dos estrategias reducen la varianza casi igual porque llenan 177 registros con un solo valor. Ese valor queda en el 22.7% de las filas, una concentración artificial. En el modelo la diferencia equivale a unas 2 predicciones de 268, así que se eligió la mediana por la distribución: conserva el valor central original y es robusta a valores extremos.
- Como la ausencia de la edad es informativa, se agregó el indicador `Age_faltante`. En el modelo casi no cambia el F1 (0.735 a 0.737), porque la mayoría de las edades faltantes son de tercera clase y `Pclass` ya da esa información.
- En `occupation`, la moda (`Prof-specialty`) pasaría de 13.48% a 18.37% y metería a personas que en su mayoría ganan `<=50K` en la ocupación con más ingresos altos (44.9% gana `>50K`). La categoría "Desconocido" conserva las proporciones del resto y funciona como indicador de valor ausente. El mismo criterio se aplicó a `workclass` y `native-country`.

### 3.2 Winsorización

| Variable | Método | Límites | Valores recortados | Desv. estándar | F1 del modelo |
|---|---|---|---|---|---|
| `Fare` | Original | 0 a 512.33 | - | 49.69 | 0.737 |
| | IQR | 0 a 65.63 | 116 | 20.48 | 0.723 |
| | Percentiles 1-99 | 0 a 249.01 | 9 | 42.52 | 0.737 |
| `hours-per-week` | Original | 1 a 99 | - | 12.35 | 0.668 |
| | IQR | 32.5 a 52.5 | 9,008 | 6.19 | 0.670 |
| | Percentiles 1-99 | 8 a 80 | 503 | 11.94 | 0.670 |

Se eligieron los **percentiles 1-99**. En `Fare`, el IQR deja con el mismo precio a 116 pasajeros, 104 de ellos de primera clase (casi la mitad de esa clase), y baja el F1 del modelo. En `hours-per-week`, recorta al 27.7% de los registros y manda a todas las personas de medio tiempo a 32.5 horas, un valor que no existía en los datos. Los percentiles solo acotan los casos más extremos, conservan la forma de la distribución y no empeoran el modelo.

### 3.3 Balanceo

Las dos razones de desbalance superan 1.5 (Titanic 1.61, Adult 3.15). Se compararon tres técnicas contra la línea base: sobremuestreo aleatorio, submuestreo aleatorio y SMOTE. En los tres casos el balanceo se aplicó solo al conjunto de entrenamiento, dentro de un pipeline, para que el conjunto de prueba conservara su distribución real.

## 4. Resultados del balanceo

Las métricas de precision, recall y F1 corresponden a la clase minoritaria.

**Titanic**

| Métrica | Sin balanceo | Sobremuestreo | Submuestreo | SMOTE |
|---|---|---|---|---|
| Accuracy | 0.806 | 0.795 | 0.817 | 0.810 |
| Precision | 0.768 | 0.703 | 0.750 | 0.728 |
| Recall | 0.709 | 0.806 | 0.786 | 0.806 |
| F1 | 0.737 | 0.751 | 0.768 | 0.765 |

**Adult Income**

| Métrica | Sin balanceo | Sobremuestreo | Submuestreo | SMOTE |
|---|---|---|---|---|
| Accuracy | 0.854 | 0.807 | 0.810 | 0.810 |
| Precision | 0.738 | 0.566 | 0.570 | 0.570 |
| Recall | 0.613 | 0.857 | 0.856 | 0.855 |
| F1 | 0.670 | 0.682 | 0.684 | 0.684 |

- En Adult, el modelo sin balanceo solo detecta al 61.3% de las personas con ingreso alto. Con cualquiera de las tres técnicas detecta cerca del 85.6%. A cambio, la precisión baja y aparecen más falsos positivos. El F1 sube en todos los casos.
- Las tres técnicas dan casi el mismo resultado. En Titanic, la diferencia de recall entre el submuestreo y las otras dos son solo dos pasajeros.
- Lo que cambia es el costo: el submuestreo elimina 11,814 filas de la clase mayoritaria en Adult (68% de esa clase) y SMOTE genera casos artificiales, con valores intermedios en columnas que solo deberían valer 0 o 1.

## 5. Conclusiones

1. Los valores faltantes no siempre son aleatorios. En los dos datasets, que falte un dato está relacionado con la variable objetivo, así que la forma de imputar puede meter sesgos si no se revisa.
2. Imputar con un solo valor reduce la varianza y crea concentraciones artificiales. Cuando el modelo no distingue entre estrategias, la decisión se debe tomar por lo que pasa con los datos.
3. La regla del IQR no sirve para todas las variables. Cuando los datos se concentran en un solo valor, como las 40 horas semanales, marca como atípicos a casi un tercio de los registros.
4. En datos desbalanceados, la accuracy no es suficiente. En Adult, un modelo que siempre dijera `<=50K` tendría 75.9% de accuracy sin detectar a nadie de la clase minoritaria.
5. La correlación solo detecta relaciones lineales. La edad parece no importar en Titanic, pero por rangos sí cambia mucho la supervivencia.

## 6. Recomendación

- Usar los datasets con el tratamiento descrito: mediana más `Age_faltante` en Titanic, "Desconocido" en las categóricas de Adult y winsorización por percentiles 1-99.
- Balancear con **sobremuestreo aleatorio** solo en el entrenamiento. Da prácticamente el mismo resultado que las otras técnicas, no pierde registros y no inventa casos.
- Evaluar los modelos con precision, recall y F1 de la clase minoritaria, no solo con accuracy.
- Probar la edad por rangos o un modelo basado en árboles para aprovechar las relaciones no lineales.
- Antes de usar un modelo de Adult para tomar decisiones sobre personas, revisar su desempeño por sexo y por raza.

## 7. Limitaciones

- La mediana y la moda se calcularon con todo el dataset, antes de separar entrenamiento y prueba. El efecto es mínimo (la mediana de la edad sería 29 en vez de 28).
- No se trataron los 24 registros duplicados de Adult, porque en un censo pueden ser personas distintas con el mismo perfil.
- `education` se mantuvo junto con `education-num` aunque son redundantes.
- Los efectos se midieron con un solo modelo y una sola separación de los datos. En Titanic, el conjunto de prueba es pequeño (268 pasajeros) y varias diferencias son de una o dos personas.
- Los datos de Adult son de 1994, así que sus conclusiones no se pueden aplicar directamente a la población actual.
