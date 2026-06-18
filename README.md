# 📊 SalesPredictor-AI

Sistema analítico de predicción de demanda y gestión de inventario para retail, desarrollado como proyecto de especialidad en Ingeniería en Data Science (AIEP, 4° semestre).

> **Contexto**: Los datos y el escenario de negocio corresponden a Dodo Supermercado, una empresa ficticia diseñada con fines analíticos y de demostración de capacidades.

---

## 🎯 Problema de negocio

Dodo Supermercado opera con **12 tiendas distribuidas a lo largo de Chile** y enfrenta dos riesgos operacionales críticos:

- **Quiebre de stock** en categorías de alta rotación → pérdida de ventas y clientes.
- **Sobrestock** en categorías de baja demanda → capital inmovilizado y mermas.

El objetivo fue anticipar ambos escenarios mediante un modelo predictivo entrenado con datos históricos reales (2023–2024) y proyectado hacia 2025.

---

## 🔍 Enfoque analítico y decisión de modelo

La propuesta inicial consideraba **ARIMA** y **LSTM**. Durante el desarrollo se identificó que ARIMA generaba proyecciones excesivamente planas al no capturar factores comerciales externos.

Se evolucionó hacia **SARIMAX**, que permite:

- Incorporar **estacionalidad semanal** (ciclo = 7), coherente con el comportamiento típico de supermercados (alza en fines de semana, baja a inicio de semana).
- Integrar **variables exógenas** como promociones, descuentos y variación de precios.

**Parámetros del modelo:**

- `order = (1, 1, 1)`
- `seasonal_order = (1, 0, 1, 7)`

---

## 🗂️ Flujo del proyecto

```
data/ (CSV raw)
    ↓
01_etl_build_db.py   →   SQLite (ventas_dodo.db)
    ↓
02_eda_queries.ipynb →   Análisis exploratorio (SQL + Python)
    ↓
03_sarimax_model.ipynb → Modelado predictivo + proyección 2025
    ↓
outputs/             →   Gráficos y resultados
```

---

## 📁 Estructura del repositorio

```
SalesPredictor-AI/
├── data/                        # Archivos CSV originales (fuente cruda)
├── database/
│   ├── create_table.sql         # Creación de tablas
│   └── schema.sql               # Esquema documentado de la base de datos
├── scripts/
│   ├── 01_etl_build_db.py       # ETL: extracción, transformación y carga a SQLite
│   ├── 02_eda_queries.ipynb     # Análisis exploratorio con SQL y visualizaciones
│   └── 03_sarimax_model.ipynb   # Modelado SARIMAX y proyección 2025
├── outputs/                     # Gráficos generados por el análisis
├── Proyecto_SalesPredictorAI.pbix  # Dashboard Power BI conectado a SQLite
├── requirements.txt
├── .gitignore
└── README.md
```

---

## ⚙️ Proceso ETL

A partir de **3 archivos CSV** con datos de ventas históricas (fechas, cantidades, precios, promociones, descuentos y categorías), se aplicó el siguiente proceso:

- **Extracción**: consolidación de información de las 12 tiendas de la cadena.
- **Transformación**: normalización de texto, conversión de tipos de datos, eliminación de nulos, duplicados y registros inconsistentes.
- **Carga**: almacenamiento en base de datos **SQLite3** para consultas SQL eficientes y análisis exploratorio.

---

## 🔎 Análisis exploratorio

Con consultas SQL sobre la base de datos se identificaron los dos casos más representativos del riesgo operacional:

| Foco                              | Ciudad          | Categoría      | Unidades vendidas (2023–2024) |
| --------------------------------- | --------------- | -------------- | ----------------------------- |
| Mayor demanda (riesgo quiebre)    | **Arica**       | **Bebidas**    | 174.171                       |
| Menor demanda (riesgo sobrestock) | **Antofagasta** | **Congelados** | 125.934                       |

**Estadísticas descriptivas previas al modelado (731 registros totales):**

| Métrica                  | Arica — Bebidas | Antofagasta — Congelados |
| ------------------------ | --------------- | ------------------------ |
| Promedio diario          | 238,05 ud       | 172,08 ud                |
| Desviación estándar      | 46,81           | 35,20                    |
| Coeficiente de variación | 19,66%          | 20,45%                   |

