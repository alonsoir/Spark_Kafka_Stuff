# Proyecto RiskShield.

	/Users/aironman/gitProjects/sopra/RiskShield

	Básicamente es un motor de fraudes, hay que reescribir uno existente usando tecnología big data.

	Probablemente los libros de Jacek son la mejor ayuda.
	https://jaceklaskowski.gitbooks.io/mastering-spark-sql/content/spark-sql.html

##  Conceptos básicos sobre Kafka. 

	Video chulo de Oscar:
	
		https://youtu.be/vrnU-KVYbSo

	KSQLDB habla del concepto de tabla en kafka, a la que puedes acceder operando a la manera SQL, pero más que una tabla, es un snapshot sobre un topic, es decir, es como sacarle una foto a ese topic, y puedes acceder a esos datos a la manera SQL. 

	El uso de la partition key es vital para conseguir consumir los mensajes en el orden en el que se produjeron. Mensajes que tengan el mismo partition key, iran a la misma particion dentro de ese topic y podrán ser consumidos en el orden en el que se produjeron. Vital en muchísimos entornos.

	Como asegurar un adecuado rebalanceo usando un partition Key? 

	Esta es una pregunta que hay que resolver. 
	Hay que procurar crear tambien consumer groups, de manera que así, los consumidores asignados a esos grupos consuman los mensajes asignados por sus partition key.

	Tienes que mirar schema registry, avro, spark, spark-sql. 

	Asumiendo que los mensajes que llegan al kafka de ING sigan un orden y sea absolutamente imperativo procesarlos en orden,
	como trata ING el problema sobre procesar mensajes problemáticos ordenados? tiene alguna solución ya implementada, probada y aceptada a nivel global?

		No, cada squad se lo guisa y se lo come.

	Hay alguna solución basada en usar consumidores con listas stash? 
	https://medium.com/datadriveninvestor/if-youre-using-kafka-with-your-microservices-you-re-probably-handling-retries-wrong-8492890899fa

## Schema registry

	Una vez que tienes levantada la plataforma, puedes ir a control-center

	http://localhost:9021/clusters/EIjI4rm5T_i_pC_4_r5u4Q/management/topics/transactions/message-viewer

## Tutorial Schema registry y un poco de avro

	https://docs.confluent.io/current/schema-registry/schema_registry_onprem_tutorial.html#schema-registry-onprem-tutorial

## ejemplo esquema avro
	cat src/main/resources/avro/io/confluent/examples/clients/basicavro/Payment.avsc

	{
	 "namespace": "io.confluent.examples.clients.basicavro",
 	 "type": "record",
 	 "name": "Payment",
 	 "fields": [
     	{"name": "id", "type": "string"},
     	{"name": "amount", "type": "double"}
 		]
	}

	https://docs.confluent.io/current/schema-registry/serdes-develop/serdes-avro.html#messages-avro-reflection

	https://docs.confluent.io/current/streams/developer-guide/datatypes.html#avro

## ejemplo cliente Kafka AVRO! Producer, consumer...

	/Users/aironman/gitProjects/examples/clients/avro

	...
	import io.confluent.kafka.serializers.KafkaAvroSerializer;
	...
	props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
	props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, KafkaAvroSerializer.class);
	...
	KafkaProducer<String, Payment> producer = new KafkaProducer<String, Payment>(props));
	final Payment payment = new Payment(orderId, 1000.00d);
	final ProducerRecord<String, Payment> record = new ProducerRecord<String, Payment>(TOPIC, payment.getId().toString(), payment);
	producer.send(record);
	...

## Probablemente tengas que editar o crear esto. localhost es lo mismo que 0.0.0.0

	cat $HOME/.confluent/java.config

	bootstrap.servers=localhost:9092
	schema.registry.url=http://localhost:8081


## Avro Producer full example
	https://github.com/confluentinc/examples/blob/6.0.0-post/clients/avro/src/main/java/io/confluent/examples/clients/basicavro/ProducerExample.java

	mvn exec:java -Dexec.mainClass=io.confluent.examples.clients.basicavro.ProducerExample -Dexec.args="$HOME/.confluent/java.config"

