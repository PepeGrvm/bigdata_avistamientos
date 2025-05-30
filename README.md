# Projecto Avistamientos Big Data

### Integrantes:
* José Ponce
* Anibal Kaune

### Asignatura:
* Big Data

### Carrera:
* Ingeniería en informática

## Introducción

El proyecto consisten en el procesamiento de datos y generación de gráficos de un dataset con datos sobre avistamientos de ovnis. 

## Descripción

El dataset con el que se ha trabajado contiene datos sobre avistamientos de ovnis documentados desde 1910, en este se mencionan las fechas de los incidentes, la fecha de los registros, la ciudad, provincia o estado, país, duración de encuentro, descripción del evento, la forma del objeto volador no identificado,latitud y longitud. Con este conjunto de datos, se procede a realizar el preprocesamento de estos para posteriormente generar gráficos con información relevante, esto en base a cumplir con los siguientes objetivos:

* Cantidad de avistamientos de las 5 formas de objetos más vistas
* Cantidad de avistamientos por año
* Cantidad de avistamientos por estado dentro de los Estados Unidos

Para realizar todos los procesos requeridos para el procesamiento de datos, consulta de los mismos y generación de gráficos, se utilizaron las herramientas disponibles en Google Cloud Platform, haciendo uso de opciones de almacenamiento (Cloud Storage, BigQuery), herramientas de preparación de datos (Dataprep) y herramientas de viualización de datos mediante gráficos (Looker Studio).

## Trabajo del dataset con Google Cloud Platform

### Roles y permisos

Antes de comenzar a trabajar con los datos y las diferentes herramientas a utilizar, es necesario verificar que usuario posee los roles y permisos necesarios para acceder y utilizar las herramientas y servicios y modificar los datos del dataset almacenado, de lo contrario, no se podrán realizar los trabajos requeridos. Para ello se debe ingresar a Menu -> IAM & admin -> IAM, una vez ahí, se selecciona el usuario principal y se observan los permisos y roles que posee, este debe tener permiso o rol de editor para poder trabajar con los datos, si este no lo tiene, se selecciona Add another role, se busca y selecciona el rol Editor y se guardan los cambios. Cabe mencionar que también se requiere agregar el rol Dataprep Service Agent.

<p align="center">
  <img src="img/Permisos perfil.png" alt="Permisos perfil" width="800"/>
  <h6 align="center"><i>Permisos agregados al perfil de usuario<i></h6>
</p>

### Almacenamiento de los datos

Se accede a Cloud Storage, una vez ahí se selecciona la opción Create Bucket, se ingresa un nombre valido para el bucket (GCP notifica al usuario si el nombre del bucket ya está en uso por otro usuario), se presiona en create y se crea el bucket. 

<p align="center">
  <img src="img/Creación Bucket.png" alt="Creación Bucket" width="800"/>
</p>

Una vez que el bucket se ha creado, se presiona en Uploud -> Uploud files, se selecciona el archivo que se necesita subir y se confirma la carga del archivo.
Nota: El archivo original se encuentra en formato xlmx, GCP no acepta este formato, por lo que se necesita transformar el archivo a CSV para subirlo.

<p align="center">
  <img src="img/csv subido.png" alt="csv subido" width="800"/>
  <h6 align="center"><i>Bucket con csv cargado<i></h6>
</p>

Una vez que se ha subido el archivo, se ingresa a BigQuery, en este se puede ver el id del proyecto, en los tres puntos al lado de este se selecciona la opción Create dataset, se ingresa el nombre del dataset (en este caso se nombró ovni) y se presiona en Create dataset.

<p align="center">
  <img src="img/Creación dataset.png" alt="Creación dataset" width="800"/>
</p>

### Dataprep

Antes de proceder con el trabajo con los datos, se requiere prepararlos para ser utilizados adecuadamente, para ello se utilizó Dataprep. Para ello se accede a Menu -> View all products -> Alteryx Designer Cloud, una vez ahí se aceptan todos los terminos y se conceden todos los permisos solicitados.
Una vez que ya se ha ingresado a dataprep, se presiona en Import data from Cloud Storage into BigQuery, se selecciona el archivo con los datos y se presiona en Import & Add to Flow.

<p align="center">
  <img src="img/Inicio Dataprep.png" alt="Dataprep" width="800"/>
</p>

<p align="center">
  <img src="img/Dataprep cloud storage.png" alt="Dataprep cloud storage" width="800"/>
  <h6 align="center"><i>Archivo csv importado a Dataprep<i></h6>
</p>

Una vez que los datos se seleccionaron e importaron, se muestran por medio de una tabla que señala la presencia de datos nulos o que no encajan con el resto de los datos (formato).

<p align="center">
  <img src="img/Dataprep dataview.png" alt="Dataprep dataview" width="800"/>
  <h6 align="center"><i>Datos sin tratar<i></h6>
</p>

Para tratar los datos nulos y los datos que no encajan (mismatch), se realizaron los siguientes pasos:
* En la columna state/province, se presiona en la linea roja sobre los datos (esto señala la presencia de datos mismatch) y en las opciones sugeridas, se selecciona eliminar filas mismatched
* Se reemplazan los missing values de la tabla por el último valor valido de la columna
* Se modifica el nombre de state/province por state_province (esto para evitar que se genere error al exportar la tabla a BigQuery)

