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


