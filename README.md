# Install cluster

Install minikube cluster with ./install_1

Wait for kafka pods to be running

Create kafka topic and filebeat release with ./install_2

# Run Kafka producer

kubectl -n kafka run kafka-producer -it --image=banzaicloud/kafka:2.13-2.4.0 --rm=true --restart=Never -- /opt/kafka/bin/kafka-console-producer.sh --broker-list kafka-headless:29092 --topic filebeat

# Run Kafka consumer (Optional)

kubectl -n kafka run kafka-consumer -it --image=banzaicloud/kafka:2.13-2.4.0 --rm=true --restart=Never -- /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka-headless:29092 --topic filebeat --from-beginning

# Show indexes in Elasticsearch

kubectl port-forward svc/elastic-es-http -n elastic-system 9200

ELASTIC_PASSWORD=$(kubectl get secret elastic-es-elastic-user -o go-template='{{.data.elastic | base64decode}}' -n elastic-system)

curl -ku "elastic:$ELASTIC_PASSWORD" https://localhost:9200/_cat/indices

# Steps to reproduce

Send message to kafka

`There is 1 message in filebeat index (expected: 1)`

Restart filebeat pod

`There are 2 messages in filebeat index (expected: 1)`

Send message to kafka

`There are 3 messages in filebeat index (expected: 2)`

Restart filebeat pod

`There are 5 messages in filebeat index (expected: 2)`