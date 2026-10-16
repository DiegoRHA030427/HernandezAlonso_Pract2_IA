# Ficha técnica de los datasets

**Práctica 2. EDA y preparación de datasets** · Diego Raymundo Hernández Alonso

## 1. Titanic

| Campo | Descripción |
|---|---|
| Fuente | Kaggle, competencia *Titanic - Machine Learning from Disaster*, archivo `train.csv`. Los datos vienen de listas históricas de pasajeros |
| Licencia | Sin licencia Creative Commons. Su uso se rige por las reglas de la competencia de Kaggle. Aquí se usa con fines académicos |
| Unidad de análisis | Un pasajero del RMS Titanic, hundido en abril de 1912 |
| Tarea | Clasificación binaria: predecir si un pasajero sobrevivió |
| Dimensiones | 891 filas, 12 columnas, sin filas duplicadas |
| Variable objetivo | `Survived`: 0 = no sobrevivió (549, 61.6%), 1 = sobrevivió (342, 38.4%). IR = 1.61 |
| Variables numéricas | `Age` (0.42 a 80), `Fare` (0 a 512.33), `SibSp` (0 a 8), `Parch` (0 a 6) |
| Variables categóricas | `Pclass` (1, 2, 3), `Sex` (male, female), `Embarked` (C, Q, S) |
| Identificadores | `PassengerId` y `Name` (únicos por fila), `Ticket` (681 valores en 891 filas) |
| Problemas de calidad | Nulos en `Cabin` (687, 77.1%), `Age` (177, 19.9%) y `Embarked` (2). `Fare` muy sesgada (asimetría de 4.79, 116 atípicos por IQR) y con 15 boletos en 0. 18 edades son estimadas (terminan en .5) |
| Riesgos | Los nulos de `Age` no son aleatorios: faltan en el 27.7% de tercera clase contra el 6% de segunda. Hay mayoría de hombres (577 contra 314) y de tercera clase (491). Son datos de personas reales con nombre |

**Tratamiento aplicado**

- Se eliminaron `PassengerId`, `Name`, `Ticket` y `Cabin`.
- Se creó `Age_faltante` (1 si la edad venía vacía). Después, `Age` se imputó con la mediana (28) y `Embarked` con la moda (S).
- `Fare` se winsorizó con los percentiles 1 y 99 (límite superior de 249.01, 9 valores recortados).
- Para el modelo, la clase minoritaria se balanceó con sobremuestreo aleatorio solo en el conjunto de entrenamiento.

**Versión final:** 891 registros y 9 variables (`Survived`, `Pclass`, `Sex`, `Age`, `SibSp`, `Parch`, `Fare`, `Embarked`, `Age_faltante`), sin nulos.

## 2. Adult Income

| Campo | Descripción |
|---|---|
| Fuente | UCI Machine Learning Repository, *Adult* (Becker y Kohavi, 1996, DOI 10.24432/C5XW20), archivo `adult.data` |
| Licencia | Creative Commons Attribution 4.0 (CC BY 4.0) |
| Unidad de análisis | Una persona mayor de 16 años registrada en el censo de Estados Unidos de 1994 |
| Tarea | Clasificación binaria: predecir si una persona gana más de 50 mil dólares al año |
| Dimensiones | 32,561 filas, 15 columnas, 24 filas duplicadas |
| Variable objetivo | `income`: `<=50K` (24,720, 75.9%), `>50K` (7,841, 24.1%). IR = 3.15 |
| Variables numéricas | `age` (17 a 90), `fnlwgt`, `education-num` (1 a 16), `capital-gain` (0 a 99,999), `capital-loss` (0 a 4,356), `hours-per-week` (1 a 99) |
| Variables categóricas | `workclass` (8), `education` (16), `marital-status` (7), `occupation` (14), `relationship` (6), `race` (5), `sex` (2), `native-country` (41) |
| Identificadores | Ninguno. `fnlwgt` es un peso de muestreo del censo, no describe a la persona |
| Problemas de calidad | Nulos marcados como `?` en `occupation` (1,843, 5.7%), `workclass` (1,836, 5.6%) y `native-country` (583, 1.8%). `capital-gain` y `capital-loss` son casi constantes (91.7% y 95.3% en 0) y `capital-gain` tiene 159 valores en 99,999, un posible tope. El 46.7% trabaja exactamente 40 horas. `education` y `education-num` son redundantes. Hay categorías con muy pocos casos, como `Never-worked` (7) |
| Riesgos | 66.9% hombres y 85.4% de raza blanca. Gana más de 50K el 30.6% de los hombres contra el 10.9% de las mujeres. `relationship` revela el sexo de forma indirecta. Los datos son de 1994 y el umbral de 50K no está ajustado por inflación |

**Tratamiento aplicado**

- Se eliminó `fnlwgt`.
- `workclass`, `occupation` y `native-country` se imputaron con la categoría "Desconocido".
- `hours-per-week` se winsorizó con los percentiles 1 y 99 (límites de 8 y 80, 503 valores recortados).
- Para el modelo, la clase `>50K` se balanceó con sobremuestreo aleatorio solo en el conjunto de entrenamiento.

**Versión final:** 32,561 registros y 14 variables, sin nulos.

## 3. Limitaciones

- La mediana y la moda se calcularon con todo el dataset, antes de separar entrenamiento y prueba. Con solo el entrenamiento, la mediana de la edad sería 29 en vez de 28.
- No se quitaron los 24 duplicados de Adult, porque en un censo pueden ser personas distintas con el mismo perfil.
- `education` se mantuvo junto con `education-num` aunque contienen la misma información.
- El conjunto de prueba de Titanic tiene solo 268 pasajeros, así que algunas diferencias entre métodos son de una o dos personas.
- Todos los efectos se midieron con un solo modelo (regresión logística) y una sola separación de los datos.