## Avro Consumer full example. Ojo, no te vale levantar el típico consumidor kafka para poder "ver" los mensajes. Necesitas algo así. 
	https://github.com/confluentinc/examples/blob/6.0.0-post/clients/avro/src/main/java/io/confluent/examples/clients/basicavro/ConsumerExample.java

	mvn exec:java -Dexec.mainClass=io.confluent.examples.clients.basicavro.ConsumerExample -Dexec.args="$HOME/.confluent/java.config"

## Podemos interactuar con control-center para ver el esquema aplicado. Por ejemplo, para el topic "transactions"

	http://localhost:9021/clusters/EIjI4rm5T_i_pC_4_r5u4Q/management/topics/transactions/schema/value

## Tambien podemos usar curl

	curl --silent -X GET http://localhost:8081/subjects/ | jq .

## con más detalle, recomendado

	curl --silent -X GET http://localhost:8081/subjects/transactions-value/versions/latest | jq .

## dado su id, si lo sabes

	curl --silent -X GET http://localhost:8081/schemas/ids/1 | jq .

## Enlaces interesantes schema-registry && AVRO

	https://docs.confluent.io/current/quickstart/ce-docker-quickstart.html#ce-docker-quickstart

	https://avro.apache.org/docs/current/spec.html

	https://www.kai-waehner.de/blog/2020/03/12/can-apache-kafka-replace-database-acid-storage-transactions-sql-nosql-data-lake/

	https://www.confluent.io/kafka-summit-sf18/a-solution-for-leveraging-kafka-to-provide-end-to-end-acid-transactions/

	https://www.google.com/search?client=safari&rls=en&q=how+to+choose+a+good+partition+key+in+kafka&ie=UTF-8&oe=UTF-8

	https://axoniq.io/blog-overview/axon-and-kafka-how-does-axon-compare-to-apache-kafka#0

	https://medium.com/datadriveninvestor/if-youre-using-kafka-with-your-microservices-you-re-probably-handling-retries-wrong-8492890899fa

	https://medium.com/better-programming/the-truth-about-your-source-of-truth-a1eb833c2d70

	https://medium.com/better-programming/commands-and-events-in-a-distributed-system-282ea5918c49

	https://levelup.gitconnected.com/microservice-monitoring-and-observability-for-software-engineers-ab47920dde12

	https://www.confluent.io/kafka-summit-san-francisco-2019/from-trickle-to-flood-with-kafkaing/

	https://github.com/alonsoir/spring-kafka-poison-pill

