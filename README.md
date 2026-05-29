<div align="center">

<img src="capturas/logo_epn.png" height="80" align="left"/>
<img src="capturas/logo_fis.png" height="80" align="right"/>

<h1><strong>Escuela Politécnica Nacional</strong></h1>

### Facultad de Ingeniería de Sistemas

**Business Intelligence (ISWD743) · GR2SW_2026-1**

**Proyecto I Bimestre**

---

**Fecha:**  
29 de Mayo, 2026

**Integrantes:**  
José Arias · Andrea Chicaiza · Andreina Pallo · Juan Quisilema · Juan Suárez

---

<h2><strong>Caso de estudio:</strong></h2>

# Mercado Laboral y Vivienda en Ecuador

*Fuente de datos: ENEMDU Q1 2026 · INEC Ecuador*

![Pentaho](https://img.shields.io/badge/Pentaho-ETL-orange?style=flat)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat&logo=postgresql&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)

</div>

---
<h2><strong>Tabla de contenidos</strong></h2>

- [Mercado Laboral y Vivienda en Ecuador](#mercado-laboral-y-vivienda-en-ecuador)
  - [Introducción](#introducción)
  - [Objetivo](#objetivo)
  - [Datasets](#datasets)
  - [1. Problema y solución](#1-problema-y-solución)
    - [1.1 ¿Qué es la ENEMDU y por qué se usa?](#11-qué-es-la-enemdu-y-por-qué-se-usa)
    - [1.2 Descripción de los datasets](#12-descripción-de-los-datasets)
      - [Dataset N°1: persona\_corregido.csv](#dataset-n1-persona_corregidocsv)
      - [Dataset N°2: vivienda\_limpio.csv](#dataset-n2-vivienda_limpiocsv)
    - [1.3 Preguntas de negocio](#13-preguntas-de-negocio)
    - [1.4 Stack tecnológico](#14-stack-tecnológico)
  - [2. Justificación de diseño y modelado dimensional](#2-justificación-de-diseño-y-modelado-dimensional)
  - [3. Proceso ETL](#3-proceso-etl)
    - [3.1 Staging](#31-staging)
    - [3.2 Dimensiones (tiempo, geografía, persona, ocupación)](#32-dimensiones-tiempo-geografía-persona-ocupación)
    - [3.3 Dimensiones (vivienda, servicios)](#33-dimensiones-vivienda-servicios)
    - [3.4 Tablas de hechos](#34-tablas-de-hechos)
    - [3.5 Job principal](#35-job-principal)
  - [4. Análisis de insights (OLAP)](#4-análisis-de-insights-olap)
  - [5. Recomendaciones al negocio](#5-recomendaciones-al-negocio)
  - [Referencias Bibliográficas](#referencias-bibliográficas)

---

## Introducción

La Encuesta Nacional de Empleo, Desempleo y Subempleo (ENEMDU) es el principal instrumento estadístico del Ecuador para medir las condiciones del mercado laboral y de vida de los hogares, producida periódicamente por el INEC con representatividad nacional. Este proyecto toma los datos del Q1 2026 (enero–marzo) para construir una solución de Business Intelligence end-to-end: desde la carga y transformación de los datos en Pentaho hacia un modelo dimensional en PostgreSQL, hasta la generación de reportes OLAP interactivos en Power BI que permitan responder preguntas concretas sobre empleo, ingresos y condiciones de vivienda en el Ecuador.

## Objetivo

Desarrollar una solución de inteligencia de negocios que recolecte datos reales del Ecuador, los transforme y cargue en un modelo analítico (ETL) en Pentaho y muestre reportes OLAP en Power BI.

---

## Datasets

| Dataset | Filas | Columnas | Granularidad |
|---|---|---|---|
| `persona_corregido.csv` | 82.894 | 23 | 1 persona por 1 período |
| `vivienda_limpio.csv` | 26.354 | 18 | 1 hogar por 1 período |


---

## 1. Problema y solución

### 1.1 ¿Qué es la ENEMDU y por qué se usa?

La Encuesta Nacional de Empleo, Desempleo y Subempleo (ENEMDU) es el principal instrumento estadístico del Ecuador para medir las condiciones del mercado laboral y las condiciones de vida de los hogares, producida por el INEC con representatividad urbana y rural en las 24 provincias del país [[1]](#referencias).

Para este proyecto se utiliza la edición Q1 2026 (enero, febrero y marzo) por las siguientes razones:

- Es la fuente oficial del Estado ecuatoriano sobre empleo e ingresos.
- Incluye el factor de expansión muestral, que permite proyectar resultados a la población nacional.
- Combina información de personas y hogares en la misma muestra, habilitando análisis cruzados entre mercado laboral y condiciones de vivienda.



---

### 1.2 Descripción de los datasets

#### Dataset N°1: persona_corregido.csv

| ![Dataset persona_corregido.csv](capturas/dataset_persona.png) |
| :---: |
| *Figura 1: Vista previa del dataset 'persona_corregido.csv*' |

Registra información individual de cada persona encuestada durante los tres meses del Q1 2026. Cubre cuatro grandes bloques temáticos: 

- **Identificación:** claves de persona y hogar
- **Perfil Demográfico:** sexo, edad, nivel de instrucción,
- **Situación Laboral:** condición de actividad, sector, rama, grupo ocupacional, flags de empleo y desempleo
- **Ingresos:** laboral y per cápita 

Incluye además las variables de geografía y tiempo que lo vinculan con el dataset de vivienda.



| Campo | Tipo | Descripción |
|---|---|---|
| `id_persona` | VARCHAR | Identificador único de la persona |
| `id_hogar` | VARCHAR | Identificador del hogar al que pertenece la persona |
| `id_vivienda` | INT | Identificador de la vivienda |
| `periodo` | INT | Período de la encuesta: 202601, 202602, 202603 |
| `area` | VARCHAR | Zona geográfica: Urbano / Rural |
| `ciudad` | INT | Código INEC de parroquia |
| `sexo` | VARCHAR | Sexo de la persona: Hombre / Mujer |
| `edad` | INT | Edad en años (rango: 0–98) |
| `nivel_instruccion` | VARCHAR | Nivel educativo alcanzado: Ninguno, Primaria, Secundaria, Superior, Centro de Alfabetización |
| `condicion_actividad` | VARCHAR | Situación en el mercado laboral: Empleado Pleno, Subempleado, Desempleado Abierto, Desempleado Oculto, Inactivo, Fuera de PEA, entre otros |
| `empleo` | BOOLEAN | Indica si la persona está empleada |
| `desempleo` | BOOLEAN | Indica si la persona está desempleada |
| `sector_empleo` | VARCHAR | Sector de trabajo: Público, Privado, Doméstico, No Remunerado, Sin información |
| `rama_actividad` | VARCHAR | Rama económica CIIU (21 categorías) |
| `grupo_ocupacional` | VARCHAR | Grupo ocupacional CIUO (10 categorías) |
| `ingreso_laboral` | DECIMAL | Ingreso mensual laboral en USD |
| `ingreso_percapita` | DECIMAL | Ingreso per cápita del hogar en USD |
| `factor_expansion` | DECIMAL | Ponderador muestral para proyección a nivel nacional |
| `cod_provincia` | INT | Código numérico de provincia (1–24) |
| `provincia` | VARCHAR | Nombre de la provincia (24 provincias del Ecuador) |
| `anio` | INT | Año de la encuesta |
| `mes` | INT | Mes numérico: 1, 2, 3 |
| `mes_nombre` | VARCHAR | Nombre del mes: Enero, Febrero, Marzo |


---

#### Dataset N°2: vivienda_limpio.csv

| ![Dataset vivienda_limpio.csv](capturas/dataset_vivienda.png) |
| :---: |
| *Figura 2: Vista previa del dataset 'vivienda_limpio.csv'* |

Registra las condiciones de cada hogar encuestado durante el Q1 2026. Cubre dos bloques temáticos principales: 
- **Características físicas de la vivienda:** tipo, materiales de piso y paredes, tenencia.
- **Acceso a servicios básicos:** fuente de agua, alumbrado, saneamiento y recolección de basura.

Además, comparte con el dataset de persona las variables de identificación del hogar, geografía y tiempo.  



| Campo | Tipo | Descripción |
|---|---|---|
| `id_hogar` | VARCHAR | Identificador único del hogar |
| `periodo` | INT | Período de la encuesta: 202601, 202602, 202603 |
| `area` | VARCHAR | Zona geográfica: Urbano / Rural |
| `ciudad` | INT | Código INEC de parroquia |
| `tipo_vivienda` | VARCHAR | Tipo de vivienda: Casa o villa, Departamento, Mediagua, Rancho, Cuarto inquilinato, Covacha, Choza/Otro |
| `material_piso` | VARCHAR | Material del piso: Entablado/Parquet, Baldosa/Vinyl, Ladrillo/Cemento, Mármol, Madera, Tierra/Caña, Piedra, Otro |
| `material_paredes` | VARCHAR | Material de las paredes: Hormigón/Bloque, Adobe/Tapia, Madera, Caña revestida, Caña no revestida, Otros, Sin paredes |
| `servicio_sanitario` | VARCHAR | Tipo de servicio sanitario: Conectado a red pública, Pozo séptico, Descarga directa, Letrina, No tiene |
| `fuente_agua` | VARCHAR | Fuente de abastecimiento de agua: Red pública, Otra tubería, Pozo, Río/Vertiente, Agua lluvia, Pila pública, Carro repartidor |
| `tipo_alumbrado` | VARCHAR | Fuente de energía eléctrica: Red de empresa eléctrica, Panel solar, Generador, Otro |
| `eliminacion_basura` | INT | Forma de eliminación de basura (código INEC 1–5) |
| `tenencia_vivienda` | INT | Tipo de tenencia de la vivienda (código INEC 1–6) |
| `factor_expansion` | DECIMAL | Ponderador muestral para proyección a nivel nacional |
| `cod_provincia` | INT | Código numérico de provincia (1–24) |
| `provincia` | VARCHAR | Nombre de la provincia (24 provincias del Ecuador) |
| `anio` | INT | Año de la encuesta |
| `mes` | INT | Mes numérico: 1, 2, 3 |
| `mes_nombre` | VARCHAR | Nombre del mes: Enero, Febrero, Marzo |

---

### 1.3 Preguntas de negocio
Con base en los datos disponibles, se definieron cinco preguntas de negocio orientadas a extraer hallazgos concretos sobre el mercado laboral y las condiciones de vida en el Ecuador. Estas preguntas guiaron todo el proyecto, desde el diseño del modelo dimensional hasta la construcción de los reportes OLAP en Power BI.

| # | Pregunta |
|---|---|
| **P1** | ¿Cómo varía la tasa de empleo y el ingreso laboral promedio entre zonas urbanas y rurales, y entre provincias? |
| **P2** | ¿Existe una brecha salarial significativa según sexo, nivel de instrucción y sector de empleo? |
| **P3** | ¿Qué porcentaje de hogares carece de agua potable, electricidad de red pública o saneamiento adecuado, y cómo se distribuye por provincia? |
| **P4** | ¿Cómo evolucionaron las tasas de empleo y desempleo mes a mes durante enero, febrero y marzo de 2026? |
| **P5** | ¿Existe correlación entre el índice de acceso a servicios básicos de un hogar y el ingreso per cápita de sus miembros? |

---

### 1.4 Stack tecnológico

| Etapa | Herramienta | Rol |
|---|---|---|
| ETL | <img src="https://www.datageeks.pl/images/Article-images/102-How-to-open-Microsoft-XLSB/pdi.png" height="20"/> Pentaho Data Integration | Staging → dimensiones → hechos, con transformaciones de negocio |
| Almacenamiento | <img src="https://www.postgresql.org/media/img/about/press/elephant.png" height="20"/> PostgreSQL | Data warehouse dimensional |
| Visualización OLAP | <img src="https://upload.wikimedia.org/wikipedia/commons/c/cf/New_Power_BI_Logo.svg" height="20"/> Power BI Desktop | Reportes interactivos, medidas DAX, jerarquías y slicers |


---
---
## 2. Justificación de diseño y modelado dimensional

### 2.1 Por qué constelación
 
El modelo dimensional elegido para este proyecto es el **modelo constelación**, también conocido como esquema de galaxia. Esta decisión responde a una característica estructural del conjunto de datos ENEMDU Q1 2026 los dos datasets fuente `persona_corregido.csv` y `vivienda_limpio.csv` que describen **procesos de negocio distintos con granularidades diferentes**, lo cual hace inviable tanto un esquema estrella simple como un esquema copo de nieve.
 
#### Diferencia de granularidad entre las dos tablas de hechos
 
| Tabla de hechos | Dataset origen | Granularidad | Filas | Unidad de observación |
|---|---|---|---|---|
| `FACT_SITUACION_LABORAL` | `persona_corregido.csv` | Una persona por periodo | 82.894 | Individuo × mes |
| `FACT_CONDICION_HOGAR` | `vivienda_limpio.csv` | Un hogar por periodo | 26.354 | Hogar × mes |
 
Mezclar ambas granularidades en una sola tabla de hechos produciría una **trampa de abanico** al unir los 82.894 registros de personas con los 26.354 registros de hogares a través de `id_hogar`, las métricas de vivienda se multiplicarían por el número de personas del hogar, inflando artificialmente los conteos. El modelo constelación resuelve esto manteniendo cada proceso de negocio en su propia tabla de hechos y conectándolas únicamente a través de dimensiones compartidas.
 
Adicionalmente, la integridad referencial entre ambas fuentes es del **100 %**. Es decir, los 26.354 hogares presentes en `vivienda_limpio.csv` coinciden exactamente con los identificadores de hogar en `persona_corregido.csv`, lo que garantiza que los cruces analíticos entre facts no generarán valores nulos ni pérdida de registros.
 
---
 
### 2.2 Las 6 dimensiones del modelo y cuáles son compartidas
 
El modelo está compuesto por **6 dimensiones**, de las cuales 2 son compartidas por ambas tablas de hechos y 4 son exclusivas de una de ellas.
 
#### Dimensiones compartidas
 
Estas dimensiones actúan como el núcleo de la constelación. En Power BI, un único slicer sobre cualquiera de ellas filtra simultáneamente `FACT_SITUACION_LABORAL` y `FACT_CONDICION_HOGAR`, permitiendo responder preguntas que cruzan la situación laboral con las condiciones del hogar.
 
**`DIM_TIEMPO`**
Construida a partir de las columnas `mes`, `mes_nombre` y `anio` presentes en ambos datasets. Se añade en Pentaho la columna derivada `orden_mes` (valores 1, 2, 3) para garantizar el ordenamiento cronológico correcto en los ejes de tiempo de Power BI, ya que el nombre del mes en texto ordena alfabonéticamente por defecto. Cardinalidad: 3 registros (Enero, Febrero, Marzo 2026).
 
**`DIM_GEOGRAFIA`**
Construida a partir de `area`, `provincia`, `cod_provincia` y `ciudad`. La columna `area` (Rural / Urbano) es el slicer geográfico principal para las preguntas de análisis P1 y P3. La columna `ciudad` contiene el código INEC (fórmula: `cod_provincia × 10.000 + parroquia`) y requiere un catálogo externo del INEC para resolver los nombres de parroquia; su cardinalidad es de 587 valores. La columna `provincia` —con sus 24 categorías— habilita visualizaciones de mapa de coropletas en Power BI. Los valores son idénticos entre ambos datasets para el mismo hogar (0 inconsistencias verificadas).
 
#### Dimensiones exclusivas de `FACT_SITUACION_LABORAL`
 
**`DIM_PERSONA`**
Contiene los atributos demográficos individuales: `sexo` (Hombre / Mujer), `nivel_instruccion` (5 categorías: Ninguno, Centro de Alfabetización, Primaria, Secundaria, Superior) y la columna derivada `grupo_etario`, que se construye en Pentaho a partir de la columna `edad`. Esta dimensión es la clave para responder la pregunta sobre brecha salarial de género por nivel de instrucción (P2).
 
**`DIM_OCUPACION`**
Contiene los atributos del mercado laboral: `condicion_actividad` (10 categorías, corregida en Python), `sector_empleo` (Público, Privado, Doméstico, No Remunerado, Sin información), `rama_actividad` (21 ramas CIIU) y `grupo_ocupacional` (10 grupos CIUO). En Pentaho se deriva la columna `categoria_pea` para clasificar a cada persona como Ocupada, Desempleada o Fuera de PEA, que es el denominador correcto para el cálculo de tasas de empleo y desempleo expandidas. Los 42.974 nulos estructurales en `sector_empleo`, `rama_actividad` y `grupo_ocupacional` corresponden a personas fuera de la PEA y se imputarán como `"No aplica"` en Pentaho.
 
#### Dimensiones exclusivas de `FACT_CONDICION_HOGAR`
 
**`DIM_TIPO_VIVIENDA`**
Contiene las características físicas de la unidad habitacional: `tipo_vivienda` (7 categorías: Casa o villa, Departamento, Mediagua, Rancho, Cuarto inquilinato, Covacha, Choza/Otro), `material_piso` (8 categorías), `material_paredes` (7 categorías) y `tenencia_vivienda`. Esta última llega como códigos numéricos 1–6 y requiere decodificación en Pentaho mediante Stream Lookup con el diccionario INEC.
 
**`DIM_SERVICIOS_BASICOS`**
Contiene el acceso a servicios públicos del hogar: `fuente_agua` (7 categorías, ya decodificada), `tipo_alumbrado` (4 categorías, ya decodificada), `servicio_sanitario` (5 categorías, ya decodificada) y `eliminacion_basura` (códigos 1–5, pendiente de decodificar en Pentaho). En Pentaho se derivan tres flags binarios: `agua_potable` (fuente_agua = "Red pública"), `electricidad_red` (tipo_alumbrado = "Red empresa eléctrica") y `saneamiento_adecuado` (servicio_sanitario = "Conectado a red pública"). Estos flags son las medidas directas para responder la pregunta P3.
 
---
 
### Diagrama ER del modelo constelación
 
El diagrama a continuación representa la estructura completa del modelo dimensional. Las tablas de hechos se ubican al centro; las dimensiones compartidas en la parte superior; las dimensiones exclusivas, agrupadas por tabla de hechos, en la parte inferior. Las relaciones son todas de tipo **muchos-a-uno** desde la tabla de hechos hacia la dimensión (notación: `*` → `1`).
 
 | ![Diseño esquema constelación](capturas/Diseño_modelo_contelacion.png) |
| :---: |
| *Figura 3: Diseño de esquema constelación* |
 
> **Nota:** La relación entre `FACT_SITUACION_LABORAL` y `FACT_CONDICION_HOGAR` a través de `id_hogar` no es una relación dimensional estándar; representa la integridad referencial del dataset de origen. En Power BI esta relación no se activa como relación de filtro entre facts; los cruces se realizan mediante medidas DAX que unen ambas facts a través de las dimensiones compartidas.


---
## 3. Proceso ETL

### 3.1 Staging

### 3.2 Dimensiones (tiempo, geografía, persona, ocupación)

### 3.3 Dimensiones (vivienda, servicios)

### 3.4 Tablas de hechos

### 3.5 Job principal


---
---
## 4. Análisis de insights (OLAP)
En esta fase del proyecto se utilizó Power BI Desktop para construir visualizaciones OLAP interactivas sobre el modelo dimensional implementado en PostgreSQL.
Las dimensiones compartidas `DIM_TIEMPO` y `DIM_GEOGRAFIA` permitieron analizar simultáneamente los procesos de negocio relacionados con empleo y condiciones de vivienda mediante filtros dinámicos (slicers), jerarquías y medidas DAX.

Las preguntas de negocio P1–P4 fueron respondidas mediante dashboards analíticos construidos sobre las tablas de hechos `FACT_SITUACION_LABORAL` y `FACT_CONDICION_HOGAR`.

---

### P1. ¿Cómo varía la tasa de empleo y el ingreso laboral promedio entre zonas urbanas y rurales, y entre provincias?

Para responder esta pregunta se construyó un dashboard comparativo utilizando:

* Segmentadores por provincia y área geográfica.
* Gráficos de barras para comparar la tasa de empleo.
* Tarjetas KPI para ingreso laboral promedio.
* Mapa geográfico por provincias.

Las medidas utilizadas en Power BI fueron:

```DAX
Total Empleados =
SUMX(
    FACT_SITUACION_LABORAL,
    FACT_SITUACION_LABORAL[empleo] *
    FACT_SITUACION_LABORAL[factor_expansion]
)

Tasa Empleo =
DIVIDE(
    [Total Empleados],
    SUM(FACT_SITUACION_LABORAL[factor_expansion])
)

Ingreso Promedio =
AVERAGE(FACT_SITUACION_LABORAL[ingreso_laboral])
```

### Hallazgos

* Las zonas urbanas presentan mayores niveles de empleo formal en comparación con las zonas rurales.
* Las provincias con mayor actividad económica muestran ingresos laborales promedio superiores al promedio nacional.
* Las diferencias territoriales evidencian concentración económica en determinadas regiones del país.

|                                 ![P1](capturas/p1_powerbi.png)                                 |
| :--------------------------------------------------------------------------------------------: |
| *Figura X. Dashboard OLAP para análisis de empleo e ingresos por provincia y área geográfica.* |

---

### P2. ¿Existe una brecha salarial significativa según sexo, nivel de instrucción y sector de empleo?

Para esta pregunta se utilizaron:

* Gráficos de barras agrupadas.
* Segmentadores por sexo y nivel educativo.
* Comparación de ingresos promedio entre hombres y mujeres.

### Medidas utilizadas

```DAX
Ingreso Promedio Hombre =
CALCULATE(
    AVERAGE(FACT_SITUACION_LABORAL[ingreso_laboral]),
    DIM_PERSONA[sexo] = "Hombre"
)

Ingreso Promedio Mujer =
CALCULATE(
    AVERAGE(FACT_SITUACION_LABORAL[ingreso_laboral]),
    DIM_PERSONA[sexo] = "Mujer"
)

Brecha Salarial % =
DIVIDE(
    [Ingreso Promedio Hombre] - [Ingreso Promedio Mujer],
    [Ingreso Promedio Hombre]
)
```

### Hallazgos

* Existe una diferencia salarial observable entre hombres y mujeres en múltiples sectores laborales.
* El nivel de instrucción superior incrementa significativamente el ingreso promedio.
* La brecha salarial es más evidente en ciertos sectores económicos.

|                           ![P2](capturas/p2_powerbi.png)                          |
| :-------------------------------------------------------------------------------: |
| *Figura X. Dashboard OLAP para análisis de brecha salarial por sexo y educación.* |

---

### P3. ¿Qué porcentaje de hogares carece de agua potable, electricidad de red pública o saneamiento adecuado, y cómo se distribuye por provincia?

Se construyeron visualizaciones geográficas y gráficos comparativos para evaluar el acceso a servicios básicos.

### Medidas utilizadas

```DAX
% Agua Potable =
DIVIDE(
    SUMX(
        FACT_CONDICION_HOGAR,
        FACT_CONDICION_HOGAR[agua_potable] *
        FACT_CONDICION_HOGAR[factor_expansion]
    ),
    SUM(FACT_CONDICION_HOGAR[factor_expansion])
)

% Electricidad =
DIVIDE(
    SUMX(
        FACT_CONDICION_HOGAR,
        FACT_CONDICION_HOGAR[electricidad_red] *
        FACT_CONDICION_HOGAR[factor_expansion]
    ),
    SUM(FACT_CONDICION_HOGAR[factor_expansion])
)

% Saneamiento =
DIVIDE(
    SUMX(
        FACT_CONDICION_HOGAR,
        FACT_CONDICION_HOGAR[saneamiento_adecuado] *
        FACT_CONDICION_HOGAR[factor_expansion]
    ),
    SUM(FACT_CONDICION_HOGAR[factor_expansion])
)
```

### Hallazgos

* Las zonas rurales presentan menor acceso a servicios básicos.
* Algunas provincias mantienen brechas importantes en saneamiento y acceso a agua potable.
* El acceso a electricidad posee mayor cobertura respecto a otros servicios básicos.

|                ![P3](capturas/p3_powerbi.png)                |
| :----------------------------------------------------------: |
| *Figura X. Dashboard OLAP sobre acceso a servicios básicos.* |

---

### P4. ¿Cómo evolucionaron las tasas de empleo y desempleo durante enero, febrero y marzo de 2026?

Se utilizaron gráficos de líneas temporales utilizando la dimensión `DIM_TIEMPO`.

### Medidas utilizadas

```DAX
Total Desempleados =
SUMX(
    FACT_SITUACION_LABORAL,
    FACT_SITUACION_LABORAL[desempleo] *
    FACT_SITUACION_LABORAL[factor_expansion]
)

Tasa Desempleo =
DIVIDE(
    [Total Desempleados],
    SUM(FACT_SITUACION_LABORAL[factor_expansion])
)
```

### Hallazgos

* Las tasas de empleo y desempleo muestran variaciones mensuales moderadas durante el trimestre.
* Se identifican fluctuaciones relacionadas con dinámicas económicas temporales.
* El análisis temporal evidencia la utilidad de la dimensión tiempo en modelos OLAP.

|                      ![P4](capturas/p4_powerbi.png)                      |
| :----------------------------------------------------------------------: |
| *Figura X. Dashboard OLAP de evolución temporal del empleo y desempleo.* |

---

### Uso de funcionalidades OLAP en Power BI

Durante el análisis se utilizaron capacidades OLAP propias de Power BI:

* Segmentadores dinámicos (Slicers)
* Drill-down jerárquico por provincia y tiempo
* Medidas DAX
* KPIs
* Visualizaciones geográficas
* Filtros cruzados entre dimensiones compartidas
* Exploración multidimensional de hechos

Estas funcionalidades permitieron realizar análisis interactivos y responder preguntas de negocio de manera visual y eficiente.


---
---
## 5. Recomendaciones al negocio


---
---
## Referencias Bibliográficas   
<a name="referencias"></a>
[1] Instituto Nacional de Estadística y Censos (INEC), "Estadísticas Laborales – abril 2026: Encuesta Nacional de Empleo, Desempleo y Subempleo (ENEMDU)," INEC Ecuador, 2026. [En línea]. Disponible en: https://www.ecuadorencifras.gob.ec/estadisticas-laborales-enemdu/ [Accedido: 27 de Mayo, 2026].
