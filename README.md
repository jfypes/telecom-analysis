# Análisis de clientes de ConnectaTel

Proyecto de análisis de datos sobre el comportamiento de los clientes de **ConnectaTel**, una empresa de telecomunicaciones de Latinoamérica con dos planes (Básico y Premium). El análisis cubre la información registrada hasta el año 2024 e incluye limpieza de datos, perfil estadístico de uso, detección de outliers y segmentación de clientes por nivel de uso y por edad, con foco en el abandono (churn).

## Objetivo

Construir un perfil estadístico de los clientes para identificar patrones de consumo, detectar comportamientos atípicos y segmentar la base de clientes, de modo que se puedan proponer estrategias de retención y mejoras en la oferta de planes.

## Datasets

| Archivo | Filas | Contenido |
|---|---|---|
| `plans.csv` | 2 | Planes actuales: nombre, mensajes incluidos, GB por mes, minutos incluidos, pago mensual en USD y costo por GB, mensaje y minuto extra. |
| `users_latam.csv` | 4,000 | Clientes: `user_id`, nombre, `age`, `city`, `reg_date`, `plan` y `churn_date`. |
| `usage.csv` | 40,000 | Registros de uso: `id`, `user_id`, `type` (`call` o `text`), `date`, `duration` (llamadas) y `length` (mensajes). |

## Etapas del análisis

1. **Carga y exploración:** lectura de los tres datasets, revisión de dimensiones, tipos de datos y vista previa de cada tabla.
2. **Problemas de calidad de datos:**
   - Nulos en `users_latam.csv`: `churn_date` (88.35%) y `city` (11.73%).
   - Nulos en `usage.csv`: `duration` (55.19%), `length` (44.74%) y `date` (0.125%, 50 filas).
   - Valores centinela: `age = -999` (55 clientes, 1.375%) y `city = "?"` (96 registros).
   - Fechas fuera de rango: registros con año 2026 en `reg_date`, cuando los datos llegan hasta 2024.
3. **Limpieza básica:**
   - `age`: el centinela `-999` se reemplaza por la mediana.
   - `city`: el centinela `"?"` se convierte en nulo.
   - `reg_date`: las fechas de 2026 se marcan como nulas (`NaT`).
   - `duration` y `length`: se verifica que sus nulos dependen de `type` (`duration` solo existe en llamadas y `length` solo en mensajes), por lo que se dejan como nulos.
   - `churn_date`: los nulos corresponden a clientes activos, por lo que se conservan.
4. **Perfil de uso por usuario:** agregación de `usage` por `user_id` (`cant_mensajes`, `cant_llamadas`, `cant_minutos_llamada`) y unión con `users` en el DataFrame `user_profile`. Resumen estadístico y distribución de planes (65% Básico, 35% Premium).
5. **Distribuciones y outliers:** histogramas por plan y boxplots de `age`, `cant_mensajes`, `cant_llamadas` y `cant_minutos_llamada`. Cálculo de límites con el método IQR para las tres variables de uso.
6. **Segmentación:**
   - `grupo_uso`: Bajo uso (llamadas < 5 y mensajes < 5), Uso medio (llamadas < 10 y mensajes < 10) y Alto uso (resto).
   - `grupo_edad`: Joven (< 30), Adulto (< 60) y Adulto mayor (resto).
   - Comparación del uso promedio y de la tasa de churn en cada segmento.
7. **Insight ejecutivo:** hallazgos, limitaciones y recomendaciones para la oferta de planes.

## Hallazgos principales

- El churn observado es de ~11.65% (466 de 4,000 clientes tienen `churn_date`).
- Los segmentos de uso se separan claramente en consumo, pero el churn no baja con el uso: **Alto uso** tiene la tasa más alta (13.98%), frente a 11.48% en Uso medio y 11.44% en Bajo uso.
- La actividad promedio es casi idéntica entre grupos de edad. El grupo **Adulto** es el más numeroso (~50% de la base) y el de mayor churn (12.49%), frente a 10.97% en Adulto mayor y 10.53% en Joven.
- Los outliers aparecen solo en el extremo superior de las tres variables de uso. Se interpretan como variabilidad real del comportamiento y se conservan.

## Limitaciones

El análisis es descriptivo. No se evaluó la significancia estadística de las diferencias de churn ni se analizó el churn por plan, por lo que los resultados se interpretan como señales a investigar y las recomendaciones como hipótesis a validar.

## Cómo ejecutar el notebook

El notebook (`Proyecto 6 - Análisis de una empresa de telecomunicaciones.ipynb`) lee los datos desde la ruta `/datasets/`, que existe en el entorno de TripleTen. Fuera de ese entorno hay que ajustar las rutas de carga.

### Opción 1: Google Colab

1. Abre [Google Colab](https://colab.research.google.com) y elige **Archivo > Subir cuaderno** (o **Archivo > Abrir cuaderno > GitHub** y pega la URL del repositorio).
2. Sube `plans.csv`, `users_latam.csv` y `usage.csv` desde el panel de archivos de la izquierda.
3. En la celda de carga de datos, cambia las rutas para que apunten a los archivos subidos:

```python
   plans = pd.read_csv('plans.csv')
   users = pd.read_csv('users_latam.csv')
   usage = pd.read_csv('usage.csv')
```

4. Ejecuta todo con **Entorno de ejecución > Ejecutar todo**.


## Guía de reproducción

1. Verifica que los tres CSV estén disponibles y que las rutas de la celda de carga coincidan con su ubicación.
2. Instala las dependencias: `pandas`, `numpy`, `matplotlib`, `seaborn` y `jupyter`.
3. Ejecuta las celdas **en orden, de arriba hacia abajo**. Varias dependen de variables creadas antes (`users`, `usage`, `usage_agg`, `user_profile`, `limites`).
4. Comprueba que los resultados clave coincidan con los de este documento:
   - `users`: 4,000 filas y 88.35% de nulos en `churn_date`.
   - `usage`: 40,000 filas y 50 nulos en `date`.
   - Distribución de planes: 64.875% Básico y 35.125% Premium.
   - Churn por grupo de uso: 13.98% (Alto), 11.48% (Medio) y 11.44% (Bajo).
5. Si algo no coincide, revisa que los CSV sean los originales y que no se hayan modificado antes de la carga.

## Tecnologías

Python, pandas, NumPy, Matplotlib, Seaborn y Jupyter Notebook.
