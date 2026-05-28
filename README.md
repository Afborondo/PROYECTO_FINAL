# 🏋️ Proyecto Final — EDA, ETL y Dashboard de un Gimnasio con Virtuagym

## 📖 Descripción del Proyecto



Este proyecto corresponde al Proyecto Final del programa de análisis de datos.

"En este proyecto se aplican y desarrollan los conocimientos adquiridos a lo largo del programa."

El proyecto se centra en el desarrollo completo de un caso real de analítica de negocio aplicado a un gimnasio.

El proyecto se centra en el análisis de datos históricos de un gimnasio utilizando información extraída desde el CRM **Virtuagym** mediante API.

- Transformación y limpieza profunda de los datos.
- Análisis descriptivo de los datos.
- Análisis estadístico de los datos.
- Visualización de los datos.
- Dashboard operativo.
- Informe explicativo del análisis.

A lo largo del proyecto se desarrollan procesos de:

- extracción de datos,
- transformación y limpieza,
- análisis exploratorio (EDA),
- análisis estadístico,
- visualización de datos,
- construcción de dashboard operativo.

---

# 🎯 Objetivo del Proyecto

En un negocio de gimnasio es fundamental entender los datos históricos (analítica descriptiva) de clientes, altas y bajas y visitas al gimnasio.

Entender los datos históricos es clave para la toma de decisiones.

Este proyecto tiene como finalidad desarrollar una analítica descriptiva sobre:

## Clientes

- Evolución temporal.
- Caracterización de clientes.
- Tipología de cuotas.
- Segmentación.

## Altas y bajas

- Evolución temporal.
- Caracterización.
- Análisis de retención.
- Comportamiento de abandono.

## Visitas al gimnasio

- Ratios de visitas por día y horario.
- Frecuencia de asistencia.
- Caracterización según tipo de cliente.
- Concurrencia y ocupación.

---

# 🛠️ Herramientas Utilizadas

## ETL y análisis de datos

- Python
- Pandas
- Requests
- python-dotenv
- Visual Studio Code

## 📈 Dashboard y Visualización



- Power BI

---

# 🗂️ Fuente de Datos

Los datos utilizados provienen del CRM **Virtuagym**.

Los datos se encuentran en el CRM Virtuagym. El acceso API se solicita al soporte del CRM junto con la documentación técnica necesaria.

Para acceder a la información fue necesario:

1. Solicitar acceso API al soporte del CRM.
2. Obtener las credenciales:

```env
club_secret
api_key
```

3. Configurar las variables de entorno mediante archivo `.env`.

Como buena práctica de seguridad, el archivo `.env` se añade al `.gitignore` para evitar compartir credenciales en GitHub.

---

# 📚 Datasets Utilizados

A partir de la documentación del API se seleccionaron los siguientes datasets:

| Dataset | Descripción |
|---|---|
| Club Members | Clientes del gimnasio |
| Club Employees | Empleados del gimnasio |
| Membership Definition | Definición de membresías |
| Membership Instances | Instancias de membresías |
| Club Visits | Registros de acceso y visitas |

---

# ⚙️ Arquitectura ETL



## Extracción de datos

La extracción se realizó mediante peticiones HTTP utilizando la librería `requests`.

El acceso y descarga de los datos se realiza con Python utilizando la librería `requests`, mientras que la transformación y limpieza se desarrolla con `pandas` y estructuras `DataFrame`.

Cada dataset posee:

- endpoints específicos,
- métodos de acceso,
- parámetros,
- sistema de paginación,
- limitaciones de consultas por hora y por día.

Para optimizar el proceso se desarrollaron distintos objetos y utilidades:

#Cada dataset dispone de endpoints, métodos, parámetros y sistemas de paginación específicos. Además, el API posee limitaciones de consultas máximas por hora y por día.

## Gestión del API

Se implementó un objeto encargado de:

- controlar límites de peticiones,
- gestionar solicitudes,
- manejar paginación,
- centralizar autenticación.

### Conversión a DataFrames

Se desarrolló un segundo objeto para:

- descargar datasets,
- convertir respuestas JSON en DataFrames,
- automatizar procesos de carga.

---

# 🔄 Actualización de Datos

Se implementaron dos estrategias de actualización:

Se implementan dos tipos de actualización de datos: carga completa (`full refresh`) y carga incremental (`incremental refresh`) según las capacidades del API.

## Full refresh

Carga completa de los datasets.

## Incremental refresh

Actualización incremental utilizando timestamps proporcionados por el API.

Todos los datasets permiten actualización incremental excepto:

- `Membership Instances`

ya que no dispone de campo timestamp.

Para soportar ambos sistemas se crearon metadatos asociados a cada dataset:

- tipo de actualización,
- timestamp de última sincronización,
- cursor de paginación.

---

# 📊 EDA — Comprensión General de los Datos



La primera fase del proyecto consiste en comprender la estructura y calidad de los datos.

## Técnicas utilizadas

- Carga inicial de datos.
- Revisión de tipos de datos.
- Estadística descriptiva.
- Resumen estadístico.
- Identificación de valores nulos.

## Métodos principales de pandas

```python
# Primeras filas
 df.head()

# Información del dataset
 df.info()

# Estadísticos descriptivos
 df.describe()
```

---

# 👥 Análisis del Dataset CLUB MEMBERS

Según la documentación del API, el dataset contiene información relacionada con:

- identificadores de clientes,
- datos personales,
- fechas de alta,
- estado de membresía,
- información demográfica.

Entre los campos más relevantes:

| Campo | Descripción |
|---|---|
| member_id | Identificador único del cliente |
| member_since | Fecha de creación del cliente |
| birthday | Fecha de nacimiento |
| gender | Género |
| active | Estado activo/inactivo |
| timestamp_edit | Fecha de última modificación |

