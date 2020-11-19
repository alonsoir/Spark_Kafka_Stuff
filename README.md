# Proyecto RiskShield.

	/Users/aironman/gitProjects/sopra/RiskShield

	Básicamente es un motor de fraudes

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

	https://spark.apache.org

	https://spark.apache.org/docs/latest/sql-programming-guide.html

	https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html




