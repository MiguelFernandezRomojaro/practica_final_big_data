# practica_final_big_data
## Introducción
Para la realización de este proyecto se ha utilizado una Máquina Virtual Ubuntu 18.04.

## Instalación y configuración
Primeramente, se ha procedido a la instalación y configuración de los programas necesarios en la Máquina Virtual.
-	Intellij (jdk_1.8)
-	Python3 (versión sugerida 3.7)
-	PIP
-	SBT
-	MongoDB
- Spark (versión obligatoria 3.1.2)
- Scala (versión sugerida 2.12)
-	Zookeeper
-	Kafka (versión obligatoria kafka_2.12-3.0.0)

## Inicio práctica
Una vez se tenga la Máquina Virtual donde se va a desarrollar la práctica bien configurada, se procede a la descarga de datos. Para ello cambie de directorio al practica_big_data_2019
```
cd practica_big_data_2019
```
Una vez instalados los programas se procede a la instalación de las librerías de python:
```
pip3 install -r requirements.txt
```
Si durante la instalación surgen problemas con la instalación de las librerías numpy, pillow o findspark, ejecute los siguientes comandos para la instalación de manera individual:
```
pip3 install pillow==8.3.2
pip3 install numpy
pip3 install findspark
```
A continuación, se descargan los datos para el análisis predictivo en tiempo real:
```
resources/download_data.sh
```
## Iniciar Zookeeper
Diríjase al directorio donde haya instalado Kafka y abra una consola de comandos (pulsando F4) e inicie y configure el servicio Zookeeper con el siguiente comando:
```
cd Kafka/kafka_2.12-3.0.0
bin/zookeeper-server-start.sh config/zookeeper.properties
```
## Iniciar Kafka
Diríjase al directorio donde haya instalado Kafka y abra una consola de comandos (pulsando F4) e inicie y configure el servicio Kafka con el siguiente comando:
```
bin/kafka-server-start.sh config/server.properties
```
Desde el mismo directorio, abra una nueva consola de comandos y cree un nuevo topic llamado “flight_delay_classification_request” desde el servicio de Kafka, en una partición y esa partición replicada en factor de uno. Para ello ejecute el siguiente comando:
```
bin/kafka-topics.sh \
 --create \
 --bootstrap-server localhost:9092 \
 --replication-factor 1 \
 --partitions 1 \
 --topic flight_delay_classification_request
```
Si el topic se ha creado correctamente, debería aparecer el siguiente mensaje en la consola de comandos:
```
Created topic "flight_delay_classification_request".
```
Para mayor comprobación de la correcta elaboración del topic, se mostrará la lista con todos los topics creados, para ello ejecute el siguiente comando en la misma consola de comandos donde creó el topic:
```
bin/kafka-topics.sh --bootstrap-server localhost:9092 –list
```
Output:
```
flight_delay_classification_request
```
## Importar datos aerolíneas
Importar todos los datos enriquecidos de las aerolíneas como la colección "aerolíneas". Para ello ejecutar el script import_distances.sh con el siguiente comando:
```
./resources/import_distances.sh
```
## Entrenamiento para obtener el modelo de datos
Primeramente, establecer las variables de entorno necesarias para el desarrollo y funcionamiento del sistema.
Establecer la variable de entorno JAVA_HOME en el directorio donde se haya realizado la instalación de Java, en el caso concreto de este proyecto:
```
export JAVA_HOME=/usr/
```
A modo de comprobación se ejecuta:
```
echo $JAVA_HOME
```
Output:
```
/usr/
```
También se debe de establecer la variable de entorno SPARK_HOME relativa a Spark, en el directorio donde se haya realizado la instalación de Spark, en el caso concreto de este proyecto:
```
export SPARK_HOME=/opt/spark
```
A modo de comprobación se ejecuta:
```
echo $SPARK_HOME
```
Output:
```
export SPARK_HOME=/opt/spark
```
A continuación, ejecutar el script de Python “train_spark_mllib_model.py”, que entrenará el modelo con PySpark mllib:
```
python3 resources/train_spark_mllib_model.py .
```
A modo de comprobación se puede listar el directorio de modelos:
```
ls ./models
```
## Arrancar el predictor de vuelos
En primer lugar, ir a la clase MakePrediction.scala en el directorio /practica_big_data_2019/flight_prediction/src y modificar la variable path a la ruta donde se haya clonado el repositorio “practica_big_data_2019”:
```
val base_path= "/home/upm/Desktop/practica_big_data_2019"
```
Para ejecutar ese código, se puede usar el compilador de código Intellij o spark-submit. 
## Ejecutar código MakePrediction.scala con Intellij
Abrir el programa previamente instalado (se ha instalado junto a su interfaz de usuario) e importar el proyecto scala “flight_prediction”. Intellij informará sobre la necesidad de instalar unas librerías de scala para el correcto funcionamiento del proyecto, instalar dichas librerías y una vez instaladas, ejecutar el proyecto.