## SPARK

	Para refrescarlo, voy a mirar de nuevo las guias de structured streaming y sql. Me bajo el repositorio para poder los mismo ejemplos de la guia oficial.

	https://github.com/apache/spark.git
	
	https://spark.apache.org

	https://dzone.com/articles/apache-spark-in-a-nutshell

	https://spark.apache.org/docs/latest/sql-programming-guide.html

	https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html

	cd /Users/aironman/gitProjects/spark

	spark-shell

	# some code

	import spark.implicits._

	//typical word count...
	sc.textFile("/Users/aironman/confluent-6.0.0/README").flatMap(line=>line.split(" ")).map(word=>(word,1)).reduceByKey((a,b)=>a+b).foreach(println)

	# DATAFRAMES
	val df = spark.read.json("examples/src/main/resources/people.json")

	// Displays the content of the DataFrame to stdout
	df.show()
	// +----+-------+
	// | age|   name|
	// +----+-------+
	// |null|Michael|
	// |  30|   Andy|
	// |  19| Justin|
	// +----+-------+

	// This import is needed to use the $-notation
	import spark.implicits._
	// Print the schema in a tree format
	df.printSchema()
	// root
	// |-- age: long (nullable = true)
	// |-- name: string (nullable = true)

	// Select only the "name" column
	df.select("name").show()
	// +-------+
	// |   name|
	// +-------+
	// |Michael|
	// |   Andy|
	// | Justin|
	// +-------+

	// Select everybody, but increment the age by 1
	df.select($"name", $"age" + 1).show()
	// +-------+---------+
	// |   name|(age + 1)|
	// +-------+---------+
	// |Michael|     null|
	// |   Andy|       31|
	// | Justin|       20|
	// +-------+---------+

	// Select people older than 21
	df.filter($"age" > 21).show()
	// +---+----+
	// |age|name|
	// +---+----+
	// | 30|Andy|
	// +---+----+

	// Count people by age
	df.groupBy("age").count().show()
	// +----+-----+
	// | age|count|
	// +----+-----+
	// |  19|    1|
	// |null|    1|
	// |  30|    1|
	// +----+-----+

	// Register the DataFrame as a SQL temporary view
	df.createOrReplaceTempView("people")

	val sqlDF = spark.sql("SELECT * FROM people")
	sqlDF.show()
	// +----+-------+
	// | age|   name|
	// +----+-------+
	// |null|Michael|
	// |  30|   Andy|
	// |  19| Justin|
	// +----+-------+

	// Register the DataFrame as a global temporary view
	df.createGlobalTempView("people")

	// Global temporary view is tied to a system preserved database `global_temp`
	spark.sql("SELECT * FROM global_temp.people").show()
	// +----+-------+
	// | age|   name|
	// +----+-------+
	// |null|Michael|
	// |  30|   Andy|
	// |  19| Justin|
	// +----+-------+

	// Global temporary view is cross-session
	spark.newSession().sql("SELECT * FROM global_temp.people").show()
	// +----+-------+
	// | age|   name|
	// +----+-------+
	// |null|Michael|
	// |  30|   Andy|
	// |  19| Justin|
	// +----+-------+

	# Creating Datasets
	case class Person(name: String, age: Long)

	// Encoders are created for case classes
	val caseClassDS = Seq(Person("Andy", 32)).toDS()
	caseClassDS.show()
	// +----+---+
	// |name|age|
	// +----+---+
	// |Andy| 32|
	// +----+---+

	// Encoders for most common types are automatically provided by importing spark.implicits._
	val primitiveDS = Seq(1, 2, 3).toDS()
	primitiveDS.map(_ + 1).collect() // Returns: Array(2, 3, 4)

	// DataFrames can be converted to a Dataset by providing a class. Mapping will be done by name
	val path = "examples/src/main/resources/people.json"
	val peopleDS = spark.read.json(path).as[Person]
	peopleDS.show()
	// +----+-------+
	// | age|   name|
	// +----+-------+
	// |null|Michael|
	// |  30|   Andy|
	// |  19| Justin|
	// +----+-------+

	## Inferring the Schema Using Reflection
	// For implicit conversions from RDDs to DataFrames
	import spark.implicits._

	// Create an RDD of Person objects from a text file, convert it to a Dataframe
	val peopleDF = spark.sparkContext
	  .textFile("examples/src/main/resources/people.txt")
	  .map(_.split(","))
	  .map(attributes => Person(attributes(0), attributes(1).trim.toInt))
	  .toDF()
	// Register the DataFrame as a temporary view
	peopleDF.createOrReplaceTempView("people")

	// SQL statements can be run by using the sql methods provided by Spark
	val teenagersDF = spark.sql("SELECT name, age FROM people WHERE age BETWEEN 13 AND 19")

	// The columns of a row in the result can be accessed by field index
	teenagersDF.map(teenager => "Name: " + teenager(0)).show()
	// +------------+
	// |       value|
	// +------------+
	// |Name: Justin|
	// +------------+

	// or by field name
	teenagersDF.map(teenager => "Name: " + teenager.getAs[String]("name")).show()
	// +------------+
	// |       value|
	// +------------+
	// |Name: Justin|
	// +------------+

	// No pre-defined encoders for Dataset[Map[K,V]], define explicitly
	implicit val mapEncoder = org.apache.spark.sql.Encoders.kryo[Map[String, Any]]
	// Primitive types and case classes can be also defined as
	// implicit val stringIntMapEncoder: Encoder[Map[String, Any]] = ExpressionEncoder()

	// row.getValuesMap[T] retrieves multiple columns at once into a Map[String, T]
	teenagersDF.map(teenager => teenager.getValuesMap[Any](List("name", "age"))).collect()
	// Array(Map("name" -> "Justin", "age" -> 19))

	26/11/2020

	Vamos a suponer que la demo para el proyecto RiskShield implica hacer una carga de datos en una bd MariaDB usando sparkSQL.

