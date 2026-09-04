# TP4 — Taller de Programación | UBA

## Clasificación de informalidad laboral en la EPH: regularización y árboles CART

**Maestría en Economía Aplicada — Universidad de Buenos Aires**
**Grupo 2**

### Integrantes

* Andrea Chasi
* Santiago Soler
* Pablo Ortiz

---

## 1. Objetivo

Este Trabajo Práctico continúa el análisis desarrollado en el **TP3** y estudia la capacidad de distintos métodos de clasificación para predecir la condición de informalidad laboral utilizando información de la **Encuesta Permanente de Hogares (EPH)**.

El análisis combina información de **2024 y 2025** y aprovecha la dimensión longitudinal de la base, incorporando la situación de informalidad observada en 2024 como predictor de la condición laboral en 2025.

Se estiman y comparan cuatro modelos:

1. **Regresión Logística sin penalización**
2. **Regresión Logística LASSO (penalización L1)**
3. **Regresión Logística Ridge (penalización L2)**
4. **Árbol de clasificación CART**

La selección de los hiperparámetros de regularización y complejidad se realiza mediante **validación cruzada de 5 folds**.

---

## 2. Datos y estrategia de modelado

La muestra final utilizada para la estimación contiene **7.188 trabajadores ocupados**.

La variable objetivo presenta la siguiente distribución:

* **6.743 trabajadores formales (93,81%)**
* **445 trabajadores informales (6,19%)**

La muestra presenta, por lo tanto, un **marcado desbalance de clases**, aspecto que resulta central para la interpretación de las métricas predictivas.

El conjunto de variables explicativas contiene **25 predictores**, que incluyen características:

* demográficas;
* educativas;
* laborales;
* del hogar;
* de cobertura de salud;
* geográficas;
* y la condición de informalidad observada en 2024.

Siguiendo las indicaciones metodológicas del curso, se excluyen como predictores:

* salario e ingresos;
* tamaño del establecimiento;
* y variables utilizadas directamente para construir la definición de informalidad.

Esta decisión busca evitar problemas de **filtración de información (*data leakage*)** y asegurar que la clasificación se base únicamente en información admisible para el ejercicio predictivo.

---

## 3. Modelos estimados

### Regresión Logística

Se utiliza como modelo de referencia una **Regresión Logística sin penalización**, que permite estimar directamente la relación entre las características observadas y la probabilidad de informalidad en 2025.

### LASSO

La regresión **LASSO (Least Absolute Shrinkage and Selection Operator)** incorpora una penalización L1 que puede reducir algunos coeficientes exactamente a cero y, por lo tanto, realizar selección automática de variables.

La validación cruzada selecciona:

**λ = 100**

Con este nivel de penalización, LASSO reduce **22 de los 25 coeficientes a cero** y conserva únicamente:

* `informal_2024`
* `tiene_cobertura_salud`
* `edad`

El resultado evidencia una fuerte concentración de la señal predictiva en un conjunto reducido de variables.

### Ridge

La regresión **Ridge** utiliza una penalización L2 que reduce la magnitud de los coeficientes, aunque sin eliminarlos necesariamente del modelo.

La validación cruzada selecciona igualmente:

**λ = 100**

### CART

Se estima además un **Classification and Regression Tree (CART)**.

La complejidad del árbol se selecciona mediante validación cruzada. El parámetro óptimo según este criterio produce un **árbol completamente podado**, es decir, sin divisiones adicionales respecto del nodo raíz.

Como análisis complementario, se examina también el mejor árbol no degenerado. En esta especificación, `informal_2024` aparece nuevamente como una de las variables de mayor relevancia.

---

## 4. Principales resultados

Uno de los resultados más consistentes del ejercicio es la importancia de la **persistencia temporal de la informalidad laboral**.

La variable:

```text
informal_2024
```

concentra una parte importante del poder predictivo de los modelos y resulta especialmente relevante tanto en **LASSO** como en **CART**.

Esto indica que la condición laboral observada en el período anterior aporta información significativa para anticipar la situación laboral del trabajador en el período siguiente.

---

## 5. Comparación del desempeño predictivo

| Modelo                 |   Accuracy | 1 - Accuracy |     Recall |   F1-score |        AUC |
| ---------------------- | ---------: | -----------: | ---------: | ---------: | ---------: |
| Logit sin penalización |     0.9378 |       0.0622 | **0.1236** | **0.1975** | **0.8015** |
| LASSO                  | **0.9381** |   **0.0619** |     0.0000 |     0.0000 |     0.7944 |
| Ridge                  | **0.9381** |   **0.0619** |     0.0000 |     0.0000 |     0.7702 |
| CART                   | **0.9381** |   **0.0619** |     0.0000 |     0.0000 |     0.5000 |

