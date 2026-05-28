# 🏋️ Proyecto Final — EDA, ETL y Dashboard de un Gimnasio con Virtuagym
🏋️ Proyecto Final — EDA, ETL y Dashboard de un Gimnasio con Virtuagym
📖 Descripción del Proyecto
Este proyecto corresponde al Proyecto Final del programa de análisis de datos.
En este proyecto se aplican y desarrollan los conocimientos adquiridos a lo largo del programa.
El proyecto se centra en el desarrollo completo de un caso real de analítica de negocio aplicado a un gimnasio.
•	Transformación y limpieza profunda de los datos.
•	Análisis descriptivo de los datos.
•	Análisis estadístico de los datos.
•	Visualización de los datos.
•	Dashboard operativo.
•	Informe explicativo del análisis.
A lo largo del proyecto se desarrollan procesos de:
•	extracción de datos,
•	transformación y limpieza,
•	análisis exploratorio (EDA),
•	análisis estadístico,
•	visualización de datos,
•	construcción de dashboard operativo.
________________________________________
🎯 Objetivo del Proyecto
En un negocio de gimnasio es fundamental entender los datos históricos (analítica descriptiva) de clientes, altas y bajas y visitas al gimnasio.
El proyecto tiene como objetivo desarrollar una solución completa de análisis de datos que permita comprender el comportamiento histórico de clientes y actividad operativa del gimnasio.
Clientes
•	Evolución temporal.
•	Caracterización de clientes.
•	Tipología de cuotas.
•	Segmentación.
Altas y bajas
•	Evolución temporal.
•	Caracterización.
•	Análisis de retención.
•	Comportamiento de abandono.
Visitas al gimnasio
•	Ratios de visitas por día y horario.
•	Frecuencia de asistencia.
•	Caracterización según tipo de cliente.
•	Concurrencia y ocupación.
________________________________________
🛠️ Herramientas Utilizadas
ETL y análisis de datos
•	Python
•	Pandas
•	Requests
•	python-dotenv
•	Visual Studio Code
Dashboard y visualización
•	Power BI
________________________________________
🗂️ 1.1 EDA. Comprensión general de los datos. Carga de los datos.
Los datos se encuentran un CRM, “Virtuagym”.
Se solicita el acceso API al soporte de CRM, y este envía la documentación.
Es una API que requiere dos credenciales, “club_secret”, y “api_key”. Se obtienen.
En la documentación de la API, se identifican los siguientes Datasets de entre todos los disponibles, que tienen la información relevante:
Dataset	Descripción
Club Members	Clientes del gimnasio
Club Employees	Empleados del gimnasio
Membership Definition	Definición de membresías
Membership Instances	Instancias de membresías
Club Visits	Registros de acceso y visitas
________________________________________
⚙️ Arquitectura ETL
La extracción de datos se realiza mediante peticiones HTTP utilizando la librería requests.
El acceso y descarga de los datos se realiza con Python utilizando la librería requests, mientras que la transformación y limpieza se desarrolla con pandas y estructuras DataFrame.
Cada dataset dispone de endpoints, métodos, parámetros y sistemas de paginación específicos. Además, el API posee limitaciones de consultas máximas por hora y por día.
Gestión del API
Se implementó un objeto encargado de:
•	controlar límites de peticiones,
•	gestionar solicitudes,
•	manejar paginación,
•	centralizar autenticación.
Conversión a DataFrames
Se desarrolló un segundo objeto para:
•	descargar datasets,
•	convertir respuestas JSON en DataFrames,
•	automatizar procesos de carga.
________________________________________
🔄 Actualización de Datos
Se implementan dos tipos de actualización de datos:
•	carga completa (full refresh)
•	carga incremental (incremental refresh)
según las capacidades del API.
Todos los datasets permiten actualización incremental excepto:
•	Membership Instances
ya que no dispone de campo timestamp.
Para soportar ambos sistemas se crean metadatos asociados a cada dataset:
•	tipo de actualización,
•	timestamp de última sincronización,
•	cursor de paginación.
El almacenanamiento de las actualizaciones de cada Dataset se realiza en dos ficheros:
•	uno general, que almacena los metadatos del Dataset en cabecera (#), y después una línea por cada dato, anadiéndose para cada dato si se recopiló por un backup full o incremental, así como el timestamp del api al actualizarse. Este general permite trazar todo el proceso de acceso y actualización en el API e incluso permitiría trazar variaciones históricas en algunos datos.
•	un snapshot, que es sólo la versión de los datos más reciente, para después ser analizado y transformado.
El proceso de acceso y descarga termina generando 5 dataframes, que son los snapshots de los Datasets:
•	df_members (Club Members)
•	df_employees (Club Employees)
•	df_memberships_definition (Membership Definition)
•	df_memberships (Membership Instances)
•	df_visits (Club Visits)
________________________________________
📊 1.2 EDA: Comprensión general de los datos. Vista rápida. Tipos de datos. Estadística descriptiva. Resumen estadístico.
En esta fase del proyecto consiste en comprender la estructura y calidad de los datos.
Técnicas utilizadas
•	Carga inicial de datos.
•	Revisión de tipos de datos.
•	Estadística descriptiva.
•	Resumen estadístico.
•	Identificación de valores nulos.
Métodos principales de pandas
# Primeras filas
df.head()

# Información del dataset
df.info()

# Estadísticos descriptivos
df.describe()
Procedemos con los todos los datasets.
CLUB MEMBERS
Se compone de los siguientes campos, según el API:
Field	Type	Optional	Values	Information
member_id	int	no		The ID for the member
club_id	int	no		The ID of the club
club_member_id	string	yes		Custom ID from external system
external_id	string	yes		External member ID
firstname	string	no		First name
lastname	string	no		Last name
email	string	no		Email address
active	boolean	no		Active/inactive
is_pro	boolean	no		Pro member
gender	string	yes	m/f	Gender
member_since	int	no		Timestamp account creation
timestamp_edit	int	no		Timestamp modification
birthday	string	yes		Birthday
lang	string	yes		Language
zip	string	yes		Zip code
street	string	yes		Street
street_extra	string	yes		Extra address info
place	string	yes		City
country	string	yes		Country
formatted_address	string	yes		Formatted address
phone	string	yes		Phone
mobile	string	yes		Mobile
rfid_tag	string	yes		RFID tag
early_booking_access	boolean	yes		Early booking access
Se realiza una vista general con df.head(), df.info(), df.describe().
Las conclusiones son:
•	Hay varios campos que no aportan ninguna información porque tienen el mismo valor (country, lang, early_access).
•	Las columnas son iguales a las de la API salvo que external_id es igual a orginal_member_id.
•	Los tipos de datos en la API string en el dataframe aparecen como object.
Será necesario en fases posteriores:
1.	fijar el índice al member_id,
2.	eliminar campos por protección de datos (firstname, lastname, email, phone, mobile),
3.	eliminar campos no relevantes (club_id, external_id, is_pro, lang, country, rfid_tag, early_booking_access, user_id, formatted_address),
4.	convertir datos a string,
5.	convertir datos a fechas (date y datetime).

CLUB EMPLOYEES
Los empleados son miembros. El dataset “Club Employees” identifica a los empleados del centro, para poderlo eliminarlo del análisis ya que lo puede distorsionar puesto que no son clientes.
Se compone de los siguientes campos, según el API:
Field	Type	Optional	Values	Information
1. member_id	int	no		The ID for the employee (“Member ID” in Virtuagym)
2. club_id	int	no		The ID of the club the user belongs to in Virtuagym (in case of a chain this is a sub-club)
3. club_member_id	string	yes		Custom ID from the external system (“Own member ID” in Virtuagym)
4. external_id	string	yes		ID from the external system
5. firstname	string	no		The first name of the employee
6. lastname	string	no		Last name of the employee
7. email	string	no		The email address of the employee, an invitation will be sent to this address when the employee will be created
8. active	boolean	no		If the employee is active or inactive
9. is_pro	boolean	no		If the employee is pro
10. gender	string	yes	“m”/“f”	The gender of the editted employee
11. member_since	int	no		The timestamp (in milliseconds) the employee made an account (yyyy-mm-dd)
12. timestamp_edit	int	no		The timestamp (in milliseconds) the employee’ information changed
13. birthday	string	yes		The birthday of the employee (YYYY-MM-DD)
14. lang	string	yes		The language the employee uses in the portal
15. zip	string	yes		The zip code of the employee
16. street	string	yes		The street address of the employee
17. street_extra	string	yes		Extra street information of the employee
18. place	string	yes		The place where the employee lives
19. country	string	yes		The country code where the employee lives
20. formatted_address	string	yes		A nicely formatted string of the street address of the employee
21. phone	string	yes		The phone number of the employee
22. mobile	string	yes		The mobile phone number of the employee
23. rfid_tag	string	yes		Rf-ID tag that is tied to the employee
Se realiza un vista general con df.info().
Esta tabla es sólo para identificar a los empleados y eliminarlos del análisis de miembros.
En efecto a través del campo member_id, se pueden identificar los empleados.
Las conclusiones son:
•	Se verifican los empleados, personal del gimnasio y también personal de mantemiento.
________________________________________
MEMBERSHIP DEFINITION
Este dataset es importante porque contiene la definición de membresía; de aquí se puede extraer la precio de cada membresía y su nombre.
Se compone de los siguientes campos, según el API:
Field	Type	Values	Description
1. membership_id	integer	-	Unique identifier for the membership
2. membership_name	string	-	Name of the membership
3. membership_group	string	-	Group the membership belongs to
4. membership_notes	string	-	Additional notes for the membership
5. membership_availability_start	date	-	Start date of membership availability
6. membership_availability_end	date	-	End date of membership availability
7. membership_available_online	boolean	true, false	Whether the membership is available online
8. membership_duration	integer	-	Duration of the membership
9. membership_duration_type	string	weeks, months	Type of duration
10. membership_auto_renew	boolean	true, false	Whether the membership auto-renews
11. membership_pro_rata_start	boolean	true, false	Whether pro-rata billing is applied at the start
12. membership_renew_duration	integer	-	Duration for renewal
13. membership_renew_term	string	weeks, months	Term for renewal duration
14. membership_renew_before	integer	-	Days before renewal
15. membership_renew_before_term	string	days, weeks, months	Term for renewal before
16. membership_renew_price	decimal	-	Price for renewal
17. membership_price	decimal	-	Price of the membership
18. membership_price_term	string	total, monthly, weekly, four_weekly	Term for the price
19. membership_income_category	string	-	Income category for the membership
20. membership_registration_fee	decimal	-	Registration fee for the membership
21. membership_club_tax	object	-	Tax information for the membership. (see below)
22. membership_billing_cycle	string	-	Billing cycle for the membership
22. default_payment_method	string	-	Default payment method for the membership
23. tmp_default_payment_method	string	-	Temporary default payment method for the membership
24. membership_creation_date	date	-	Date the membership was created
25. membership_last_edit_date	date	-	Date the membership was last edited
26. membership_invoice_creation_term	string	-	Term for invoice creation
27. access_times	array	-	Access times for the membership. (see below)
Se realiza un vista general con df.head(), df.info(), df.describe().
Las conclusiones son:
6.	Se identifican los distintos tipos de membresía.
7.	Hay que cambiar el tipo de membership_name (tipo object) a string.
8.	Hay que eliminar todos los campos salvo:
o	membership_id
o	membership_name
o	membership_price
o	membership_registration_fee
________________________________________
MEMBERSHIP INSTANCES
Este dataset es importante porque contiene la asociación de miembro a tipo de membresía, la fecha de alta, de baja si existe por cada tipo, y de ahí se puede determinar también si se está activo o no.
Se compone de los siguientes campos, según el API:
Field	Type	Optional	Values	Information
1. instance_id	int	no		The id of the membership instance
2. member_id	int	no		The id of the [[Club Members]]
3. membership_id	int	no		The id of the [Membership Definition]

4. active	boolean	no		If the membership instance is active or inactive
5. cancelled	boolean	no		If the membership was manually cancelled by an employee with termination in the future
6. contract_autorenewed	boolean	no		If the membership was already renewed at this point or it’s still in its initial period
7. completed	boolean	no		If the membership has reached the contract end date automatically and was not renewed
8. paused	boolean	no		If the membership instance was paused by an employee
9. stopped	boolean	no		If the membership was manually cancelled by an employee with immediate termination
10. start_date	string	no		The actual start date of the membership (yyyy-mm-dd)
11. contract_start_date	string	no		The contractual start date of the membership (yyyy-mm-dd)
12. contract_end_date	string	no		The contractual end date of the membership (yyyy-mm-dd)
13. membership_name	string	no		The name of the membership
Se realiza un vista general con df.head(), df.info(), df.describe().
Las conclusiones son:
9.	Se identifican los miembros con membresía asociada con su membresía (nombres únicos).
10.	Hay que cambiar los tipos de datos de las fechas (tipo object) a datetime.
11.	Hay que quitar muchos campos que no se emplearán como:
o	membership_id
o	active
o	cancelled
o	contract_autorenewed
o	completed
o	paused
o	stopped
o	start_time
________________________________________
CLUB VISITIS
Este dataset es importante porque contiene la asistencia de los clientes al gimnasio.
Sería posible entender patrones de visitas de horas-días, frecuencias, duraciones, diferencias en asistenca por sexo -edad, patrones de baja asociados a la frecuencia de visita, horarios, etc.
Además, otra cuestión es detectar posibles accesos no permitidos (pej múltiples check-in sin check-out, acceso con diferentes dispositivos).
Se compone de los siguientes campos, según el API:
Field	Type	Values	Information
1. id	int		The unique identifier of the visit
2. club_id	int		The id of the corresponding club
3. member_id	int		The id of the member who triggered the visit
4. check_in_timestamp	int		The timestamp (in ms) of the check-in
5. check_out_timestamp	int		The timestamp (in ms) of the check-out
6. status	string	ok, warning, rejected	A human readable status of the visit
7. status_message	string		Supplementary text of the visit
Se realiza un vista general con df.head(), df.info(), df.describe().
Como conclusiones son:
12.	Hay un campo adicional, device_id, dispositivo con el que se accede que podría explotarse para identifacar accesos indebidos (se facilita el usuario de la aplicación a otra persona por ejemplo).
13.	Se pueden eliminar los campos status, status_id; es la membresía que es un dato que no aporta, se puede eliminar.
14.	Las fechas están como número entero, hay que convertirlas a datetime.


________________________________________
📊 2. EDA: Transformación y limpieza
En esta etapa, se preparan y mejoran los datos para que sean más fáciles de analizar. La transformación y limpieza incluyen la detección y corrección de errores, la normalización o estandarización de variables, y la transformación de datos cuando sea necesario.
Técnicas
•	Eliminación de datos protegidos por RGPD y datos irrelevantes para el análisis.
•	Corrección de tipos de datos: convertir las variables mal etiquetadas (por ejemplo, transformar variables categóricas en variables de tipo adecuado).
•	Eliminación de datos duplicados.
•	Manejo de valores nulos: determinar si es necesario eliminar, imputar o dejar sin cambios los valores faltantes.
•	Normalización y estandarización: ajustar los datos para que tengan una escala consistente, lo que es útil para ciertas técnicas de análisis.
•	Transformaciones: aplicar transformaciones logarítmicas o de potencias para corregir distribuciones sesgadas.
________________________________________
CLUB MEMBERS
Las tareas de transformación y limpieza son:
•	fijar el índice al member_id que es la id único de los clientes
•	eliminar campos por protección de datos (firstname, lastname, email, mobile, phone)
•	eliminar campos no relevantes (club_id, external_id, is_pro, active, lang, country, rfid_tag, early_booking_access, user_id, formatted_address)
•	se convierte a tipo date las fechas (birthday está como string, membership_since y timestamp_edit están en milisegundos)
•	timestamp_edit (tiempo en que se actualiza el registro), se elimina pero se crean dos campos, uno con timestamp_edit_date, y otro como datetime (timestamp_edit_datetime): timestamp_edit_date si es igual contract_end_date se trata de un baja, y timestamp_edit_datetime su valor máximo determina la fecha y hora de la última modificación
•	se eliminan los empleados de la tabla de miembros
•	se eliminan los miembros que no tienen una membresía asociada (clientes no activos cuando se hizo la carga inicial del CRM en el año 2023)
________________________________________
MEMBERSHIP DEFINITION
Las tareas de transformación y limpieza son:
•	se fija el índice a membership_id
•	se eliminan todos los campos salvo membership_id, membership_name, membership_price
Al mantenerse sólo el membership_price los ingresos calculados como suma de membership_price indicarán la “base de ingresos” proforma, pero no la facturación real ya que esa implicaría hacer prorrata de la cuota y sumar también la matrícula.
La facturación no es el objetivo del análisis.
•	se cambia el tipo de datos de membership_name (tipo object) a string y el tipo de membership_price de tipo int a tipo float
________________________________________
MEMBERSHIP INSTANCES
Las tareas de transformación y limpieza son:
•	fijar el índice a instance_id
•	los clientes sólo tienen una membresía al mismo tiempo (el gimnasio es mono producto), pero las membresías han cambiado durante el tiempo; por ello, se elige para cada cliente únicamente la membresía contract_end_date más reciente
•	se seleccionan únicamente las columnas relevantes para el análisis: member_id, membership_id, contract_end_date
•	se convierte el tipo de datos contract_end_date de tipo string a tipo date
•	se indexa por member_id
________________________________________
CLUB VISITS
Las tareas de transformación y limpieza son:
•	se fija como índice el campo id
•	se seleccionan los campos relevantes (id, member_id, check_in_timestamp, check_out_timestamp, status)
•	se consideran sólo visitas asociadas a los miembros, ya que es lo que se quiere caracterizar (no las de empleados)
•	se consideran las visitas con status ok (las que se corresponden con el evento de entrada o salida con éxito, sino sería intentos de entrada/salida sería parte de otro análisis)
•	se convierte check_in_timestamp en formato date para que sirva como eje temporal
•	se convierte check_in_timestamp en formato datetime
•	se convierte check_out_timestamp en formato datetime
•	se eliminan los días que se cerró el gimnasio
•	se crea una columna adicional que es la duración de la visita en minutos, tendrá valor vacío si no existe check_out_timestamp
•	se eliminan las visitas de duración menores a 5 minutos, las de menor duración se entienden que son anómalas
•	se deduplican accesos que han ocurrido en menos de 10 min (umbral para deduplicar), se elige el último que se entiende que ha sido el exitoso
•	se elimina la última visita de cada baja al gimnasio, tanto en la tabla de visitas como en la df_visits_minute_details que suele ser la visita presencial para comunicarla la baja, ya que no refleja un acceso real al gimnasio
•	una vez depuradas las visitas, es crítico calcular la concurrencia al centro en cada minuto de cada visita de cada cliente, puesto que esto determina su experiencia
Para ello:
•	es necesario saber cuándo acaba y termina una visita; como muchas visitas no tienen checkout, se estima un check_out aún cuando cliente no lo haya hecho. Para ello se estima la duración de la visita en función del promedio de las visitas de los últimos 30 días naturales anteriores de ese cliente y si no se toma un promedio del centro que son 75 min. Esa estimación es auxiliar sólo para calcular el check_out ajustado, no es que la reemplace cuando sea cero, para no contaminar otros análisis.
•	se realiza un dataframe auxiliar (df_datetimemin_visits) con todos los datetimemin de apertura del centro y que contengan el número de visitas concurrentes en cada minuto; de esta forma el cálculo de la concurrencia en cada minuto se realiza una única vez y no una por cada visita por cada miembro
•	a partir del dataframe anterior, se realiza un dataframe auxiliar que para id de visita (df_visits_minute_detail), id de cliente, minuto de la visita, determina la concurrencia (con cuántos clientes coincidió incluyendo él)
________________________________________
🔗 CONSOLIDACIÓN DE DATASETS
Para mayor facilidad en en análisis, se consolida la información en dos datasets, para que cada uno contenga toda la información
•	Clientes (df_clients): dataset indexado por member_id que consolida por member_id, la información de los datasets: Club Members, Membership Definition, Membership Instances
•	Visitas (df_visits): dataset indexado por id (visita) que consolida la información de los dataset de Clientes: fecha de nacimiento, sexo, fecha de alta, fecha de baja, membresía
•	Detalle de al visita (df_visits_minute_detail), que contiene la concurrencia por minuto de cada visita de cada miembro
________________________________________
📁 GENERACIÓN DE FICHEROS DE DATOS
Se generan tres ficheros de datos csv, con los dataset de clientes (“Clients.csv”), visitas (“Visits.csv”), y detalle de visitas (“VisitsMinute_detail.csv”). Estos serían el origen de datos de PowerBI.
________________________________________
📈 Dashboard y Visualización
El dashboard se ha desarrollado íntegramente en Power BI.
Dashboard operativo orientado al análisis de:
•	comportamiento histórico de clientes,
•	evolución de altas y bajas,
•	visitas y concurrencia,
•	ocupación,
•	segmentación,
•	métricas operativas.
Principios de diseño
Como principio general, se realizan como medidas de PowerBI todas las variables que son calculadas y que varían en el tiempo (p.ej. número clientes activos, edad, antigüedad de un cliente, días desde la última visita etc ), incluso su categorización -en su caso- con segmentación dinámica (pej. segmentos de edad o permanencia).
Las razones son:
•	las medidas de PoweBI permiten calcular de forma eficiente variables dinámicas (ejecución bajo demanda, tablas temporales incluídas en el propio código DAX de la medida)
•	se reduce la complejidad de la ETL: sólo se cargan Datasets con datos estáticos, no tablas con ejes temporales y valor de indicadores en éstos ejes
________________________________________
🧠 Preparación del Modelo en Power BI

En la preparación del Dashboard en PowerBI se han seguido estas fases:
•	ETL para importación y carga a través de PowerQuery; proceso muy sencillo ya que la limpieza y transformación está ya realizada en python en fases anteriores. Sólo carga del csv- encabezados promovidos, de las tablas “Clients”, “Visits”, “Visits_Minute_detail”
•	Creación de dos tablas de eje temporal para poder realizar evolutivos: una de días “Calendario” para cálculos de kpis día, otra de minutos “CalendarioDateTimeMin” para poder hacer cálculos de KPIs-minutos (concurrencia), dos tablas para hacer el cálculos de cohortes (“Matriz_Altas_Cohortes”, “Matriz_Bajas_Cohortes) que agrupan las fechas de la tabla Calendario por meses y las combinan con meses de antigüedad. Estas tablas además tienen la etiqueta del mes y día en formato numérico o texto y tienen un columna orden.
•	activación de relaciones entre las tablas de datos: Clients, Visits, Visist_Minute_detail entre sí (relación a través de member_id), y de estas tablas con las tablas de tiempo Calendario, CalendarioDateTimeMin, Matriz_Altas_Cohortes y Matriz_Bajas_Cohortes
•	estructuración del dashboard y medidas por página, agrupando medidas calculadas en tablas para mejor acceso (pej las métricas de Clientes por un lado, las de Visitas por otro etc)
•	uso de la herramienta externa “Tabular Editor”, para hacer los comparativos de un valor con respecto a referencias pasadas con Calculation Items (por ejemplo, comparativa con el mes anterior, año anterior etc). De esta forma, se realiza un Calculation Item por cada comparativo y no medida calculada para cada métrica y comparativo.
________________________________________
📌 Dashboard: KPIs y Métricas Analizadas
El Dashboard de PowerBI se ha estructurado con estas páginas:
________________________________________
1. “Resumen ejecutivo”
Permite conocer de un vistazo el desempeño del negocio.
Estructura
Dos franjas.
Primera franja
Tarjetas de KPIs actuales en grande y comparativos abajo más pequeño:
•	Número de Clientes
•	ARPU
•	Permanencia
•	MRR
Debajo:
•	comparación con el 1 de mes,
•	el mes pasado,
•	y ese mes el año pasado al ser un negocio estacional,
•	comparación tanto en valor absulto como relativo.
También:
•	Churn
Debajo:
•	comparación con el valor mes -1,
•	mes -2,
•	mes -3.
Segunda franja
Dos evolutivos para ver tendencias:
15.	Evolutivo mensual dos últimos años de:
o	Clientes a 1ro de mes,
o	Altas,
o	Bajas.
16.	Evolutivo mensual dos últimos años de:
o	ARPU,
o	Churn.
________________________________________
2. “Clientes. Caracterización”
Datos demográficos.
Estructura
Dos franjas.
Primera franja
Tarjetas de KPIs actuales en grande y debajo la distrubución de segmentos:
•	Edad Media
•	Sexo
•	Membresías Familiares
•	Mismo CP (si su domicilio tiene el mismo CP que el centro)
•	Permanencia
Segunda franja
Dos evolutivos para ver tendencias:
17.	Evolutivo mensual dos últimos años de segmentos de edad y Edad Media.
18.	Evolutivo mensual dos últimos años de:
o	%Sexo femenino,
o	%Mismo CP,
o	Clientes con permanencia de más de 3 años tanto en valor absoluto como relativo.
________________________________________
3. “Clientes.Visitas.”
Caracterización de las visitas desde la óptica de cliente, no del centro.
Estructura
Dos franjas.
Primera franja
Tarjetas de KPIs actuales en grande y comparativos abajo más pequeño.
Los KPIs son:
•	Clientes Activos (los que han visitado el centro en el último mes)
•	Visitas mes por activo (último mes)
•	Duración de la Visita en minutos (último mes)
•	Días desde la última visita
Debajo del KPI actual en mayúscula, está debajo más pequeño:
•	el valor del mes actual,
•	del mes anterior,
•	y de ese mes el año pasado.
Segunda franja
19.	Gráfico evolutivo mensual de los últimos dos años del %de clientes activos y su distribución por número de visitas.
20.	Gráfico evolutivo mensual últimos dos años de Distribución de Clientes actuales por segmentos de Edad-Sexo, en activos y distribución de número de visitas.
________________________________________
4. “Centro. Ocupación. Evolutivo Mensual”
Visitas recibidas, duración, minutos de saturación del centro (concurrencia en el centro de más de 90 personas incluído el cliente).
Primera franja
Valores actuales de los KPIs y debajo los valores:
•	el mes anterior,
•	hace dos meses,
•	y mismo mes hace un año.
Segunda franja
Evolutivo mensual de los últimos dos años de:
•	Visitas por mes,
•	distribución de la ocupación mensual,
•	minutos de saturación por mes.
Segunda Franja
Evolutivo mensual de:
•	Visitas de Clientes únicos,
•	Duración promedio de visita por cliente,
•	Distribución de la Experiencia de Cliente.
La Experiencia de Cliente se realiza en función del porcentaje de minutos de visita del cliente que no existió saturación.
________________________________________
5. “Centro. Ocupación. Evolutivo diario”
Idem que el anterior pero diario y de los últimos 30 días.
________________________________________
6. “Mapa de Calor de Ocupación día-hora”
Concurrencia, Duración Visita, por Día de la Semana, Hora y Segmentadores de:
•	Sexo,
•	Edad,
•	Membresía,
•	CP.
Está hecho por heatmap y no curva contínua porque:
•	se ve mejor los datos,
•	permite colores,
•	es más visual,
•	y con más datos.
Permite conocer:
•	cuántas personas hay en cada hora,
•	cuánto dura su visita,
•	y emplear segmentadores si se quiere.
Tiene además:
•	filas y columnas totales,
•	promedios.
________________________________________
7. “Altas. Caracterización”
Datos demográficos.
Estructura
Dos franjas.
Primera franja
Tarjetas de KPIs:
1. “Altas hoy”
En grande, debajo en pequeño:
•	el mes acumulado,
•	los últimos 30 días,
•	el mes anterior,
•	y el mismo período el año pasado.
2. De las altas en los útimos 30 días
•	Edad Media en grande (debajo en pequeño distribución por Sexo)
•	Sexo Femenino% en grande (debajo más pequeño la distribución por sexos- franjas de edad)
•	Membresías Familiares% (debajo distribución por tipo de Membresía)
•	Mismo CP en grande, debajo más pequeño distribución por CP
•	ARPU en grande, debajo en pequeño:
o	ARPU acumulado del mes,
o	del mes anterior,
o	y del mismo mes el año pasado.
Segunda franja
Dos evolutivos para ver tendencias:
21.	Evolutivo mensual de las altas en los dos últimos años de segmentos de edad y Edad Media.
22.	Evolutivo mensual de altas dos últimos años de:
o	%Sexo femenino,
o	%Mismo CP.
________________________________________
8. “Altas. Engagement.”
Visitas de las altas en los primeros 30 días.
Primera Franja
Mismos KPIs que la página anterior.
Segunda franja
1.	Gráfico evolutivo mensual de los últimos dos años del %de altas activas y su distribución por número de visitas.
2.	Grafico evolutivo mensual últimos dos años de Distribución de Clientes actuales por segmentos de Edad-Sexo, en activos y distribución de número de visitas.
________________________________________
9. “Bajas. Caracterización”
Datos demográficos.
Estructura
Dos franjas.
Primera franja
Tarjetas de KPIs:
1. “Bajas hoy”
En grande, debajo en pequeño:
•	el mes acumulado,
•	los últimos 30 días,
•	el mes anterior,
•	y el mismo período el año pasado.
2. De las bajas en los útimos 30 días
•	Edad Media en grande (debajo en pequeño distribución por Sexo)
•	Sexo Femenino% en grande (debajo más pequeño la distribución por sexos- franjas de edad)
•	Membresías Familiares% (debajo distribución por tipo de Membresía)
•	Mismo CP en grande, debajo más pequeño distribución por CP
•	ARPU en grande, debajo en pequeño:
o	ARPU acumulado del mes,
o	del mes anterior,
o	y del mismo mes el año pasado.
Segunda franja
Dos evolutivos para ver tendencias:
1.	Evolutivo mensual de las bajas en los dos últimos años de segmentos de edad y Edad Media.
2.	Evolutivo mensual de bajas en los dos últimos años de:
o	%Sexo femenino,
o	%Mismo CP.
________________________________________
10. “Bajas. Engagement.”
Bajas activas y visitas en el último mes.
Primera Franja
Mismos KPIs que la página anterior.
Segunda franja
1.	Gráfico evolutivo mensual de los últimos dos años del %de bajas activas en el último mes y su distribución por número de visitas.
2.	Grafico evolutivo mensual últimos dos años de:
o	promedio días desde la última visita,
o	visitas promedio en el último mes.
________________________________________
11. “Altas. Cohortes”
Agrupación de altas por cohortes mensuales, evolutivo mensual por antigüedad desde el mes 0 (mes del alta) hasta el mes 12, de datos de:
•	% de Altas que permanecen como clientes,
•	% de altas activas (con visitas ese mes),
•	y número de visitas.
Esta primera página es en forma matricial con:
•	totales,
•	y desglose por mes-año de cohorte
para poder ver patrones y diferencias entre las cohortes.
________________________________________
12. “Altas. Cohortes. Resumen”
Dos franjas.
Primera franja
El evolutivo mensual por mes de antiguedad de:
•	Altas que se mantienen como clientes
•	y de Altas que son activas (visitan el centro).
Franja de abajo
Es el evolutivo mensual por mes de antigüedad de:
•	distribución de las altas por segmentos de experiencia (saturación del centro)
•	y duración promedio de la visita en minutos.
Como son gráficos agrupados por antigüedad, se ha puesto arriba de la página un segmentador por:
•	cohorte (mes-año),
•	sexo,
•	segmento de edad,
•	membresía,
•	y Código Postal.
________________________________________
13. “Bajas. Cohortes”
Agrupación de bajas por cohortes mensuales, evolutivo mensual por antigüedad inversa desde el mes 0 (mes de la baja) hasta el mes -12, de datos de:
•	% de Bajas que permanecen como clientes,
•	% de bajas activas (con visitas ese mes),
•	y número de visitas.
Esta primera página es en forma matricial con:
•	totales
•	y desglose por mes-año de cohorte
para poder ver patrones y diferencias entre las cohortes.
________________________________________
14. “Bajas. Cohortes. Resumen”
Dos franjas.
Primera franja
El evolutivo mensual por mes de antiguedad inversa de:
•	bajas que se mantienen como clientes
•	y de bajas que son activas (visitan el centro).
Franja de abajo
Es el evolutivo mensual por mes de antigüedad inversa de:
•	distribución de las bajas por segmentos de experiencia (saturación del centro)
•	y duración promedio de la visita en minutos.
Como son gráficos agrupados por antigüedad, se ha puesto arriba de la página un segmentador por:
•	cohorte (mes-año),
•	sexo,
•	segmento de edad,
•	membresía,
•	y Código Postal.


________________________________________
🗂️ Estructura del Proyecto
.
├── data/
├── notebooks/
│   └── etl_proyecto_final_python.ipynb
├── dashboards/
├── .env
├── requirements.txt
└── README.md
________________________________________
⚙️ Instalación y Requisitos
Clonar el repositorio
git clone https://github.com/usuario/repositorio.git
cd repositorio
Crear entorno virtual
python -m venv venv
Windows
venv\Scripts\activate
Linux / Mac
source venv/bin/activate
Instalar dependencias
pip install -r requirements.txt
Configurar variables de entorno
CLUB_SECRET=tu_club_secret
API_KEY=tu_api_key
Ejecutar notebook
jupyter notebook
________________________________________
📊 Resultados y Conclusiones
El proyecto permitió obtener una visión completa del comportamiento de clientes y operaciones del gimnasio.
Entre los principales resultados destacan:
•	análisis histórico de clientes,
•	evolución temporal de altas y bajas,
•	patrones de asistencia,
•	análisis de ocupación,
•	cohortes de retención,
•	análisis operativo del gimnasio.
La combinación entre ETL en Python y modelado analítico en Power BI permitió separar correctamente:
•	procesamiento de datos,
•	lógica analítica,
•	visualización,
•	cálculos dinámicos mediante DAX.
________________________________________
🔄 Próximos Pasos
Los siguientes pasos, podrían ser:
•	Automatización completa de la ETL
•	Dashboard en tiempo real, con migración a base de datos, botón refresco y/o actualización periódica
•	Modelos predictivos de churn.
•	Segmentación avanzada.
•	Forecasting de ocupación.
________________________________________
✒️ Autor
Antonio Fernández Borondo
Proyecto Final — Análisis de Datos, ETL y Business Intelligence
GitHub: https://github.com/afborondo