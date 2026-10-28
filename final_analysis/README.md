# Servicios para adultos mayores en la CDMX — proyección a 2026, 2027, 2029 y 2031

Proyección de la población de 60 y más y de los asilos (SCIAN 623311 y 623312) a nivel AGEB para la
Ciudad de México, a partir del Censo 2010, la Encuesta Intercensal 2015 y el Censo 2020, cruzados con
el DENUE.

## Por dónde empezar

| Archivo | Para quién | Qué contiene |
|---|---|---|
| **`analisis_resumen_v3.ipynb`** | Quien necesita el resultado | El camino mínimo de datos a conclusiones. 25 celdas. |
| **`analisis_completo_v3.ipynb`** | Quien necesita defender el resultado | Lo mismo más la evidencia de cada decisión metodológica. 38 celdas. |
| `analisis_agebs_cdmx_v3.ipynb` | Referencia | Notebook de trabajo: todo lo que se probó y se descartó. |

Las dos versiones producen los **mismos números**. La corta no esconde nada; simplemente cita los
diagnósticos en vez de ejecutarlos.

Ambas siguen la misma estructura:

1. **EDA** — qué hay en los datos y cómo se distribuye en el territorio.
2. **Proyección de población** — el insumo de todo lo demás.
3. **¿Cuántos asilos?** — modelo de conteo sobre los seis cortes del DENUE, proyección a 2026, 2027,
   2029 y 2031.
4. **¿Dónde?** — el desajuste territorial entre oferta y demanda, sin supuestos de distancia.
5. **Conclusiones y limitaciones.**

## Los resultados en una tabla

| Pregunta | Respuesta | Confianza |
|---|---|---|
| ¿Cuánta población de 60+ habrá en 2031? | 2.1-2.4 millones (+43 a +58%) | **Alta** — backtest R² 0.94, MAPE 10% |
| ¿Cuántos asilos habrá? | 112 en 2026, 114 en 2027, 119 en 2029 y **124 en 2031** [95, 157], contra 109 hoy | **Media** — la dirección sí, la magnitud no |
| ¿Cuántos en un AGEB específico? | No es respondible con estos datos | **Nula** — se documenta por qué |
| ¿Dónde está el desajuste? | 752 mil personas 60+ nuevas sobre 108 asilos: 6.6x de diferencia entre alcaldías | **Alta** — es aritmética, sin supuestos de distancia |

**El hallazgo de fondo:** la elasticidad oferta-población es **0.51**, no 1. La oferta crece a poco más
de la mitad del ritmo de la demanda demográfica, y se ubica donde *estuvo* la demanda y no donde va a
estar. La brecha de 2031 no es un problema de cantidad, es de territorio.

## Cómo correrlo

Python **3.11 o 3.12**. Con 3.13 y 3.14, PyMC y PyTensor todavía fallan.

```bash
python3.11 -m venv .venv
source .venv/bin/activate          # en Windows: .venv\Scripts\activate
pip install -r requirements.txt
python -m ipykernel install --user --name inegi --display-name "Python (inegi)"
```

En VS Code: abrir la carpeta, abrir el notebook, y en **Select Kernel** elegir `Python (inegi)`.

Si PyMC no compila —lo habitual en macOS con Apple Silicon— **no hace falta arreglarlo**: el notebook
detecta el fallo y cae solo a una aproximación variacional en Python puro, con los mismos resultados
cualitativos. Si se quiere el muestreo completo con NUTS:

```bash
rm -rf ~/.pytensor
xcode-select --install
export SDKROOT=$(xcrun --show-sdk-path)
pip install nutpie        # alternativa: muestreador en Rust, sin compilar C++
```

## Entradas y salidas

**Entradas** (deben estar junto al notebook):

- `datos/results2010.csv`, `results2015.csv`, `results2020.csv` — población de 60+ y conteo de
  establecimientos por AGEB. El corte de 2015 viene de la Encuesta Intercensal, así que su columna se
  llama `p_60ymas_est` y es estimación de muestra, con error de muestreo propio.
- `datos/datos_denue_2018.csv`, `_2023.csv`, `_2025.csv` — cortes adicionales del DENUE (solo oferta).
  Con ellos la serie de asilos llega a seis puntos: 78 → 79 → 87 → 108 → 108 → 109.
- `geo/2020_1_09_A.shp` — marco geoestadístico 2020, AGEB urbana CDMX (INEGI).

**Salidas** que generan los notebooks:

- `desajuste_por_alcaldia.csv` — **la salida operativa**: por alcaldía, población de 60+ en 2020 y
  2031, asilos, oferta por cada diez mil, demanda nueva por establecimiento existente y carga
  proyectada.
- `panel_asilos_6cortes.csv` — asilos por AGEB en los seis cortes del DENUE.
- `figuras/` — las gráficas en PNG: `poblacion_y_oferta`, `lisa`, `alcaldias_vs_regiones`,
  `crecimiento_prediccion` y `desajuste_territorial`.

## Dos advertencias de interpretación

**La proyección no es una estimación de necesidad.** Es "cuántos asilos habrá si el mercado sigue
comportándose como hasta ahora". No existe en México un estándar normativo de camas por cada mil
adultos mayores contra el cual medir suficiencia.

**El "dónde" no es un mapa de acceso individual.** Un asilo es residencial: quien entra se muda, así
que la distancia desde su AGEB de origen no es la fricción que le impide usar el servicio. Por eso se
descartaron los métodos de área de captación (2SFCA), diseñados para clínicas y farmacias: el resultado
dependía por completo del radio —740 AGEBs sin cobertura con 2 km, 69 con 5 km— y el radio es la
definición de qué cuenta como accesible, no un parámetro que se afine.

Lo que la sección 4 mide en su lugar es **la aritmética del desajuste**, que no necesita ninguna
distancia: cuánta demanda nueva le toca a cada establecimiento existente, y cuál es la oferta por cada
diez mil personas de 60 y más.

**Y una limitación que conviene tener presente: el DENUE no trae número de camas.** Un asilo de 8 pesa
igual que uno de 80. Eso impide contestar si alguien conseguirá lugar, que depende de la capacidad
total de la ciudad. Es la mejora que más cambiaría este análisis.
