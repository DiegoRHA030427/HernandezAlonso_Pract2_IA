# Informe ejecutivo

**Práctica 2. Análisis Exploratorio de Datos (EDA) y preparación de datasets**
Diego Raymundo Hernández Alonso

## 1. Objetivo

Analizar y preparar dos datasets de clasificación binaria, Titanic y Adult Income, para dejarlos listos para entrenar modelos. Se revisó la calidad de los datos, se exploraron sus patrones y se trataron los valores faltantes, los valores atípicos y el desbalance de clases. Cada decisión se justifica con los datos.

## 2. Hallazgos principales

**Titanic**

- El sexo es el factor que más separa a los sobrevivientes: sobrevivió el 74% de las mujeres contra el 19% de los hombres.
- La clase también influye mucho: sobrevivió el 63% de primera clase contra el 24% de tercera. `Pclass` es la variable numérica más correlacionada con `Survived` (-0.34), seguida de `Fare` (0.26).
- `Cabin` tiene 77% de nulos y no se puede recuperar de forma confiable.
- A la tercera clase le falta la edad en el 27.7% de los casos, contra el 13.9% en primera y el 6% en segunda, así que los nulos no son aleatorios.
- `Fare` está muy sesgada: la mediana es 14.45 y el máximo es 512.33.

**Adult Income**

- La variable más relacionada con el ingreso es `education-num` (0.34), seguida de `age` y `hours-per-week` (0.23 cada una).
- Los datos tienen sesgos marcados: 67% son hombres y el 30.6% de ellos gana más de 50K, contra el 10.9% de las mujeres. Un modelo entrenado sin cuidado podría reproducir esta desigualdad.
- Los nulos están solo en variables categóricas y no son aleatorios: las personas sin ocupación registrada ganan más de 50K en el 10.36% de los casos, contra el 24.08% general.
- `fnlwgt` no aporta información para predecir (correlación de -0.01), por lo que se descartó.
- El 46.7% de las personas trabaja exactamente 40 horas a la semana, lo que vuelve muy angosto el rango intercuartílico de `hours-per-week`.

## 3. Decisiones y justificación

### 3.1 Imputación

| Variable | Estrategia A | Estrategia B | Decisión |
|---|---|---|---|
| `Age` (varianza original 211.02) | Media: 169.05 (-19.89%) | Mediana: 169.51 (-19.67%) | **Mediana** |
| `occupation` (Gini original 0.8945) | Moda: 0.8885 | "Desconocido": 0.9029 | **"Desconocido"** |

- En `Age`, las dos estrategias reducen la varianza casi igual, porque llenan 177 registros con un solo valor. Se eligió la mediana porque conserva el valor central original, pierde un poco menos de varianza y es robusta a valores extremos.
- En `occupation`, la moda (`Prof-specialty`) pasaría de 13.48% a 18.37%. Además, metería a personas que en su mayoría ganan `<=50K` en la ocupación con más ingresos altos (44.9% gana `>50K`). La categoría "Desconocido" conserva las proporciones del resto de las ocupaciones y mantiene la información de que el dato faltaba. El mismo criterio se aplicó a `workclass` y `native-country`.

### 3.2 Winsorización

| Variable | Método | Límites | Valores recortados | Desv. estándar |
|---|---|---|---|---|
| `Fare` | Original | 0 a 512.33 | - | 49.69 |
| | IQR | 0 a 65.63 | 116 | 20.48 |
| | Percentiles 1-99 | 0 a 249.01 | 9 | 42.52 |
| `hours-per-week` | Original | 1 a 99 | - | 12.35 |
| | IQR | 32.5 a 52.5 | 9,008 | 6.19 |
| | Percentiles 1-99 | 8 a 80 | 503 | 11.94 |

Se eligieron los **percentiles 1-99**. En `Fare`, el IQR deja con el mismo precio a 116 pasajeros, 104 de ellos de primera clase (casi la mitad de esa clase). En `hours-per-week`, recorta al 27.7% de los registros y manda a todas las personas de medio tiempo a 32.5 horas, un valor que no existía en los datos. Los percentiles solo acotan los casos más extremos y conservan la forma de la distribución.

### 3.3 Balanceo

Las dos razones de desbalance superan 1.5 (Titanic 1.61, Adult 3.15). Se usó **sobremuestreo aleatorio** de la clase minoritaria porque no elimina registros y es el método más simple. Los datos se separaron en entrenamiento (70%) y prueba (30%) antes de balancear, y solo se balanceó el entrenamiento, para que las filas duplicadas no aparecieran también en el conjunto de prueba.

## 4. Resultados del balanceo

Se entrenó una regresión logística con y sin balanceo y se evaluó sobre el mismo conjunto de prueba. Las métricas corresponden a la clase minoritaria.

| Métrica | Titanic sin | Titanic con | Adult sin | Adult con |
|---|---|---|---|---|
| Accuracy | 0.802 | 0.799 | 0.854 | 0.807 |
| Precision | 0.745 | 0.717 | 0.738 | 0.565 |
| Recall | 0.738 | 0.786 | 0.613 | 0.856 |
| F1 | 0.741 | 0.750 | 0.670 | 0.681 |

- En Titanic el efecto es pequeño, porque el desbalance era leve.
- En Adult, el modelo sin balanceo solo detectaba al 61.3% de las personas con ingreso alto. Con sobremuestreo detecta al 85.6%. A cambio, la precisión baja y aparecen más falsos positivos. El F1 sube en ambos casos.

## 5. Conclusiones

1. Los valores faltantes no siempre son aleatorios. En los dos datasets, que falte un dato está relacionado con la variable objetivo, así que la forma de imputar puede meter sesgos si no se revisa.
2. Imputar con un solo valor siempre reduce la varianza. Con 20% de nulos, la varianza de `Age` bajó cerca de 20% sin importar si se usó la media o la mediana.
3. La regla del IQR no sirve para todas las variables. Cuando los datos se concentran en un solo valor, como las 40 horas semanales, el IQR marca como atípicos a casi un tercio de los registros. Los percentiles dieron un resultado más razonable.
4. En datos desbalanceados, la accuracy no es suficiente. En Adult, un modelo que siempre dijera `<=50K` tendría 75.9% de accuracy sin detectar a nadie de la clase minoritaria. Balancear mejoró el recall a costa de la precisión.

## 6. Limitaciones

- No se trataron los 24 registros duplicados de Adult, porque en un censo pueden ser personas distintas con el mismo perfil.
- `education` se mantuvo junto con `education-num` aunque son redundantes.
- La regresión logística se usó solo para medir el efecto del balanceo, no como modelo final.