# Primero 

necesito el driver jdbc de MariaDB en el classpath de spark. En mi caso, lo tengo en:
	
		/Users/aironman/.m2/repository/org/mariadb/jdbc/mariadb-java-client/2.4.4


# Segundo, necesito tener levantado el servidor:

		# start MariaDB Server
		mysql.server start

		# or autostart it (optional if above dont apply)
		brew services start mariadb

		# 
		mariadb-secure-installation

		#log in as your user
		mysql

		#Or log in as root 
		mysql -u root -p

		#password is set as root
	
	Tengo una base de datos llamada commands, que contiene una tabla user_data:

		MariaDB [commands]> show databases;
		+--------------------+
		| Database           |
		+--------------------+
		| arqu_local         |
		| commands           |
		| information_schema |
		| javistepa_banking  |
		| mysql              |
		| performance_schema |
		| queries            |
		| sys                |
		+--------------------+
		8 rows in set (0.003 sec)

		MariaDB [commands]> desc user_data;
		+---------------+---------+------+-----+---------+-------+
		| Field         | Type    | Null | Key | Default | Extra |
		+---------------+---------+------+-----+---------+-------+
		| ID_USER_DATA  | int(11) | YES  |     | NULL    |       |
		| NAME          | text    | YES  |     | NULL    |       |
		| DATE_REGISTER | text    | YES  |     | NULL    |       |
		+---------------+---------+------+-----+---------+-------+
		3 rows in set (0.002 sec)


# Tercero

arranco la spark-shell. --packages descargará el jar de maven central. --driver-class-path es necesario para que no de una excepción a la hora de cargar una tabla en un Dataframe.

	spark-shell --driver-class-path org.mariadb.jdbc:mariadb-java-client:2.4.4 --packages org.mariadb.jdbc:mariadb-java-client:2.4.4
	
En mi caso, estoy trabajando con Spark 3.0.1 y MariaDB 10.5.8-MariaDB Homebrew

Si todo ha ido bien, comprobamos que tenemos el jar cargado:

		Class.forName("org.mariadb.jdbc.Driver")

		import java.io.InputStream;
		import java.util.Properties;
		import org.apache.spark.sql.Dataset;
		import org.apache.spark.sql.Row;
		import org.apache.spark.sql.SparkSession;
		import static org.apache.spark.sql.functions.col;

		val jdbcHostname = "localhost"
		val jdbcPort = 3306
		val jdbcDatabase = "commands"
		val jdbcUsername = "root"
		val jdbcPassword = "root"
		val source_table = "user_data"
		// Create the JDBC URL without passing in the user and password parameters.
		val jdbcUrl = s"jdbc:mysql://${jdbcHostname}:${jdbcPort}/${jdbcDatabase}"

		val connectionProperties = new Properties()
		connectionProperties.put("user", jdbcUsername)
		connectionProperties.put("password", jdbcPassword)

		val userDataDF = spark.read.option("driver","org.mariadb.jdbc.Driver").jdbc(jdbcUrl, source_table, connectionProperties)

		userDataDF.printSchema

		userDataDF.schema

		case class user_data(ID_USER_DATA: Integer, NAME: String, DATE_REGISTER: String)

		val dfUsers = List(user_data(1,"ALONSO", "26-11-2020"),user_data(2,"PAPA", "26-11-2020")).toDF()

		dfUsers.write.format("jdbc")
						.mode("overwrite")
						.option("driver","org.mariadb.jdbc.Driver")
						.option("url", jdbcUrl)
						.option("truncate","false")
						.option("dbtable", source_table)
						.option("user", jdbcUsername)
						.option("password", jdbcPassword)
						//.saveAsTable("USERS")
						.save()

		// Bien usas saveAsTable para guardar el resultado en una tabla de spark que he llamado USERS, bien usas save() para guardar los datos en la tabla de MariaDB.
		// Las dos no puedes a la vez.
		// Después de ejecutar la anterior sentencia, en la base de datos podemos ver estos datos.
		MariaDB [commands]> select * from user_data;
		+--------------+--------+---------------+
		| ID_USER_DATA | NAME   | DATE_REGISTER |
		+--------------+--------+---------------+
		|            2 | PAPA   | 26-11-2020    |
		|            1 | ALONSO | 26-11-2020    |
		+--------------+--------+---------------+
		2 rows in set (0.000 sec)


		// Guardamos los datos en una tabla de Spark
		dfUsers.write.mode("overwrite").saveAsTable("USER_DATA")
		val recover_dfUsers = spark.read.table("USER_DATA")
		recover_dfUsers.show()
		+------------+------+-------------+
		|ID_USER_DATA|  NAME|DATE_REGISTER|
		+------------+------+-------------+
		|           1|ALONSO|   26-11-2020|
		|           2|  PAPA|   26-11-2020|
		+------------+------+-------------+

