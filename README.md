# Predicción de mora en créditos hipotecarios · Grupo 24

Proyecto Final Integrador (Unidad 10) de la Diplomatura en Ciencia de Datos y Análisis Avanzado, UTN BA.

**Integrantes:** Ailin Soriano, Fernanda Pardo, Pablo Magnino, Germán Langoni.

El notebook estima la probabilidad de que una solicitud de crédito hipotecario entre en mora y la usa para decidir si se aprueba o se rechaza. El resultado económico se compara contra reglas simples basadas en LTV y deuda/ingreso.

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/pabloamagnino-coder/grupo24-utn-tpfinal/blob/main/repo/Prediccion_mora_hipotecaria.ipynb)

El informe final y la presentación están en la carpeta `docs/`. El notebook tiene solo el código y sus resultados: la explicación de cada sección está en el informe.

---

## Resultados principales (test: 33.335 solicitudes)

| | Mejor regla LTV/DTI | Modelo XGBoost |
|---|---|---|
| Moras detectadas, rechazando como máximo 20% | 31,3% | **54,6%** |
| Pérdida crediticia (simulación) | USD 126,2 M | **USD 71,3 M** |
| Resultado neto de la cartera (simulación) | USD 122,9 M | **USD 178,9 M** |
| PR-AUC (azar = 0,16) | — | **0,621** |

Las cifras en dólares son una simulación con margen de 3% y recupero de la garantía de 70%.

### Cómo se eligió el modelo

En la validación cruzada, HistGradientBoosting y XGBoost quedan empatados (PR-AUC 0,618 los dos). La regla fijada antes de modelar desempata por calibración (Brier) y ahí la diferencia es mínima: 0,0921 para XGBoost contra 0,0922 para HistGradientBoosting. Una diferencia tan chica puede invertirse con otra versión de scikit-learn, por eso fijamos las versiones con las que se obtuvieron los resultados: **scikit-learn 1.8.0 y xgboost 3.2.0**. En Colab, la primera celda del notebook las instala. Si se ejecuta con otras versiones y el desempate se invierte, el notebook lo avisa y mantiene XGBoost, el modelo del informe.

---

## Estructura

```
.
├── README.md
├── docs/
│   ├── Informe final - Grupo 24.pdf
│   └── Presentacion final - Grupo 24.pdf
└── repo/
    ├── Prediccion_mora_hipotecaria.ipynb   notebook ejecutado de punta a punta
    ├── README.md                           este mismo archivo
    ├── requirements.txt
    ├── data/
    │   ├── Loan_Default.zip                dataset (el CSV comprimido)
    │   └── glosario_variables.md           descripción de las 34 columnas
    └── outputs/final/                      se regeneran al ejecutar el notebook
        ├── figures/                        gráficos
        └── tables/                         tablas con los resultados
```

---

## Cómo ejecutarlo

**En Google Colab**

1. Abrir el notebook con el botón **Open in Colab** de arriba.
2. `Entorno de ejecución` → `Ejecutar todas`.
3. La primera celda instala scikit-learn 1.8.0 y xgboost 3.2.0 (y shap, si falta). Si pide reiniciar la sesión: `Entorno de ejecución` → `Reiniciar sesión` y volver a `Ejecutar todas`.

El dataset se descarga solo desde este repositorio.

**En la computadora** (Python 3.11 o posterior)

```bash
git clone https://github.com/pabloamagnino-coder/grupo24-utn-tpfinal.git
cd grupo24-utn-tpfinal/repo
python -m venv .venv
# Windows: .venv\Scripts\activate    Linux/Mac: source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook Prediccion_mora_hipotecaria.ipynb
```

La ejecución completa tarda unos 10-15 minutos. Las semillas y las versiones son fijas, así que los resultados se reproducen.

---

## Qué hace el notebook

Sigue CRISP-DM. Las tablas y los gráficos se guardan en `outputs/final/`.

| Sección | Contenido | Salidas |
|---|---|---|
| 1. Comprensión de los datos | Variables, valores faltantes, un bloque de 15.333 solicitudes con 100% de mora, EDA de las hipótesis H1-H4 | t01-t02, f01-f02 |
| 2. Preparación de los datos | Filas excluidas, variables nuevas (`lti`, `ingreso_cero`), partición 75/25 estratificada | t03-t04 |
| 3. Modelado | Supuestos económicos, cuatro algoritmos con validación cruzada de 5 folds, regla de selección, hiperparámetros, calibración, modelo final con 5 semillas | t05, f03-f04 |
| 4. Evaluación | Comparación contra reglas LTV/DTI con un tope de 20% de rechazo, a igual tasa de rechazo, intervalos bootstrap, métricas de clasificación | t06-t11, f05-f06 |
| 5. Qué aporta cada variable | Ablaciones, curva de aprendizaje, efecto del bloque excluido | t12-t14, f07-f08 |
| 6. Interpretación (SHAP) | Importancia global de las variables | t15, f09 |
| 7. Impacto, riesgos y ética | Sensibilidad a los supuestos, simulación a tres años, auditoría por género, edad y región | t16-t18, f13-f15 |

---

## Dataset

[Loan Default Dataset](https://www.kaggle.com/datasets/yasserh/loan-default-dataset) (Kaggle, usuario yasserh): 148.670 solicitudes de 2019, 34 columnas, sin datos personales identificables.

Está incluido en `repo/data/Loan_Default.zip`, comprimido porque el CSV pesa 28 MB. No hace falta descomprimirlo: el notebook lo lee directamente y, si no lo encuentra (por ejemplo, en Colab), lo descarga de este repositorio. La descripción de cada columna está en `repo/data/glosario_variables.md`.
