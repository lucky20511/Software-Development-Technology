# Example of Producer and Consumer

### Producer

[https://kafka.apache.org/0100/javadoc/org/apache/kafka/clients/producer/KafkaProducer.html](https://kafka.apache.org/0100/javadoc/org/apache/kafka/clients/producer/KafkaProducer.html)

```
 Properties props = new Properties();

 props.put("bootstrap.servers", "localhost:9092");
 props.put("acks", "all");
 props.put("retries", 0);
 props.put("batch.size", 16384);
 props.put("linger.ms", 1);
 props.put("buffer.memory", 33554432);
 props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
 props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");


 // sync or async
 Producer<String, String> producer = new KafkaProducer<>(props);
 
 for (int i = 0; i < 100; i++) {
     //sync
     producer.send(new ProducerRecord<String, String>("my-topic", Integer.toString(i), Integer.toString(i)))
         .get();

    //async
    producer.send(new ProducerRecord<String, String>("my-topic", Integer.toString(i), Integer.toString(i)),
        new DemoCallBack(startTime, messageNo, messageStr));

}
 producer.close();
```

### Consumer

[https://kafka.apache.org/0100/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html](https://kafka.apache.org/0100/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html)

```
 Properties props = new Properties();
 props.put("bootstrap.servers", "localhost:9092");
 props.put("group.id", "test");
 props.put("enable.auto.commit", "true");
 props.put("auto.commit.interval.ms", "1000");
 props.put("session.timeout.ms", "30000");
 props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
 props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

 KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);

 consumer.subscribe(Arrays.asList("foo", "bar"));

 while (true) {
     ConsumerRecords<String, String> records = consumer.poll(100);
     for (ConsumerRecord<String, String> record : records)
         System.out.printf("offset = %d, key = %s, value = %s", record.offset(), record.key(), record.value());
 }
```



