# TP4 — Taller de Programación | UBA

## Clasificación de informalidad en la EPH: Regularización y CART

**Maestría en Economía Aplicada — Universidad de Buenos Aires**  
**Grupo 2**

**Integrantes**
- Andrea Chasi
- Santiago Soler
- Pablo Ortiz

---

## Objetivo

Este Trabajo Práctico continúa el análisis desarrollado en el TP3 y estudia la capacidad de distintos métodos de clasificación para identificar trabajadores informales en la Encuesta Permanente de Hogares (EPH).

El análisis utiliza información de 2024 y 2025 y explota la dimensión longitudinal incorporando el estatus de informalidad observado en 2024 como predictor de la situación laboral en 2025.

Se comparan cuatro modelos:

1. Regresión Logística sin penalización
2. LASSO (L1)
3. Ridge (L2)
4. Árbol de decisión CART

La selección de los parámetros de regularización y de complejidad se realiza mediante validación cruzada de 5 folds.

---

## Datos y estrategia

La muestra final de modelado contiene **7.188 ocupados**, de los cuales:

- **6.743 (93,81%)** son clasificados como formales.
- **445 (6,19%)** corresponden a la categoría de informalidad analizada por el Grupo 2.

El modelo utiliza **25 predictores**, incluyendo características demográficas, educativas, del hogar, cobertura de salud, región y el estatus de informalidad observado en 2024.

De acuerdo con las indicaciones metodológicas del curso, se excluyen de los predictores el salario/ingreso, el tamaño del establecimiento y las variables utilizadas directamente en la construcción de la definición de informalidad.

---

## Principales resultados

La validación cruzada selecciona **λ = 100** para LASSO y Ridge.

LASSO reduce **22 de los 25 coeficientes a cero**, conservando únicamente:

- `informal_2024`
- `tiene_cobertura_salud`
- `edad`

La informalidad observada en 2024 constituye la señal predictiva más persistente de la informalidad en 2025.

En CART, el parámetro seleccionado mediante validación cruzada genera un árbol completamente podado. Un análisis complementario con el mejor árbol no degenerado confirma nuevamente la importancia dominante de `informal_2024`.

---

## Comparación predictiva

| Modelo | Accuracy | 1-Accuracy | Recall | F1 | AUC |
|---|---:|---:|---:|---:|---:|
| Logit sin penalización | 0.9378 | 0.0622 | **0.1236** | **0.1975** | **0.8015** |
| LASSO | **0.9381** | **0.0619** | 0.0000 | 0.0000 | 0.7944 |
| Ridge | **0.9381** | **0.0619** | 0.0000 | 0.0000 | 0.7702 |
| CART | **0.9381** | **0.0619** | 0.0000 | 0.0000 | 0.5000 |

Con un umbral de clasificación de **p = 0,5**, el Logit sin penalización es el único modelo que identifica trabajadores informales.

El resultado muestra por qué, frente a una clase minoritaria, la accuracy no debe utilizarse aisladamente como criterio de selección: LASSO, Ridge y CART alcanzan una accuracy ligeramente superior clasificando todas las observaciones como formales.

---

## Implicancia de política pública

Para una Secretaría de Trabajo interesada en identificar grupos vulnerables y asignar recursos escasos, el costo de no detectar a un trabajador informal puede ser sustancialmente mayor que el costo de incorporar un falso positivo.

Bajo las reglas de evaluación del TP, el **Logit sin penalización presenta el mejor compromiso predictivo**, al obtener el mayor AUC y ser el único modelo que identifica verdaderos positivos con `p = 0,5`.

La evidencia también confirma la importancia de la **persistencia temporal de la informalidad**, dado que `informal_2024` constituye uno de los predictores más relevantes tanto en LASSO como en CART.

---

## Estructura del repositorio

```text
TP4_Programacion_UBA/
│
├── README.md
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
