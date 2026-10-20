# Predicción de mora en créditos hipotecarios · Grupo 24

Entrega final del Proyecto Final Integrador (Unidad 9) de la Diplomatura en Ciencia de Datos y Análisis Avanzado, UTN BA.

**Integrantes:** Ailin Soriano, Fernanda Pardo, Pablo Magnino, Germán Langoni.

El notebook estima la probabilidad de que una solicitud de crédito hipotecario entre en mora y la usa para decidir si se aprueba o se rechaza, comparando el resultado económico contra una regla simple basada en LTV y deuda/ingreso.

---

## Dataset

[Loan Default Dataset](https://www.kaggle.com/datasets/yasserh/loan-default-dataset) (Kaggle, usuario yasserh): 148.670 solicitudes de 2019, 34 columnas.

El dataset está incluido en `data/Loan_Default.zip` (comprimido, porque el CSV pesa 28 MB). No hace falta descomprimirlo ni descargarlo aparte: el notebook lo lee directamente y, si no lo encuentra (por ejemplo, al abrirlo en Colab), lo descarga de este repositorio.

---

## Estructura

```
├── Prediccion_mora_hipotecaria.ipynb   notebook completo, ya ejecutado
├── data/
│   ├── Loan_Default.zip                dataset (CSV comprimido)
│   └── glosario_variables.md           descripción de las 34 columnas
├── outputs/                            se generan al ejecutar
│   ├── figures/                        gráficos
│   └── tables/                         tablas con los resultados
└── requirements.txt
```

---

## Cómo ejecutarlo

**En Google Colab**

1. Abrir [el notebook en Colab](https://colab.research.google.com/github/pabloamagnino-coder/grupo24-utn-tpfinal/blob/main/Prediccion_mora_hipotecaria.ipynb).
2. Si falta alguna librería, ejecutar `!pip install shap xgboost` en una celda nueva (el resto ya viene en Colab).
3. `Entorno de ejecución` → `Ejecutar todas`. El dataset se descarga solo desde el repositorio.

**En la computadora**

```bash
git clone https://github.com/pabloamagnino-coder/grupo24-utn-tpfinal.git
cd grupo24-utn-tpfinal
pip install -r requirements.txt
jupyter notebook Prediccion_mora_hipotecaria.ipynb
```

Tarda entre 5 y 15 minutos según la máquina. Las semillas son fijas, así que los resultados se reproducen.

---

## Qué hace el notebook

1. **Negocio:** la decisión de aprobar o rechazar, el costo de cada error y los objetivos.
2. **Datos y EDA:** calidad, faltantes y un bloque de 15.333 solicitudes con 100% de mora que se excluye.
3. **Preparación:** filas y columnas excluidas, imputación, outliers y variables nuevas.
4. **Modelado:** regresión logística, Random Forest, HistGradientBoosting y XGBoost, con una regla de selección fijada antes de ver los resultados.
5. **Evaluación:** comparación contra reglas simples con LTV y deuda/ingreso, con un tope de 20% de rechazo e intervalos bootstrap.
6. **Ablaciones y curva de aprendizaje.**
7. **Interpretación con SHAP:** importancia global y explicación de un rechazo.
8. **Impacto, riesgos y ética:** sensibilidad, simulación a tres años con CAPEX, OPEX y valor actual neto, y auditoría por género, edad y región.
9. **Conclusiones.**

---

## Resultados principales (test: 33.335 solicitudes)

| | Mejor regla LTV/DTI | Modelo XGBoost |
|---|---|---|
| Moras detectadas, rechazando como máximo 20% | 31,3% | **54,6%** |
| Resultado neto de la cartera (simulación) | USD 122,9 M | **USD 178,9 M** |
| PR-AUC (azar = 0,16) | — | **0,621** |

Las cifras en dólares son una simulación con margen de 3% y recupero de la garantía de 70%.
