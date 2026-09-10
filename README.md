# proyecto_databrick

Proyecto práctico de Ingeniería de Datos desarrollado en **Databricks Free Edition**, utilizando **PySpark, SQL, Unity Catalog y Delta Lake**.

El proyecto simula un proceso de ingesta y procesamiento de datos de viajes, incorporando controles de calidad, manejo de registros rechazados, deduplicación, transformación analítica y controles de conciliación.

## Arquitectura

```text
Fuente de datos simulada
        ↓
00_simulacion_data
        ↓
01_ingesta_bronze
        ↓
      BRONZE
        ↓
02_data_quality
     ├── QUARANTINE
     └── SILVER
          ↓
      03_gold
     ├── GOLD DIARIO
     └── GOLD MENSUAL
          ↓
      04_control
```

## Tecnologías

* **Databricks Free Edition**
* **PySpark**
* **SQL**
* **Delta Lake**
* **Unity Catalog**
* **Python**
* **Git / GitHub**

## Flujo del proyecto

### 00 — Simulación de datos

Se generan cuatro cargas de datos a partir de la tabla de ejemplo de viajes de Nueva York.

La simulación incorpora:

* Registros duplicados dentro de las cargas.
* Duplicados históricos entre diferentes cargas.
* Errores de calidad en fechas, distancia y tarifa.
* Archivos independientes por cada carga.

Los archivos generados se almacenan en un **Unity Catalog Volume** para simular una fuente de datos de entrada.

### 01 — Ingesta Bronze

Se detectan automáticamente las cargas disponibles y se incorporan a una tabla **Delta Lake**.

La capa Bronze conserva los datos originales junto con metadatos de procesamiento, incluyendo:

* Identificador de lote.
* Fecha de ingesta.
* Fecha de proceso.
* Fuente.
* Archivo de origen.
* Hash del registro.

También se mantiene una tabla de control de cargas para registrar el resultado del procesamiento.

### 02 — Calidad de datos

Se aplican reglas para identificar registros inválidos, incluyendo:

* Fechas de recogida nulas.
* Fechas de destino nulas.
* Distancias negativas.
* Tarifas negativas.
* Fechas de destino anteriores a la fecha de recogida.

Los registros inválidos se almacenan en **Quarantine**.

Posteriormente se identifican los registros duplicados mediante un hash del contenido del registro y se conserva una única ocurrencia para construir la capa **Silver**.

Resultado del procesamiento:

| Resultado                  | Registros |
| -------------------------- | --------: |
| Registros procesados       |    21.261 |
| Registros válidos          |    20.832 |
| Registros únicos aceptados |    19.628 |
| Registros duplicados       |     1.204 |
| Registros rechazados       |       429 |

### 03 — Gold

A partir de Silver se construyen tablas analíticas agregadas:

* **Gold diario:** indicadores por fecha.
* **Gold mensual:** indicadores por año y mes.

Entre los indicadores calculados se encuentran:

* Cantidad de viajes.
* Distancia total.
* Distancia promedio.
* Ingresos totales.
* Tarifa promedio.

### 04 — Control

Se realizan controles de conciliación para verificar la consistencia del procesamiento.

Se valida que:

```text
Registros leídos
=
Registros únicos aceptados
+ Registros duplicados
+ Registros rechazados
```

También se verifica la consistencia entre:

```text
Silver ↔ registros únicos aceptados

Quarantine ↔ registros rechazados

Silver ↔ Gold diario

Silver ↔ Gold mensual
```

Todos los controles implementados finalizan correctamente con estado **OK**.

## Estructura del repositorio

```text
proyecto_databrick/
│
├── README.md
│
└── notebook/
    ├── Schema.ipynb
    ├── 00_simulacion_data.ipynb
    ├── 01_ingesta_bronze.ipynb
    ├── 02_data_quality.ipynb
    ├── 03_gold.ipynb
    └── 04_control.ipynb
```

## Objetivo

El objetivo del proyecto es demostrar, mediante un caso práctico, conocimientos en:

* Ingesta de datos.
* Procesamiento distribuido con PySpark.
* Arquitectura Bronze, Silver y Gold.
* Delta Lake.
* Unity Catalog.
* Calidad y validación de datos.
* Deduplicación.
* Trazabilidad de cargas.
* Conciliación de información.
* Transformaciones analíticas.
* Uso de Git y GitHub para control de versiones.