Utilizando un umbral de clasificación de:

```text
p = 0.5
```

el **Logit sin penalización** es el único modelo que logra identificar trabajadores pertenecientes a la clase informal.

LASSO, Ridge y CART alcanzan una **accuracy ligeramente superior**, pero lo hacen clasificando todas las observaciones como pertenecientes a la clase mayoritaria.

Este resultado evidencia una limitación importante de la **accuracy** cuando existe un fuerte desbalance entre clases: un modelo puede presentar una elevada proporción global de observaciones correctamente clasificadas y, simultáneamente, no detectar ningún caso de la categoría de mayor interés.

Por esta razón, la evaluación debe considerar conjuntamente métricas como:

* **Recall**
* **F1-score**
* **Área bajo la curva ROC (AUC)**
* **Accuracy**

---

## 6. Interpretación de los resultados

Bajo las reglas de evaluación utilizadas en el trabajo, el **Logit sin penalización presenta el mejor compromiso predictivo**.

Aunque su accuracy es marginalmente inferior a la de los modelos regularizados, presenta:

* el mayor **AUC (0.8015)**;
* el mayor **Recall (0.1236)**;
* el mayor **F1-score (0.1975)**;
* y es el único modelo capaz de identificar verdaderos positivos utilizando un umbral de `p = 0.5`.

Los resultados muestran además que una regularización elevada puede mejorar la estabilidad o simplificación del modelo, pero no necesariamente su capacidad para detectar observaciones pertenecientes a una clase minoritaria cuando se utiliza un umbral de clasificación fijo.

---

## 7. Implicancia para política pública

Desde una perspectiva de política pública, una institución interesada en identificar trabajadores en condiciones laborales vulnerables enfrenta costos diferentes frente a distintos tipos de error.

Por ejemplo, para una **Secretaría de Trabajo**, el costo de no detectar a un trabajador informal —un **falso negativo**— puede ser mayor que el costo de incorporar temporalmente dentro de una población objetivo a un trabajador formal —un **falso positivo**—.

En este contexto, la selección de un modelo no debería basarse exclusivamente en la accuracy global.

El ejercicio muestra que el **Logit sin penalización** proporciona una mejor capacidad de discriminación entre trabajadores formales e informales, mientras que los modelos regularizados y CART tienden, con el umbral utilizado, a reproducir la categoría mayoritaria.

Asimismo, la relevancia de `informal_2024` sugiere una importante **persistencia intertemporal de la informalidad**, lo que puede resultar relevante para el diseño y focalización de políticas laborales.

---

## 8. Estructura del repositorio

```text
TP4_Programacion_UBA/
│
├── README.md
│
├── TP4_Programacion_Grupo2.ipynb
│
├── bases/
│   ├── TP4_Base_Modelado_2025_MM.csv
│   └── TP4_Base_Modelado_2025_MM.xlsx
│
├── figuras/
│   └── Figuras de regularización, CART y evaluación predictiva
│
└── tablas/
    └── Resultados de validación cruzada, coeficientes y métricas
```

---

## 9. Herramientas utilizadas

El análisis fue desarrollado principalmente en **Python**, utilizando herramientas para:

* manipulación y procesamiento de datos;
* estimación de modelos de clasificación;
* regularización LASSO y Ridge;
* construcción de árboles CART;
* validación cruzada;
* cálculo de métricas de desempeño;
* y visualización de resultados.

El desarrollo y ejecución del trabajo se realizó en un entorno de **Jupyter Notebook / Google Colab**.

---

## 10. Reproducibilidad

El notebook principal:

```text
TP4_Programacion_Grupo2.ipynb
```

contiene el flujo completo del trabajo, incluyendo:

1. carga y preparación de las bases;
2. construcción de la muestra de modelado;
3. definición de variables;
4. preparación de los predictores;
5. estimación del Logit;
6. validación cruzada para LASSO y Ridge;
7. estimación de CART;
8. evaluación fuera de muestra;
9. matrices de confusión;
10. curvas ROC;
11. comparación de métricas;
12. exportación de tablas y figuras.

De esta manera, los principales resultados reportados pueden reproducirse directamente a partir del notebook y de las bases incluidas en el repositorio.
