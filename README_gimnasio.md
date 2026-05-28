# 🏋️ Proyecto Final — EDA, ETL y Dashboard de un Gimnasio con Virtuagym
## 📖 Descripción del Proyecto
Este proyecto corresponde al Proyecto Final del programa de análisis de datos.
En este proyecto se aplican y desarrollan los conocimientos adquiridos a lo largo del programa.
El proyecto se centra en el desarrollo completo de un caso real de analítica de negocio aplicado a un gimnasio.
Transformación y limpieza profunda de los datos.
Análisis descriptivo de los datos.
Análisis estadístico de los datos.
Visualización de los datos.
Dashboard operativo.
Informe explicativo del análisis.
A lo largo del proyecto se desarrollan procesos de:
extracción de datos,
transformación y limpieza,
análisis exploratorio (EDA),
análisis estadístico,
visualización de datos,
construcción de dashboard operativo.

# 🎯 Objetivo del Proyecto
En un negocio de gimnasio es fundamental entender los datos históricos (analítica descriptiva) de clientes, altas y bajas y visitas al gimnasio.
El proyecto tiene como objetivo desarrollar una solución completa de análisis de datos que permita comprender el comportamiento histórico de clientes y actividad operativa del gimnasio.
## Clientes
Evolución temporal.
Caracterización de clientes.
Tipología de cuotas.
Segmentación.
## Altas y bajas
Evolución temporal.
Caracterización.
Análisis de retención.
Comportamiento de abandono.
## Visitas al gimnasio
Ratios de visitas por día y horario.
Frecuencia de asistencia.
Caracterización según tipo de cliente.
Concurrencia y ocupación.

# 🛠️ Herramientas Utilizadas
## ETL y análisis de datos
Python
Pandas
Requests
python-dotenv
Visual Studio Code
## Dashboard y visualización
Power BI

# 🗂️ 1.1 EDA. Comprensión general de los datos. Carga de los datos.
Los datos se encuentran un CRM, “Virtuagym”.
Se solicita el acceso API al soporte de CRM, y este envía la documentación.
Es una API que requiere dos credenciales, “club_secret”, y “api_key”. Se obtienen.
En la documentación de la API, se identifican los siguientes Datasets de entre todos los disponibles, que tienen la información relevante:

# ⚙️ Arquitectura ETL
La extracción de datos se realiza mediante peticiones HTTP utilizando la librería requests.
El acceso y descarga de los datos se realiza con Python utilizando la librería requests, mientras que la transformación y limpieza se desarrolla con pandas y estructuras DataFrame.
Cada dataset dispone de endpoints, métodos, parámetros y sistemas de paginación específicos. Además, el API posee limitaciones de consultas máximas por hora y por día.
## Gestión del API
Se implementó un objeto encargado de:
controlar límites de peticiones,
gestionar solicitudes,
manejar paginación,
centralizar autenticación.
## Conversión a DataFrames
Se desarrolló un segundo objeto para:
descargar datasets,
convertir respuestas JSON en DataFrames,
automatizar procesos de carga.

