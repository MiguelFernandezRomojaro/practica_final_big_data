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