---

## 📈 Resultados del modelo SARIMAX (Test 2024)

El modelo fue entrenado con datos de **2023** y evaluado con datos reales de **2024** (datos no vistos durante el entrenamiento):

| Ciudad — Categoría       | RMSE  | MAE   | MAPE      |
| ------------------------ | ----- | ----- | --------- |
| Arica — Bebidas          | 55,90 | 48,78 | **21,3%** |
| Antofagasta — Congelados | 55,01 | 49,89 | **31,1%** |

> Los errores son considerados adecuados para el entorno retail, donde promociones, descuentos y variación de precios generan fluctuaciones significativas en la demanda. El MAPE más alto en Antofagasta se explica por la mayor irregularidad de la categoría congelados.

---

## 🔮 Proyección 2025 vs real 2024

Las variables exógenas de la proyección se construyeron con ruido controlado basado en patrones históricos:

- **Precio**: incremento leve de ~0,96% según variación histórica observada.
- **Promoción**: aumentada de 43,3% (histórico) a 44,7%, con mayor probabilidad en fines de semana.
- **Descuento**: mantenido en torno al promedio histórico de 5,33%.

| Caso                     | Unidades 2024 (real) | Unidades 2025 (proyección) | Promedio diario 2024 | Promedio diario 2025 |
| ------------------------ | -------------------- | -------------------------- | -------------------- | -------------------- |
| Arica — Bebidas          | 87.495               | **119.033**                | 239 ud/día           | **326 ud/día**       |
| Antofagasta — Congelados | 62.941               | **81.149**                 | 172 ud/día           | **222 ud/día**       |

**Hallazgos clave:**

- **Arica – Bebidas**: riesgo alto de quiebre de stock entre octubre y noviembre 2025, con picos proyectados de 580–600 unidades diarias.
- **Antofagasta – Congelados**: riesgo de sobrestock en febrero, junio y julio; oportunidades de venta en septiembre–octubre.

---

## 🛠️ Tecnologías utilizadas

| Categoría             | Herramientas                  |
| --------------------- | ----------------------------- |
| Lenguaje              | Python 3                      |
| Manipulación de datos | Pandas, NumPy                 |
| Modelado predictivo   | Statsmodels (SARIMAX)         |
| Base de datos         | SQLite3                       |
| Visualización Python  | Matplotlib                    |
| Visualización BI      | Power BI (conectado a SQLite) |
| Entorno               | Jupyter Notebook              |
| Control de versiones  | Git / GitHub                  |

---

## 🚀 Cómo ejecutar el proyecto

**1. Instalar dependencias:**

```bash
pip install -r requirements.txt
```

**2. Ejecutar el ETL (genera la base de datos SQLite):**

```bash
py scripts/01_etl_build_db.py
```

**3. Análisis exploratorio:**
Abrir `scripts/02_eda_queries.ipynb` en Jupyter Notebook.

**4. Modelado y proyección:**
Abrir `scripts/03_sarimax_model.ipynb` en Jupyter Notebook.

**5. Dashboard Power BI:**
Abrir `Proyecto_SalesPredictorAI.pbix` (requiere Power BI Desktop).

---

## 🔭 Trabajo futuro

- Ajuste de hiperparámetros con búsqueda automática (grid search).
- Incorporación de nuevas variables exógenas (estacionalidad climática, feriados).
- Exploración de modelos alternativos: LSTM, Prophet, enfoques híbridos.
- Automatización del flujo completo ETL → modelado → visualización para ejecución periódica.
- Escalamiento hacia las 12 tiendas y todas las categorías disponibles.

---

## 👥 Autores

**Joel Eduardo Bravo Bravo**
Técnico Analista de Datos | Ingeniería en Data Science (en curso) — AIEP
[LinkedIn](https://www.linkedin.com/in/joelbravob) · [GitHub](https://github.com/joelbravob93)

**Marieth Randolph** · **Kareanyelis Chirino**

> Proyecto desarrollado en equipo como trabajo de especialidad —
> Ingeniería en Data Science, AIEP (4° semestre).