# 🔄 Actualización de Datos
Se implementan dos tipos de actualización de datos:
carga completa (full refresh)
carga incremental (incremental refresh)
según las capacidades del API.
Todos los datasets permiten actualización incremental excepto:
Membership Instances
ya que no dispone de campo timestamp.
Para soportar ambos sistemas se crean metadatos asociados a cada dataset:
tipo de actualización,
timestamp de última sincronización,
cursor de paginación.
El almacenanamiento de las actualizaciones de cada Dataset se realiza en dos ficheros:
uno general, que almacena los metadatos del Dataset en cabecera (#), y después una línea por cada dato, anadiéndose para cada dato si se recopiló por un backup full o incremental, así como el timestamp del api al actualizarse. Este general permite trazar todo el proceso de acceso y actualización en el API e incluso permitiría trazar variaciones históricas en algunos datos.
un snapshot, que es sólo la versión de los datos más reciente, para después ser analizado y transformado.
El proceso de acceso y descarga termina generando 5 dataframes, que son los snapshots de los Datasets:
df_members (Club Members)
df_employees (Club Employees)
df_memberships_definition (Membership Definition)
df_memberships (Membership Instances)
df_visits (Club Visits)

# 📊 1.2 EDA: Comprensión general de los datos. Vista rápida. Tipos de datos. Estadística descriptiva. Resumen estadístico.
En esta fase del proyecto consiste en comprender la estructura y calidad de los datos.
## Técnicas utilizadas
Carga inicial de datos.
Revisión de tipos de datos.
Estadística descriptiva.
Resumen estadístico.
Identificación de valores nulos.
## Métodos principales de pandas
# Primeras filas
df.head()

# Información del dataset
df.info()

# Estadísticos descriptivos
df.describe()
Procedemos con los todos los datasets.
## CLUB MEMBERS
Se compone de los siguientes campos, según el API:
Se realiza una vista general con df.head(), df.info(), df.describe().
Las conclusiones son:
Hay varios campos que no aportan ninguna información porque tienen el mismo valor (country, lang, early_access).
Las columnas son iguales a las de la API salvo que external_id es igual a orginal_member_id.
Los tipos de datos en la API string en el dataframe aparecen como object.
Será necesario en fases posteriores:
fijar el índice al member_id,
eliminar campos por protección de datos (firstname, lastname, email, phone, mobile),
eliminar campos no relevantes (club_id, external_id, is_pro, lang, country, rfid_tag, early_booking_access, user_id, formatted_address),
convertir datos a string,
convertir datos a fechas (date y datetime).

# 📊 2. EDA: Transformación y limpieza
En esta etapa, se preparan y mejoran los datos para que sean más fáciles de analizar.
La transformación y limpieza incluyen:
detección y corrección de errores,
normalización,
estandarización,
transformación de datos.
## Técnicas
Eliminación de datos protegidos por RGPD y datos irrelevantes para el análisis.
Corrección de tipos de datos.
Eliminación de datos duplicados.
Manejo de valores nulos.
Normalización y estandarización.
Transformaciones logarítmicas o de potencias.
## CLUB MEMBERS
Las tareas de transformación y limpieza son:
fijar el índice al “member_id” que es la id único de los clientes
eliminar campos por protección de datos (firstname, lastname, email, mobile, phone)
eliminar campos no relevantes (club_id, external_id, is_pro, active, lang, country, rfid_tag, early_booking_access, user_id, formatted_address)
se convierte a tipo date las fechas (birthday está como string, membership_since y timestamp_edit están en milisegundos)
timestamp_edit (tiempo en que se actualiza el registro), se elimina pero se crean dos campos, uno con timestamp_edit_date, y otro como datetime (timestamp_edit_datetime): timestamp_edit_date si es igual contract_end_date se trata de un baja, y timestamp_edit_datetime su valor máximo determina la fecha y hora de la última modificación
se eliminan los empleados de la tabla de miembros
se eliminan los miembros que no tienen una membresía asociada (clientes no activos cuando se hizo la carga inicial del CRM en el año 2023)
## MEMBERSHIP DEFINITION
Las tareas de transformación y limpieza son:
se fija el índice a membership_id
se eliminan todos los campos salvo membership_id, membership_name, membership_price
Al mantenerse sólo el membership_price los ingresos calculados como suma de membership_price indicarán la “base de ingresos” proforma, pero no la facturación real ya que esa implicaría hacer prórrata de la cuota y sumar también la matrícula.
La facturación no es el objetivo del análisis.
se cambia el tipo de datos de datos membership_name (tipo object) a string y el tipo de membership_price de tipo int a tipo float
## MEMBERSHIP INSTANCES
Las tareas de transformación y limpieza son:
fijar el índice a instance_id
los clientes sólo tienen una membresía al mismo tiempo (el gimnasio es mono producto), pero las membresías han cambiado durante el tiempo; por ello, se elige para cada cliente únicamente la membresía contract_end_date más reciente
se seleccionan únicamente las columnas relevantes para el análisis: member_id, membership_id, contract_end_date
se convierte el tipo de datos contract_end_date de tipo string a tipo date
se indexa por member_id
## CLUB VISITS
Las tareas de transformación y limpieza son:
se fija como índice el campo ‘id’
se seleccionan los campos relevantes (id, member_id, check_in_timestamp, check_out_timestamp, status)
se consideran sólo visitas asociadas a los miembros, ya que es lo que se quiere caracterizar (no las de empleados)
se consideran las visitas con status ok (las que se corresponden con el evento de entrada o salida con éxito, sino sería intentos de entrada/salida sería parte de otro análisis)
se convierte check_in_timestamp en formato date para que sirva como eje temporal
se convierte check_in_timestamp en formato datetime
se convierte check_out_timestamp en formato datetime
se eliminan los días que se cerró el gimnasio
se crea una columna adicional que es la duración de la visita en minutos, tendrá valor vacío si no existe check_out_timestamp
se eliminan las visitas de duración menores a 5 minutos, las de menor duración se entienden que son anómalas
se deduplican accesos que han ocurrido en menos de 10 min (umbral para deduplicar), se elige el último que se entiende que ha sido el exitoso
se elimina la última visita de cada baja al gimnasio, tanto en la tabla de visitas como en la df_visits_minute_details que suele ser la visita presencial para comunicarla la baja, ya que no refleja un acceso real al gimnasio
una vez depuradas las visitas, es crítico calcular la concurrencia al centro de en cada minuto de cada visita de cada cliente, puesto que esto determina su experiencia.
Para ello:
- es necesario saber cuándo acaba y termina una visita, como muchas visitas no tienen checkou, se estima un check_out aún cuando cliente no lo haya hecho. Para ello se estima la duración de la vista en función del promecio de las visitas de los últimos 30 días naturales anteriores de ese cliente y si no se toma un promedio del centro que son 75 min. Esa estimación es auxiliar sólo para calcular el check_out ajustado, no es que la reemplace cuando sea cero, para no contaminar otros análisis.

- se realiza un dataframe auxiliar (df_datetimemin_visits) con todos dos datetimemin de apertura del centro y que contengan el número de visitas concurrentes en cada minuto; de esta forma el cálculo la concurrencia en cada minuto se realiza un única vez y no una por cada visita por cada miembro

- a partir del dataframe anterior, se realiza un datafrema auxiliar que para id de visita (df_visits_minute_detail), id de cliente, minuto de la visita, determina la concurrencia (con cuántos clientes coincidió incluyendo él)

# 🔗 CONSOLIDACIÓN DE DATASETS
Para mayor facilidad en en análisis, se consolida la información en dos datasets, para que cada uno contenga toda la información
Clientes (df_clients): dataset indexado por member_id que consolida por member_id, la información de los datasets: Club Members, Membership Definition, Membership Instances
Visitas (df_visits): dataset indexado por id (visita) que consolida la información de los dataset de Clientes: fecha de nacimiento, sexo, fecha de alta, fecha de baja, membresía
Detalle de al visita (df_visits_minute_detail), que contiene la concurrencia por minuto de cada visita de cada miembro

# 📁 GENERACIÓN DE FICHEROS DE DATOS
Se generan tres ficheros de datos csv, con los dataset de clientes (“Clients.csv”), visitas (“Visits.csv”), y detalle de visitas (“VisitsMinute_detail.csv”). Estos serían el origen de datos de PowerBI.

## 📈 Dashboard y Visualización
El dashboard se ha desarrollado íntegramente en Power BI.
Dashboard operativo orientado al análisis de:
comportamiento histórico de clientes,
evolución de altas y bajas,
visitas y concurrencia,
ocupación,
segmentación,
métricas operativas.
## Principio de diseño
Como principio general, se realizan como medidas de PowerBI todas las variables que son calculadas y que varían en el tiempo (p.ej. número clientes activos, edad, antigüedad de un cliente, días desde la última visita etc ), incluso su categorización -en su caso- con segmentación dinámica (pej. segmentos de edad o permanencia).
Las razones son:
las medidas de PoweBI permiten calcular de forma eficiente variables dinámicas (ejecución bajo demanda, tablas temporales incluídas en el propio código DAX de la medida)
se reduce la complejidad de la ETL: sólo se cargan Datasets con datos estáticos, no tablas con ejes temporales y valor de indicadores en éstos ejes

# 🧠 Preparación del Modelo en Power BI

En la preparación del Dashboard en PowerBI se han seguido estas fases:
ETL para importación y carga a través de PowerQuery; proceso muy sencillo ya que la limpieza y transformación está ya realizada en python en fases anteriores. Sólo carga del csv- encabezados promovidos, de las tablas “Clients”, “Visits”, “Visits_Minute_detail”
Creación de dos tablas de eje temporal para poder realizar evolutivos: una de días “Calendario” para cálculos de kpis día, otra de minutos “CalendarioDateTimeMin” para poder hacer cálculos de KPIs-minutos (concurrencia), dos tablas para hacer el cálculos de cohortes (“Matriz_Altas_Cohortes”, “Matriz_Bajas_Cohortes) que agrupan las fechas de la tabla Calendario por meses y las combinan con meses de antigüedad. Estas tablas además tienen la etiqueta del mes y día en formato numérico o texto y tienen un columna orden.
activación de relaciones entre las tablas de datos: Clients, Visits, Visist_Minute_detail entre sí (relación a través de member_id), y de estas tablas con las tablas de tiempo Calendario, CalendarioDateTimeMin, Matriz_Altas_Cohortes y Matriz_Bajas_Cohortes
estructuración del dashboard y medidas por página, agrupando medidas calculadas en tablas para mejor acceso (pej las métricas de Clientes por un lado, las de Visitas por otro etc)
uso de la herramienta externa “Tabular Editor”, para hacer los comparativos de un valor con respecto a referencias pasadas con Calculation Items (por ejemplo, comparativa con el mes anterior, año anterior etc). De esta forma, se realiza un Calculation Item por cada comparativo y no medida calculada para cada métrica y comparativo.

# 📌 Dashboard: KPIs y Métricas Analizadas
El Dashboard de PowerBI se ha estructurado con estas páginas:
“Resumen ejecutivo”: permite conocer de un vistazo el desempeño del negocio. Dos franjas. Primera con tarjetas de KPIs actuales en grande y comparativos abajo más pequeño: Número de Clientes, ARPU, Permanencia y MRR (debajo comparación con el 1 de mes, el mes pasado, y ese mes el año pasado al ser un negocio estacional, comparación tanto en valor absulto como relativo), y Churn (debajo comparación con el valor mes -1, mes -2, mes - 3). Segunda franja dos evolutivos para ver tendencias: 1 evolutivo mensual dos últimos años de Clientes a 1ro de mes, Altas, Bajas.
“Clientes. Caracterización”: datos demográficos. Dos franjas. Primera con tarjetas de KPIs actuales en grande y debajo la distrubución de segmentos: Edad Media, Sexo, Membresías Familiares, Mismo CP (si su domicilio tiene el mismo CP que el centro), Permanencia. Segunda franja: dos evolutivos para ver tendencias: 1 evolutivo mensual dos últimos años de segmentos de edad y Edad Media. 2. evolutivo mensual dos últimos años de %Sexo femenino, %Mismo CP, Clientes con permanencia de más de 3 años tanto en valor absoluto como relativo
“Clientes.Visitas.” caracterización de las visitas desde la óptica de cliente, no del centro. Dos franjas. Primer franja, tarjetas de KPIs actuales en grande y  y comparativos abajo más pequeño. Los KPIs son: Clientes Activos (los que han visitado el centro en el último mes), Visitas mes por activo

# 🗂️ Estructura del Proyecto
.
├── data/
├── notebooks/
│   └── etl_proyecto_final_python.ipynb
├── dashboards/
├── .env
├── requirements.txt
└── README.md

# ⚙️ Instalación y Requisitos
## Clonar el repositorio
git clone https://github.com/usuario/repositorio.git
cd repositorio
## Crear entorno virtual
python -m venv venv
### Windows
venv\Scripts\activate
### Linux / Mac
source venv/bin/activate
## Instalar dependencias
pip install -r requirements.txt
## Configurar variables de entorno
CLUB_SECRET=tu_club_secret
API_KEY=tu_api_key
## Ejecutar notebook
jupyter notebook

# 📊 Resultados y Conclusiones
El proyecto permitió obtener una visión completa del comportamiento de clientes y operaciones del gimnasio.
Entre los principales resultados destacan:
análisis histórico de clientes,
evolución temporal de altas y bajas,
patrones de asistencia,
análisis de ocupación,
cohortes de retención,
análisis operativo del gimnasio.
La combinación entre ETL en Python y modelado analítico en Power BI permitió separar correctamente:
procesamiento de datos,
lógica analítica,
visualización,
cálculos dinámicos mediante DAX.

# 🔄 Próximos Pasos
Automatización completa del pipeline.
Orquestación ETL.
Integración con base de datos.
Dashboard en tiempo real.
Modelos predictivos de churn.
Segmentación avanzada.
Forecasting de ocupación.

# ✒️ Autor
Antonio Fernandez
Proyecto Final — Análisis de Datos, ETL y Business Intelligence
GitHub: https://github.com/afborondo

| Dataset | Descripción |
| --- | --- |
| Club Members | Clientes del gimnasio |
| Club Employees | Empleados del gimnasio |
| Membership Definition | Definición de membresías |
| Membership Instances | Instancias de membresías |
| Club Visits | Registros de acceso y visitas |

| Field | Type | Optional | Values | Information |
| --- | --- | --- | --- | --- |
| member_id | int | no |  | The ID for the member |
| club_id | int | no |  | The ID of the club |
| club_member_id | string | yes |  | Custom ID from external system |
| external_id | string | yes |  | External member ID |
| firstname | string | no |  | First name |
| lastname | string | no |  | Last name |
| email | string | no |  | Email address |
| active | boolean | no |  | Active/inactive |
| is_pro | boolean | no |  | Pro member |
| gender | string | yes | m/f | Gender |
| member_since | int | no |  | Timestamp account creation |
| timestamp_edit | int | no |  | Timestamp modification |
| birthday | string | yes |  | Birthday |
| lang | string | yes |  | Language |
| zip | string | yes |  | Zip code |
| street | string | yes |  | Street |
| street_extra | string | yes |  | Extra address info |
| place | string | yes |  | City |
| country | string | yes |  | Country |
| formatted_address | string | yes |  | Formatted address |
| phone | string | yes |  | Phone |
| mobile | string | yes |  | Mobile |
| rfid_tag | string | yes |  | RFID tag |
| early_booking_access | boolean | yes |  | Early booking access |