<p align="center">
  <img src="img/Dataprep datos tratados.png" alt="Dataprep datos tratados" width="800"/>
  <h6 align="center"><i>Datos tratados<i></h6>
</p>

Una vez que ya se realizaron los cambios, se realizan las transformaciones presionan en Run Job. Una vez que se completa las transformaciones, se realiza la creación de una tabla en BigQuery (llamada avistamientos) y se publican los nuevos datos a esta nueva tabla.

<p align="center">
  <img src="img/Dataprep flow 3.png" alt="Dataprep flow 3" width="800"/>
  <h6 align="center"><i>Flujo de trabajo Dataprep<i></h6>
</p>

<p align="center">
  <img src="img/Dataprep publish.png" alt="Dataprep publish" width="800"/>
  <h6 align="center"><i>Datos tratados publicados en BigQuery<i></h6>
</p>

### Consultas BigQuery

Una vez que ya se ha generado la tabla en BigQuery, se accede y se visualizan los datos exportados desde Dataprep en la tabla avistamientos del dataset ovni. Desde ahí se busca obtener información relevante para los objetivos, para ello se abre una nueva pestaña para consultas y en esta página se ingresan las siguientes consultas:

```
SELECT 
  UFO_shape,
  COUNT(*) AS Sightings
FROM 
  `qwiklabs-gcp-02-19c8362ed85d.ovni.avistamientos`
WHERE 
  country = 'us'
GROUP BY 
  UFO_shape
ORDER BY 
  Sightings DESC
LIMIT 5;
```

<p align="center">
  <img src="img/BigQuery Query 1.png" alt="BigQuery Query 1" width="800"/>
  <h6 align="center"><i>Resultados primera consulta<i></h6>
</p>

```
SELECT 
  EXTRACT(YEAR FROM DATETIME(Date_time)) AS Year,
  COUNT(*) AS Sightings
FROM 
  `qwiklabs-gcp-02-19c8362ed85d.ovni.avistamientos`
WHERE 
  country = 'us'
GROUP BY 
  Year
ORDER BY 
  Sightings DESC;
```

<p align="center">
  <img src="img/BigQuery Query 2.png" alt="BigQuery Query 2" width="800"/>
  <h6 align="center"><i>Resultados segunda consulta<i></h6>
</p>

```
SELECT 
  `state_province` AS State,
  COUNT(*) AS Sightings
FROM 
  `qwiklabs-gcp-02-19c8362ed85d.ovni.avistamientos`
WHERE 
  country = 'us'
GROUP BY 
  State
ORDER BY 
  Sightings DESC;
```

<p align="center">
  <img src="img/BigQuery Query 3.png" alt="BigQuery Query 3" width="800"/>
  <h6 align="center"><i>Resultados tercera consulta<i></h6>
</p>

### Looker Studio

Una vez que los datos ya se encuentran preparados para ser utilizados, se ingresa a Looker Studio y se crea un reporte en blanco (sin template), se solicita que se asocie una fuente de datos, en este caso se selecciona BigQuery y la tabla con los datos transformados (avistamientos), se acepta y se abre el editor de reportes.
Nota: Looker Studio solicita que se adquiera una licencia de uso para el usuario.

<p align="center">
  <img src="img/Menu Looker Studio.png" alt="Menu Looker Studio" width="800"/>
</p>

<p align="center">
  <img src="img/Looker add data.png" alt="Looker add data" width="800"/>
  <h6 align="center"><i>Carga de los datos desde BigQuery<i></h6>
</p>

Una vez ya abierto el reporte, se generan los siguientes gráficos:

#### Gráfico 1: Cantidad de avistamientos de las 5 formas de objetos más vistas

Se generó un gráfico con torta de orden descendente con las formas de los objetos más vistos.

<p align="center">
  <img src="img/Gráf 1.png" alt="Gráf 1" width="800"/>
</p>

#### Gráfico 2: Cantidad de avistamientos por año

Se generó un gráfico de barra de orden descendente con la cantidad de avistamientos por año. Para ello se extrajo el año de la columna Date_time.

<p align="center">
  <img src="img/Gráf 2.png" alt="Gráf 2" width="800"/>
</p>

#### Gráfico 3: Cantidad de avistamientos por estado dentro de los Estados Unidos

Se generó un gráfico de mapa (geomap chart) con la cantidad de avistamientos por estado dentro de Estados Unidos. Para ello, se utilizó la columna state_province (esto se debe realizar manualmente agregando en la dimensiones del gráfico la columna y eliminando la dimensión country que aparece por defecto), una vez seleccionado se edita el tipo de dato por Geo -> Country Subdivition 1, una vez aplicado se crea un filtro para que el gráfico solo muestre los datos de las columnas con country igual a us (Estados Unidos).

<p align="center">
  <img src="img/Gráf 3.png" alt="Gráf 3" width="800"/>
</p>

Finalmente, los gráficos acorde a los requisitos del caso se han completado, mostrando la información requerida de forma correcta.

<p align="center">
  <img src="img/Gráficos terminados(ahorita si joven).png" alt="Gráficos terminados" width="800"/>
</p>
