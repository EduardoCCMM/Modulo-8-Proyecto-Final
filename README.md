# ECOBICI CDMX 
## **Módulo 8 · Proyecto Final**

**Dashboard publicado:** https://rpubs.com/EliLopez01/1464119

**Notebook:** https://github.com/EduardoCCMM/Modulo-8-Proyecto-Final/blob/main/notebook_equipo_final.Rmd

**Reporte PDF:** https://github.com/EduardoCCMM/Modulo-8-Proyecto-Final/blob/main/Reporte_Ecobici_eq9.pdf

**Dashboard:**

Dashboard HTML:**
---
### Integrantes

**Equipo 9**

- Ayala López Elizabeth
- Cruz Miguel Eduardo
- Marco Antonio Díaz López
- Romero Rossano Sebastian
- Toriz Pacheco Vanessa
- Hugo Valverde Guadalupe
---
## ¿De qué trata el proyecto?

ECOBICI, el sistema de bicicletas compartidas de la Ciudad de México, conecta estaciones mediante viajes cuya intensidad cambia según la hora y el territorio. Este proyecto analiza los **viajes registrados entre enero y diciembre de 2025** para responder:

> **¿Cómo varía la demanda de ECOBICI por estación y hora del día durante 2025, y pueden agruparse las estaciones en perfiles de uso similares que ayuden a identificar periodos de mayor presión operativa?**

Para ello se hace lo siguiente:

- **Descripción de la demanda:** por hora, día de la semana, mes, estación y alcaldía.
- **Balance de flujos:** arribos menos retiros por estación.
- **Clustering (K-Means):** agrupa estaciones según su perfil horario de retiros.
- **Regresión:** modela la demanda horaria de la estación con más retiros y evalúa su error fuera de muestra.

> El análisis describe viajes efectivamente realizados. No mide personas únicas, disponibilidad instantánea de bicicletas ni demanda no atendida.

---

## Archivos del repositorio

| Archivo | Descripción |
|---|---|
| `notebook_equipo_final.Rmd` | Notebook del equipo: obtención, exploración y limpieza de los datos. |
| `dashboard_ecobici_2025.Rmd` | Código fuente del dashboard (`flexdashboard`). |
| `dashboard_ecobici_2025.html` | Dashboard ya generado (Knit), listo para abrirse en el navegador. |
| `Reporte_Ecobici` | Entrega de reporte en PDF. |
| `README.md` | Este archivo. |

Los **datos no se incluyen** en el repositorio por su tamaño. Hay que descargarlos como se explica abajo.

---

## Datos

### Fuente

Datos abiertos de ECOBICI: **https://ecobici.cdmx.gob.mx/datos-abiertos/**

Se necesitan:

1. **Los 12 archivos mensuales de viajes de 2025** (de enero a diciembre).
2. **El catálogo de estaciones** (con colonia, alcaldía y coordenadas).

> Registra en tu reporte la **fecha de descarga** y la **versión del catálogo**, porque los datos pueden actualizarse.

### Dónde colocarlos

Crea una carpeta `data/` en la misma carpeta donde estén los `.Rmd`, con esta estructura:

```
Proyecto Final/
├── notebook_equipo_final.Rmd
├── dashboard_ecobici_2025.Rmd
├── dashboard_ecobici_2025.html
├── README.md
└── data/
    ├── Catálogo Ecobici.csv          ← catálogo de estaciones
    ├── viajes_limpios_2025.rds       ← se genera automáticamente (ver abajo)
    └── raw/
        ├── 2025-01.csv
        ├── 2025-02.csv
        ├── ...
        └── 2025-12.csv
```

**Importante:**
- Los viajes deben llamarse exactamente `2025-01.csv`, `2025-02.csv`, … `2025-12.csv` y estar en `data/raw/`.
- Debe haber **un solo** catálogo: `data/Catálogo Ecobici.csv` **o** `data/Catálogo_Ecobici.csv` (no ambos).
- El archivo `viajes_limpios_2025.rds` **no se descarga**: se crea la primera vez que corres el dashboard desde los CSV.

### Columnas esperadas

**Viajes (CSV mensuales):** `Edad_Usuario`, `Bici`, `Fecha_Retiro`, `Hora_Retiro`, `Fecha_Arribo`, `Hora_Arribo`, `Genero_Usuario`, `Ciclo_Estacion_Retiro`, `Ciclo_EstacionArribo`.

**Catálogo:** `num_cicloe`, `colonia`, `alcaldia`, `latitud`, `longitud`.

---

## Cómo reproducir el dashboard

### 1. Instalar los paquetes (una sola vez)

```r
install.packages(c("flexdashboard", "data.table", "dplyr", "tidyr", "lubridate",
                   "ggplot2", "stringr", "scales", "cluster", "DT", "plotly",
                   "leaflet", "broom", "jsonlite"))
```

### 2. Preparar las rutas

Abre `dashboard_ecobici_2025.Rmd` y revisa el chunk `setup`. Las rutas deben ser **relativas**, así funcionan en cualquier computadora:

```r
ruta_rds <- "data/viajes_limpios_2025.rds"
ruta_raw <- "data/raw"
candidatos <- c("data/Catálogo Ecobici.csv", "data/Catálogo_Ecobici.csv")
```

> `ruta_rds` debe apuntar a un **archivo** `.rds`, no a una carpeta.

### 3. Primera ejecución: construir la base desde los CSV

En el mismo chunk, cambia:

```r
reconstruir_desde_csv <- TRUE
```

Haz **Knit** (o ejecuta `rmarkdown::render("dashboard_ecobici_2025.Rmd")`). Esto lee los 12 CSV, limpia los datos y guarda `data/viajes_limpios_2025.rds`. Puede tardar varios minutos.

### 4. Ejecuciones siguientes

Regresa a `reconstruir_desde_csv <- FALSE` para que el dashboard cargue directamente el `.rds` (mucho más rápido).

El resultado es `dashboard_ecobici_2025.html`, que puede abrirse en cualquier navegador. Los mapas requieren conexión a internet para cargar el mapa base.

---

## Decisiones de limpieza (resumen)

- Se eliminan duplicados exactos y registros con fecha u hora no interpretable.
- Solo se conservan retiros ocurridos en 2025.
- Duraciones fuera de **1–180 min** y edades fuera de **12–90 años** se tratan como faltantes únicamente en sus análisis específicos (no se eliminan viajes de la demanda).
- Los IDs de estación se conservan como texto (por ejemplo `004`).
- K-Means usa estaciones con **al menos 50 retiros**, representadas por su proporción de viajes en cada una de las 24 horas.
- La regresión usa el 80 % inicial de las fechas para entrenamiento y el 20 % restante para prueba.

Estos rangos son reglas analíticas del equipo, no límites oficiales de operación.

---

## Limitaciones

- Un solo año permite describir variación mensual, pero no confirmar estacionalidad entre años.
- No se cuenta con inventario instantáneo, capacidad de anclajes, clima, festivos ni movimientos de redistribución del operador.
- El balance (arribos − retiros) indica señales de desequilibrio, pero no demuestra que una estación estuviera vacía o llena.
- Las ubicaciones provienen del catálogo disponible y no reconstruyen cambios históricos de la red.

---


---

## Fuente de datos

Gobierno de la Ciudad de México · ECOBICI — Datos abiertos: https://ecobici.cdmx.gob.mx/datos-abiertos/
