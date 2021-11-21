# practica_final_big_data
## INTRODUCCIÓN
Para la realización de este proyecto se ha utilizado una Máquina Virtual Ubuntu 18.04.

## INSTALACIÓN Y CONFIGURACIÓN
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

## INICIO PRÁCTICA
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
## INICIAR ZOOKEEPER
Diríjase al directorio donde haya instalado Kafka y abra una consola de comandos (pulsando F4) e inicie y configure el servicio Zookeeper con el siguiente comando:
```
cd Kafka/kafka_2.12-3.0.0
bin/zookeeper-server-start.sh config/zookeeper.properties
```
## INICIAR KAFKA
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
## IMPORTAR DATOS AEROLINEAS
Importar todos los datos enriquecidos de las aerolíneas como la colección "aerolíneas". Para ello ejecutar el script import_distances.sh con el siguiente comando:
```
./resources/import_distances.sh
```
## ENTRENAMIENTO PARA OBTENER EL MODELO DE DATOS
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
## ARRANCAR EL PREDICTOR DE VUELOS
En primer lugar, ir a la clase MakePrediction.scala en el directorio /practica_big_data_2019/flight_prediction/src y modificar la variable path a la ruta donde se haya clonado el repositorio “practica_big_data_2019”:
```
val base_path= "/home/upm/Desktop/practica_big_data_2019"
```
Para ejecutar ese código, se puede usar el compilador de código Intellij o spark-submit. 
## EJECUTAR CÓDIGO MakePrediction.scala con Intellij
Abrir el programa previamente instalado (se ha instalado junto a su interfaz de usuario) e importar el proyecto scala “flight_prediction”. Intellij informará sobre la necesidad de instalar unas librerías de scala para el correcto funcionamiento del proyecto, instalar dichas librerías y una vez instaladas, ejecutar el proyecto.
![image](https://user-images.githubusercontent.com/94782443/142758749-9ae92a32-0f3a-4ac8-80bb-f82f622ec653.png)
## EJECUTAR CÓDIGO MakePrediction.scala con Spark-submit
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
## CONFIGURACIÓN DE FLASK PARA LA APLICACIÓN WEB DE PREDICCION DE VUELOS
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
## COMPROBACIÓN DEL RETRASO REGISTRADO EN MongoDB
Finalmente, se va a comprobar que el retraso se ha registrado correctamente en MongoDB. Para ello conectar con MongoDB y buscar en la base de datos:
```
mongo
>	use agile_data_science;
>	db.flight_delay_classification_response.find();
```
Output:
```
{ "_id" : ObjectId("618026a2eee0d202a9eb9582"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2021-11-01T17:40:47.198Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "483202f7-9175-4248-8251-1efb9391eaab", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 2 }
{ "_id" : ObjectId("618abb7bcd10505d497eef50"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2021-11-09T18:18:32.339Z"), "FlightDate" : ISODate("2016-12-25T00:00:00Z"), "Carrier" : "AA", "UUID" : "3e9800db-cae3-4d7d-835d-4b854fd2ca83", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 2 }
{ "_id" : ObjectId("618ac8e1244a44306a3582a0"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2021-11-09T19:15:42.262Z"), "FlightDate" : ISODate("2016-12-25T00:00:00Z"), "Carrier" : "AA", "UUID" : "d7586a79-fa5c-4335-b0c8-4b3539ff5ffb", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 2 }
**{ "_id" : ObjectId("619a26c21abac41411b3904c"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 15, "Timestamp" : ISODate("2021-11-21T11:00:16.706Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "eb9c1472-f75f-4d6f-983c-21c6a42da04d", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 2 }**
```
