![image](https://user-images.githubusercontent.com/94782443/142758749-9ae92a32-0f3a-4ac8-80bb-f82f622ec653.png)
## Ejecutar código MakePrediction.scala con Spark-submit
Primeramente se usará el compilador SBT con el objetivo de compilar el proyecto “flight_prediction” y generar un JAR de la aplicación para su uso con Spark-submit.
Compilar el proyecto “flight_prediction”, abrir una consola de comandos en el directorio /practica_big_data_2019/flight_prediction y ejecutar:
```
sbt compile
```
Crear el JAR de la aplicación para su posterior uso en Spark-submit. Abrir una consola de comandos en el directorio /practica_big_data_2019/flight_prediction y ejecutar:
```
sbt package
```
Finalmente, con la herramienta Spark-submit, arrancar el Spark-Streaming con los dos conectores MongoDB y Kafka, junto con el JAR de la aplicación de vuelos previamente generado con SBT:
```
/opt/spark/bin/spark-submit
 --class es.upm.dit.ging.predictor.MakePrediction
 --packages org.mongodb.spark:mongo-spark-connector_2.12:3.0.1,org.apache.spark:spark-sql-kafka-0-10_2.12:3.1.2
 /home/upm/Desktop/practica_big_data_2019/flight_prediction/target/scala-2.12/flight_prediction_2.12-0.1.jar 
```
## Configuración de Flask para la aplicación web de predicción de vuelos
Establecer la variable de entorno con el path al repositorio “practica_big_data_2019” descargado, en este caso:
```
export PROJECT_HOME=/home/upm/Desktop/practica_big_data_2019
```
A modo de comprobación se ejecuta:
```
echo $PROJECT_HOME
```
Output:
```
/home/upm/Desktop/practica_big_data_2019
```
Dirigirse al directorio /practica_big_data_2019/resources/web y ejecutar el script de funcionamiento de la aplicación web “predict_flask.py” con los siguientes comandos:
```
cd practica_big_data_2019/resources/web
python3 predict_flask.py
```
Acceder a la aplicación web en http://localhost:5000/flights/delays/predict_kafka y añadir algún retraso.
![image](https://user-images.githubusercontent.com/94782443/142759150-2255d9a4-a2fd-46c6-a099-3718220646b8.png)

## Comprobación del retraso registrado en MongoDB
Finalmente, se va a comprobar que el retraso se ha registrado correctamente en MongoDB. Para ello conectar con MongoDB y buscar en la base de datos:
```
mongo
>use agile_data_science;
>db.flight_delay_classification_response.find();
```
Output:
```
{ "_id" : ObjectId("618026a2eee0d202a9eb9582"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2021-11-01T17:40:47.198Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "483202f7-9175-4248-8251-1efb9391eaab", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 2 }
{ "_id" : ObjectId("618abb7bcd10505d497eef50"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2021-11-09T18:18:32.339Z"), "FlightDate" : ISODate("2016-12-25T00:00:00Z"), "Carrier" : "AA", "UUID" : "3e9800db-cae3-4d7d-835d-4b854fd2ca83", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 2 }
{ "_id" : ObjectId("618ac8e1244a44306a3582a0"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2021-11-09T19:15:42.262Z"), "FlightDate" : ISODate("2016-12-25T00:00:00Z"), "Carrier" : "AA", "UUID" : "d7586a79-fa5c-4335-b0c8-4b3539ff5ffb", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 2 }
{ "_id" : ObjectId("619a26c21abac41411b3904c"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 15, "Timestamp" : ISODate("2021-11-21T11:00:16.706Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "eb9c1472-f75f-4d6f-983c-21c6a42da04d", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 2 }
```
## Dockerfiles
A continuación  se va a proceder a la contenerización del sistema mediante el uso de Docker. Se va a implementar la aplicación de igual manera, con la diferencia de que se van a separar los servicios de Zookeeper, Kafka, MongoDB, Spark y Flask, configurando cada uno de los mismo en contenedores Docker independientes.

El procedimiento general para el despliegue y configuración de este sistema consiste en la creación de un fichero de configuración Dockerfile para cada contenedor, que importará una imagen base y sobre esa imagen instalará programas, ejecutará comandos, añadirá variables de entorno, etc. para lograr tener una imagen ajustada a las necesidades del proyecto.

Con el fichero de configuración ya creado, se construirá la imagen del contenedor Docker deseado, y finalmente, con esa imagen se arrancará el contenedor Docker que contendrá el servicio deseado.

La comunicación entre los contenedores Docker se va a realizar mediante la conexión mediante puertos en la red local (localhost).
### Dockerfile Zookeeper
Para este contenedor se ha descargado una imagen base de zookeeper (última versión) desde bitnami y se ha configurado el puerto sobre el que funcionará el contenedor, que en el caso de Zookeeper, tiene asignado por defecto el puerto 2181.

Creación de la imagen Zookeeper:
```
sudo docker build -t=zookeeper-image .
```
Arranque del contenedor de Zookeeper a partir de la imagen Zookeeper:
```
sudo docker run --name zookeeper-container --network host zookeeper-image.
```
### Dockerfile kafka
Para este contenedor se ha descargado una imagen base de kafka (última versión) desde wurstmeister y se ha configurado el puerto sobre el que funcionará el contenedor, que en el caso de Kafka, tiene asignado por defecto el puerto 9092. Dentro de las variables de entorno del Dockerfile se define que Kafka actuará sobre la red local (localhost), se conecta al contenedor Zookeeper a través del puerto 2181 de la red local y se crea directamente un topic llamado “flight_delay_classification_request” en 1 partición y replicado en factor de 1.

Creación de la imagen Kafka:
```
sudo docker build -t= kafka-image .
```
Arranque del contenedor de Kafka a partir de la imagen Kafka:
```
sudo docker run --name kafka-container --network host kafka-image
```
### Dockerfile MongoDB
Para este contenedor se ha descargado una imagen base de una máquina Ubuntu_18.04 y sobre esa máquina se ha instalado MongoDB. El puerto por defecto de MongoDB es el 27017. Mediante ENTRYPOINT definimos el comando que se ejecutará en el momento de arranque del contenedor Docker para el arranque del daemon de MongoDB. Para el correcto funcionamiento, se copia la carpeta “practica_final_2019” dentro del contenedor.

Creación de la imagen MongoDB:
```
sudo docker build -t=mongodb-image . 
```
Arranque del contenedor de MongoDB a partir de la imagen MongoDB:
```
sudo docker run --name mongodb-container --network host mongodb-image
```
Una vez creados y arrancados todos y cada uno de los contenedores, entrar dentro del contenedor de MongoDB:
```
sudo docker exec -it mongodb-container bash
```
Ya dentro del contenedor MongoDB, ir a la carpeta donde se haya copiado la carpeta “practica_final_2019” e importar todos los datos enriquecidos de las aerolíneas como la colección "aerolíneas". Para ello ejecutar el script import_distances.sh con el siguiente comando:
```
cd CarpetaMongoAux/
/bin/bash resources/import_distances.sh
```
### Dockerfile Spark
Para este contenedor se ha descargado una imagen base de una máquina Ubuntu_18.04 y sobre esa máquina se han instalado los programas, herramientas y ficheros necesarios para el funcionamiento de Spark. El puerto asignado por defecto a Spark es el 7077.
A modo de resumen, se instala una configuración por defecto, Java, Python, PIP, SBT, Scala, Spark. Se copia la carpeta completa del proyecto “practica_final_2019” dentro del contenedor, se compila el proyecto Scala “flight_prediction” y se genera un JAR de la aplicación para su posterior uso con Spark-submit.
Para resolver la problemática de que sólo se puede ejecutar una orden en el ENTRYPOINT, se ha creado un fichero SH que contiene los tres comandos que se necesitan ejecutar en la construcción del contenedor Spark: arrancar el master de Spark, arrancar los slaves de Spark y arrancar la aplicación MakePrediction generada con SBT.

Creación de la imagen Spark:
```
sudo docker build -t= spark-image . 
```
Arranque del contenedor de Spark a partir de la imagen Spark:
```
sudo docker run --name spark-container --network host spark-image
```
### Dockerfile Flask
Para este contenedor se ha descargado una imagen base con el python3.7 ya configurado y se han instalado las librerías del requirements.txt, el puerto por defecto del Flask es el 3000. Par lograr el funcionamiento de Flask en el contenedor Docker, se ha copiado la carpeta del proyecto “practica_final_2019” dentro del contenedor. Finalmente, se ejecuta arranca el Flask en el ENTRYPOINT con el programa “predict_flask.py”.

Creación de la imagen Flask:
```
sudo docker build -t=flask-image .
```
Arranque del contenedor de Flask a partir de la imagen Flask:
```
sudo docker run --name flask-container --network host flask-image
```
## Docker-compose
Con Docker-Compose se van a contenerizar los servicios de Zookeeper, Kafka, MongoDB, Spark y Flask a partir de las imágenes generadas con los DockerFiles, la diferencia con el aparado anterior es que los servicios se van a larzar/arrancar todos a la vez en lugar de uno a uno.
Para lograr el objetivo, primeramente se tiene que instalar Compose en la Máquina Virtual:
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo chmod 777 /home/upm/Desktop/practica_big_data_2019/docker/docker-compose.yml
```
A modo de comprobación de la correcta instalación del Compose, comprobar la versión instalada con el siguiente comando:
```
docker-compose --version
```
Para la creación del “docker-compose.yml” se definen los cinco servicios y dentro de cada servicio se define:
-	Nombre del contenedor que se creará
-	Nombre de la imagen ya creada con la que se creará el contenedor
-	Modo de conectividad, en este caso se usará la red local (localhost)
- En el caso concreto de Kafka, también se define su dependencia con Zookeeper
Una vez creado el “Docker-compose.yml”, simplemente queda situarse en el mismo directorio que el fichero y ejecutar el comando:
```
sudo docker-compose up
```
## Docker-compose en Google Cloud
De igual manera que en el apartado anterior, se va a contenerizar la aplicación, pero en vez de en una máquina virtual local, se va a realizar mediante una máquina virtual Compute Engine de Google Cloud en la nube, es decir, remota.
Para ello se han de instalar una serie de programas necesarios para la ejecución de la aplicación, ya que la máquina viene vacía.
En primer lugar, se establece una sesión SSH para subir los ficheros de la aplicación. Una vez subido se instala un descompresor para descomprimir el fichero subido:
### Instalar descompresor
```
sudo apt install p7zip-full
7za x practica_final_2019.7z.001
```
### Instalar Docker
```
sudo apt-get install docker.io
```
### Instalar Compose
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo chmod 777 /home/fromojaromiguel/practica_big_data_2019/docker/docker-compose.yml
```
A modo de comprobación:
```
docker-compose --version 
```
### Construcción de imágenes Docker 
A continuación se procede a la creación de la imágenes docker, situarse en los directorios donde se ubican los Dockerfile de cada servicio (Zookeeper, Kafka, MongoDB, Spark y Flask) y ejecutar los siguientes comandos en la consola:
```
sudo docker build -t=zookeeper-image .
sudo docker build -t=kafka-image .
sudo docker build -t=mongodb-image .
sudo docker build -t=spark-image .
sudo docker build -t=flask-image .
```
### Arrancar Compose
Con Docker-Compose se van a contenerizar los servicios de Zookeeper, Kafka, MongoDB, Spark y Flask a partir de las imágenes generadas con los Dockerfiles:
```
sudo docker-compose up -d
```
![image](https://user-images.githubusercontent.com/94782443/142883370-5474b5c6-a705-4682-ab4e-c28a9f3196e6.png)

Comprobación contenedores creados:
```
sudo docker ps
```
![image](https://user-images.githubusercontent.com/94782443/142883398-0b74ee2d-db02-438a-8e56-453306b2fbdc.png)

### Importar distancias en MongoDB
Importar todos los datos enriquecidos de las aerolíneas como la colección "aerolíneas". Para ello ejecutar el script import_distances.sh con el siguiente comando:
```
sudo docker exec -it docker_mongodb-container_1 bash 
cd CarpetaMongoAux/
/bin/bash resources/import_distances.sh
```
### Instalar interfaz gráfica navegador
En esta instancia de Google Cloud no hay instalada ninguna interfaz gráfica, es por ello que, para el un uso más amigable de la aplicación y su comprobación, se instala la interfaz gráfica del navegador Lynx para ejecutar la aplicación de vuelos:
Instalar la interfaz gráfica Lynx:
```
sudo apt-get install Lynx
```
Abrir la interfaz gráfica con la página de la aplicación:
```
lynx http://localhost:5000/flights/delays/predict_kafka
```
La siguiente imagen muestra el menú de la aplicación:

![image](https://user-images.githubusercontent.com/94782443/142883483-618317f7-8f21-489c-a41b-ea1b6c70eede.png)

Tras consultar un retraso, se obtiene una salida de confirmación del sistema:

![image](https://user-images.githubusercontent.com/94782443/142883526-bd6e0d61-47b9-4d15-a2c0-0b88df624b29.png)

### Comprobar resultados en MongoDB
Finalmente comprobamos que la operación se ha registrado de manera satisfactoria consultando MongoDB. Entrar al contenedor con el comando:
```
sudo docker exec -it docker_mongodb-container_1 bash
```
Una vez dentro:
```
mongo
>use 
>db.flight_delay_classification_response.find();
```
Obteniendo la siguiente salida con los registros:

![image](https://user-images.githubusercontent.com/94782443/142883588-b9c50d26-8017-4d55-940c-7c1a594ea5e6.png)
