https://docs.databricks.com/data/data-sources/sql-databases.html#step-1-check-that-the-jdbc-driver-is-available

# Uniendo Spark, kafka y MariaDB.

Necesitas esta dependencia como mínimo si vas a trabajar con maven.
	
	groupId = org.apache.spark
	artifactId = spark-sql-kafka-0-10_2.12
	version = 3.0.1

Lanzo la spark-shell

	spark-shell --driver-class-path org.mariadb.jdbc:mariadb-java-client:2.4.4 --packages org.mariadb.jdbc:mariadb-java-client:2.4.4,org.apache.spark:spark-sql-kafka-0-10_2.12:3.0.1

Comprobamos si tenemos un topic con el que trabajar

	docker-compose exec broker kafka-topics --list --bootstrap-server 0.0.0.0:9092

Si no tenemos ninguno, habrá que crearlo, se llamará transactions

	docker-compose exec broker kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic transactions

// producimos unos cuantos mensajes el topic transactions...

	docker-compose exec broker kafka-console-producer --topic transactions --bootstrap-server 0.0.0.0:9092

// algo de codigo para la spark-shell
// Ojito, que si pones el comando en varias líneas, spark-shell va a asignar una variable a cada una de las líneas. Algo absurdo.
// Creating a Kafka Sink for Streaming Queries
// Write key-value data from a DataFrame to a specific Kafka topic specified in an option
import org.apache.spark.sql.streaming.Trigger

val df = spark.readStream.format("kafka").option("kafka.bootstrap.servers", "localhost:9092").option("subscribe", "transactions").load()

df.selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)").as[(String, String)]

df.printSchema

// escribimos en la consola , previo casting de binario a UTF8 para que podamos verlos bien...
val messages = df.selectExpr("cast (value as string) AS Content")

messages.writeStream.outputMode(“append”).format(“console”).option(“truncate”, false).option("checkpointLocation", "/tmp").trigger(Trigger.ProcessingTime("1 second")).start().awaitTermination()

TROUBLESHOOTING

	1) Al lanzar la spark-shell, ojito, que no puede haber espacios en el packages, separados por comas.

	2) Al lanzar el comando para subscribirte al topic kafka y tenerlo en un dataframe, si pones el comando en varias líneas, spark-shell va a asignar una variable a cada una de las líneas. Algo absurdo.

		val df = spark.readStream.format("kafka").option("kafka.bootstrap.servers", "localhost:9092").option("subscribe", "transactions").load()

	3) 
 

	org.apache.spark.sql.AnalysisException: checkpointLocation must be specified either through option("checkpointLocation", ...) or SparkSession.conf.set("spark.sql.streaming.checkpointLocation", ...);

https://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html

https://spark.apache.org/docs/latest/submitting-applications.html

