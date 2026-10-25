# Glosario de variables — Loan_Default.csv

34 columnas, 148.670 filas, una fila = una solicitud de préstamo hipotecario. Fuente:
[Kaggle — Loan Default Dataset (yasserh)](https://www.kaggle.com/datasets/yasserh/loan-default-dataset).

Las descripciones salen del **diccionario de datos oficial** de la ficha de Kaggle. Quedan
marcadas las pocas columnas que el diccionario oficial tampoco explica en detalle: los valores
concretos que toman `loan_type`, `loan_purpose` y `Credit_Worthiness`.

| Columna | Tipo | Qué es (según el diccionario oficial) | Notas |
|---|---|---|---|
| `ID` | numérica | Identificador de la solicitud. | No es predictiva. |
| `year` | numérica | Año de la solicitud. | Constante = 2019 en todo el dataset. |
| `loan_limit` | categórica | Si el préstamo es conforme (`cf`) o no conforme (`ncf`) — conforming loan limit de EE.UU. | 2,25% nulo. |
| `Gender` | categórica | Género del solicitante (male, female, joint, "sex not available"). | Variable demográfica sensible: no entra al modelo, se usa para auditar (sección 7.3 del notebook). |
| `approv_in_adv` | categórica | Si el préstamo fue aprobado por adelantado (`pre`) o no (`nopre`). | 0,61% nulo. |
| `loan_type` | categórica (type1/2/3) | Tipo de préstamo. | El diccionario oficial no especifica qué producto es cada tipo. |
| `loan_purpose` | categórica (p1-p4) | Propósito del préstamo. | El diccionario oficial no especifica qué motivo es cada código. |
| `Credit_Worthiness` | categórica (l1/l2) | Solvencia crediticia del solicitante. | El diccionario oficial no especifica qué distingue l1 de l2. |
| `open_credit` | categórica | Si el solicitante tiene cuentas de crédito abiertas (`opc`) o no (`nopc`). | Muy desbalanceada (556 vs 148.114). |
| `business_or_commercial` | categórica | Si el préstamo es para uso de negocio/comercial (`ob/c`) o personal (`nob/c`). | Se refiere al uso del inmueble, no a que sea un crédito sin garantía. |
| `loan_amount` | numérica (USD) | Monto del préstamo solicitado. | Variable clave. |
| `rate_of_interest` | numérica (%) | Tasa de interés cobrada sobre el préstamo. | **Excluida del modelo: nula en ~99% de Status=1** (artefacto de calidad de datos, ver más abajo). |
| `Interest_rate_spread` | numérica | Diferencia entre la tasa del préstamo y una tasa de referencia (benchmark). | **Excluida del modelo: nula en 100% de Status=1.** |
| `Upfront_charges` | numérica (USD) | Cargos iniciales asociados a la formalización del préstamo. | **Excluida del modelo: nula en ~99,6% de Status=1.** |
| `term` | numérica (meses) | Duración del préstamo. | Predominantemente 360 (30 años). |
| `Neg_ammortization` | categórica | Si el préstamo permite amortización negativa. | |
| `interest_only` | categórica | Si el préstamo tiene una opción de pago solo-interés. | |
| `lump_sum_payment` | categórica | Si requiere un pago único (balloon payment) al final del plazo. | |
| `property_value` | numérica (USD) | Valor de la propiedad que se financia. | 10,16% nulo; **41,2% nulo cuando Status=1** (ver secciones 1.2 y 1.3 del notebook). |
| `construction_type` | categórica | Tipo de construcción: `sb` = site built (construcción tradicional), `mh` = manufactured home (vivienda prefabricada). | Muy desbalanceada. |
| `occupancy_type` | categórica | Tipo de ocupación: `pr` = residencia principal, `sr` = residencia secundaria, `ir` = propiedad de inversión. | |
| `Secured_by` | categórica | Tipo de garantía que respalda el préstamo: `home` o `land`. | Confirma que es un dataset hipotecario. |
| `total_units` | categórica (1U-4U) | Cantidad de unidades en la propiedad financiada. | |
| `income` | numérica (USD/mes) | Ingreso del solicitante. El diccionario no aclara el período, pero **es mensual**: la mediana es USD 5.760 para préstamos de USD 296.500, y la deuda/ingreso mediana (38%) solo cierra con ingreso mensual. | 6,15% nulo; 1.260 filas en 0 (348 fuera del bloque artefacto). |
| `credit_type` | categórica | Buró de crédito del solicitante: `CIB` = credit information bureau, `CRIF` = CRIF credit information bureau, `EXP` = Experian, `EQUI` = Equifax. | **`EQUI` tiene 100% de mora** y coincide con las filas sin tasación: es el bloque artefacto que la entrega final excluye. |
| `Credit_Score` | numérica | Puntaje crediticio del solicitante. | Escala 500-900 en este dataset (no la escala FICO estándar 300-850). |
| `co-applicant_credit_type` | categórica (CIB/EXP) | Buró de crédito del co-solicitante. | |
| `age` | categórica (bins) | Edad del solicitante. | 0,13% nulo. |
| `submission_of_application` | categórica | Cómo se envió la solicitud: `to_inst` = a la institución, `not_inst` = no a la institución. | 0,13% nulo. |
| `LTV` | numérica (%) | Loan-to-Value = `loan_amount / property_value * 100`. | **Identidad exacta**, verificada al 100%. |
| `Region` | categórica | Región geográfica donde está la propiedad (North, south, central, North-East). | |
| `Security_Type` | categórica | Tipo de garantía: directa o indirecta. | El dataset trae "Indriect" (typo de "Indirect"); muy desbalanceada. |
| `Status` | numérica (0/1) | **Variable objetivo.** Indica si el préstamo entró en mora (`1`) o no (`0`). | Es lo que el modelo predice. |
| `dtir1` | numérica (%) | Debt-to-Income ratio total. | 16,22% nulo; 44,5% nulo cuando Status=1. Tope observado en 61%, sugiere truncamiento. |

## El target: `Status`

`Status` es la variable que el modelo predice. El diccionario oficial la define textualmente
como *"indicates whether the loan has been defaulted (1) or not (0)"*: mora en sentido
estricto.

## Las tres columnas excluidas del modelo

`rate_of_interest`, `Interest_rate_spread` y `Upfront_charges` están nulas en ~99-100% de las
filas con `Status = 1` y casi nunca en las demás. Es decir: **la simple ausencia del dato
delata el target**, así que no se pueden usar como variables predictoras — un modelo que las
incluyera aprendería a detectar celdas vacías en vez de evaluar riesgo crediticio.

El diccionario oficial no explica por qué faltan justamente ahí, si son condiciones que se
fijan en la originación, antes de que el default pueda ocurrir. La hipótesis más razonable es
un **artefacto del proceso de recolección de los datos** (por ejemplo, que el sistema de
servicing haya dejado de actualizar esos campos una vez que el crédito entró en mora). La
decisión de excluirlas no depende de resolver esa incógnita.

La evidencia completa está en la sección 1.2 del notebook (tabla de valores faltantes).

## El bloque artefacto (hallazgo de la entrega final)

15.333 filas (10,3%) tienen 100% de mora: las de `credit_type = EQUI` y las de
`property_value` vacío (son casi las mismas: 15.096 en común), más 200 con `age` vacío y 33 con
`Secured_by = land`. Explican el 42% de todas las moras. No son un segmento real de clientes
sino un artefacto de cómo se armó el dataset, y la entrega final las excluye del modelado
(ver secciones 1.3 y 2.1 del notebook).