---

# 🧹 Transformación y Limpieza de Datos



En esta etapa se preparan los datos para el análisis.

## Técnicas aplicadas

- Eliminación de datos protegidos por RGPD.
- Eliminación de variables irrelevantes.
- Corrección de tipos de datos.
- Eliminación de duplicados.
- Manejo de valores nulos.
- Normalización y transformación de variables.

---

# 🧽 Limpieza Específica — CLUB MEMBERS

Las principales transformaciones realizadas fueron:

## Gestión de identificadores

- Se fija `member_id` como índice único.

Se eliminan campos sensibles por protección de datos como `firstname`, `lastname`, `email`, `phone` y `mobile`.

## Protección de datos (RGPD)

Se eliminan campos sensibles:

- firstname
- lastname
- email
- mobile
- phone

## Eliminación de campos irrelevantes

Se eliminan:

- club_id
- external_id
- is_pro
- active
- lang
- country
- rfid_tag
- early_booking_access
- user_id
- formatted_address

## Conversión de fechas

Se convierten correctamente:

- `birthday`
- `member_since`
- `timestamp_edit`

Los timestamps originales en milisegundos se transforman a formatos `date` y `datetime`.

La comparación entre `timestamp_edit_date` y `contract_end_date` permite identificar bajas de clientes.

## Gestión de bajas

A partir de `timestamp_edit` se generan:

- `timestamp_edit_date`
- `timestamp_edit_datetime`

La comparación entre `timestamp_edit_date` y `contract_end_date` permite identificar bajas.

## Filtrado de registros

- Se eliminan empleados de la tabla de clientes.
- Se eliminan clientes sin membresía asociada.

---

# Dashboard y visualización

El dashboard se desarrolló íntegramente en **Power BI**.

Dashboard operativo

que permitiera explotar visualmente toda la información obtenida en el proceso ETL y EDA.

## Principio de diseño

Las variables dinámicas y KPIs temporales se implementan como medidas DAX en Power BI en lugar de precalcularlas en la ETL.

Esto permite:

- reducir complejidad del pipeline,
- mejorar rendimiento,
- ejecutar cálculos bajo demanda,
- facilitar segmentación dinámica.

---

# 🧠 Preparación del Modelo en Power BI

## ETL en Power Query

La preparación en Power Query es mínima debido a que la limpieza principal ya se realiza en Python.

Se importan principalmente:

- Clients
- Visits
- Visits_Minute_detail

---

# 🗓️ Modelado Temporal

Se crean tablas temporales para análisis evolutivos:

## Calendario

Tabla diaria para KPIs por fecha.

## CalendarioDateTimeMin

Tabla a nivel minuto para:

- concurrencia,
- ocupación,
- análisis horario.

## Cohortes

Se crean:

- Matriz_Altas_Cohortes
- Matriz_Bajas_Cohortes

Estas tablas permiten analizar:

- retención,
- antigüedad,
- comportamiento temporal.

---

# 🔗 Relaciones del Modelo

Se crean relaciones entre:

- Clients
- Visits
- Visits_Minute_detail
- Calendario
- CalendarioDateTimeMin
- tablas de cohortes

utilizando principalmente `member_id` y claves temporales.

---

# 📌 KPIs y Métricas Analizadas

Entre las métricas desarrolladas:

- Número de clientes activos.
- Altas y bajas.
- Antigüedad media.
- Edad media.
- Días desde última visita.
- Ratio de visitas.
- Horarios de máxima ocupación.
- Cohortes de retención.
- Distribución de membresías.

---

# 🗂️ Estructura del Proyecto

```bash
.
├── data/
├── notebooks/
│   └── etl_proyecto_final_python.ipynb
├── dashboards/
├── .env
├── requirements.txt
└── README.md
```

---

# ⚙️ Instalación y Requisitos

## Clonar el repositorio

```bash
git clone https://github.com/usuario/repositorio.git
cd repositorio
```

## Crear entorno virtual

```bash
python -m venv venv
```

### Windows

```bash
venv\Scripts\activate
```

### Linux / Mac

```bash
source venv/bin/activate
```

## Instalar dependencias

```bash
pip install -r requirements.txt
```

## Configurar variables de entorno

```env
CLUB_SECRET=tu_club_secret
API_KEY=tu_api_key
```

## Ejecutar notebook

```bash
jupyter notebook
```

---

# 📊 Resultados y Conclusiones

A partir del EDA y del dashboard desarrollado en Power BI se consiguió construir una visión analítica completa sobre:

- comportamiento histórico de clientes,
- evolución de altas y bajas,
- patrones de asistencia,
- análisis temporal,
- ocupación y concurrencia,
- segmentación de clientes,
- métricas operativas del gimnasio.

La combinación entre ETL en Python y modelado analítico en Power BI permitió separar correctamente:

- procesamiento de datos,
- lógica analítica,
- visualización,
- cálculos dinámicos mediante DAX.


El proyecto permitió obtener una visión completa del comportamiento de clientes y operaciones del gimnasio.

Entre los principales resultados y aprendizajes destacan:


- consumo de APIs,
- automatización ETL,
- limpieza de datos,
- modelado analítico,
- análisis exploratorio,
- métricas de negocio,
- Power BI,
- DAX,
- cohortes y retención,
- dashboards operativos.

---

# 🔄 Próximos Pasos

- Automatización completa del pipeline.
- Orquestación ETL.
- Integración con base de datos.
- Dashboard en tiempo real.
- Modelos predictivos de churn.
- Segmentación avanzada.
- Forecasting de ocupación.

---

# ✒️ Autor

Antonio Fernandez

Proyecto Final — Análisis de Datos, ETL y Business Intelligence

GitHub: https://github.com/afborondo

