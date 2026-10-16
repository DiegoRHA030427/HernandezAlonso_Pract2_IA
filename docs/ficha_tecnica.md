# Ficha técnica de los datasets

**Práctica 2. EDA y preparación de datasets** · Diego Raymundo Hernández Alonso

## 1. Titanic

| Campo | Descripción |
|---|---|
| Fuente | Kaggle, *Titanic - Machine Learning from Disaster*, archivo `train.csv` |
| Dominio | Pasajeros del RMS Titanic, hundido en abril de 1912 |
| Tarea | Clasificación binaria: predecir si un pasajero sobrevivió |
| Tamaño | 891 registros, 12 variables, sin filas duplicadas |
| Variable objetivo | `Survived`: 0 = no sobrevivió (549, 61.6%), 1 = sobrevivió (342, 38.4%) |
| Razón de desbalance | IR = 549 / 342 = 1.61 |
| Variables numéricas | `Age` (0.42 a 80), `Fare` (0 a 512.33), `SibSp` (0 a 8), `Parch` (0 a 6) |
| Variables categóricas | `Pclass` (1, 2, 3), `Sex` (male, female), `Embarked` (C, Q, S) |
| Identificadores | `PassengerId` y `Name` (únicos por fila), `Ticket` (681 valores en 891 filas) |
| Valores faltantes | `Cabin` 687 (77.1%), `Age` 177 (19.9%), `Embarked` 2 (0.2%) |
| Valores atípicos | `Fare` muy sesgada a la derecha (asimetría de 4.79), con 116 valores fuera del rango IQR. Hay 15 boletos con precio 0 |
| Fugas de información | No se identificaron variables registradas después del hundimiento |
| Sesgos | Mayoría de hombres (577 contra 314 mujeres) y de tercera clase (491). Los nulos de `Age` se concentran en tercera clase (27.7% contra 13.9% en primera y 6% en segunda) |

**Tratamiento aplicado**

- Se eliminaron `PassengerId`, `Name`, `Ticket` y `Cabin`.
- `Age` se imputó con la mediana (28) y `Embarked` con la moda (S).
- `Fare` se winsorizó con los percentiles 1 y 99 (límite superior de 249.01, 9 valores recortados).
- Para el modelo de prueba, la clase minoritaria se balanceó con sobremuestreo aleatorio solo en el conjunto de entrenamiento.

**Versión final:** 891 registros, 8 variables (`Survived`, `Pclass`, `Sex`, `Age`, `SibSp`, `Parch`, `Fare`, `Embarked`), sin nulos.

## 2. Adult Income

| Campo | Descripción |
|---|---|
| Fuente | UCI Machine Learning Repository, *Adult*, archivo `adult.data` (conjunto de entrenamiento) |
| Dominio | Censo de Estados Unidos de 1994 |
| Tarea | Clasificación binaria: predecir si una persona gana más de 50 mil dólares al año |
| Tamaño | 32,561 registros, 15 variables, 24 filas duplicadas |
| Variable objetivo | `income`: `<=50K` (24,720, 75.9%), `>50K` (7,841, 24.1%) |
| Razón de desbalance | IR = 24,720 / 7,841 = 3.15 |
| Variables numéricas | `age` (17 a 90), `fnlwgt`, `education-num` (1 a 16), `capital-gain` (0 a 99,999), `capital-loss` (0 a 4,356), `hours-per-week` (1 a 99) |
| Variables categóricas | `workclass` (8), `education` (16), `marital-status` (7), `occupation` (14), `relationship` (6), `race` (5), `sex` (2), `native-country` (41) |
| Identificadores | Ninguno. `fnlwgt` es un peso de muestreo del censo y no describe a la persona (correlación con el objetivo de -0.01) |
| Valores faltantes | Marcados como `?`: `occupation` 1,843 (5.7%), `workclass` 1,836 (5.6%), `native-country` 583 (1.8%) |
| Valores atípicos | `capital-gain` tiene 91.7% de ceros y 159 registros con 99,999, un posible valor tope. En `hours-per-week`, el 46.7% vale exactamente 40, por lo que la regla IQR marca 9,008 registros como atípicos |
| Redundancia | `education` y `education-num` contienen la misma información. `relationship` se relaciona con `sex` y `marital-status` |
| Fugas de información | No se identificaron variables derivadas del ingreso |
| Sesgos | 66.9% hombres y 85.4% de raza blanca. El 30.6% de los hombres gana más de 50K contra el 10.9% de las mujeres. Por raza, el 25.6% de las personas blancas contra el 12.4% de las personas negras |

**Tratamiento aplicado**

- Se eliminó `fnlwgt`.
- `workclass`, `occupation` y `native-country` se imputaron con la categoría "Desconocido".
- `hours-per-week` se winsorizó con los percentiles 1 y 99 (límites de 8 y 80, 503 valores recortados).
- Para el modelo de prueba, la clase `>50K` se balanceó con sobremuestreo aleatorio solo en el conjunto de entrenamiento.

**Versión final:** 32,561 registros, 14 variables, sin nulos.
