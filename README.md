`splunk-streaming-lite-connect will be used with [logdna-streaming](https://github.ibm.com/cloud-partners/logdna-streaming) to forward the IBM logDNA logs to splunk.

# Prerequisites:
1. IBM Kubernetes cluster.
2. Helm package manager.
3. IBM event stream `<BOOTSTRAP_SERVERS>` details and `<APIKEY>`credentials.

# Install splunk-streaming-lite-connect 

clone the splunk-streaming-lite-connect helm chart repository

```shell
git clone https://github.ibm.com/cloud-partners/splunk-streaming-lite-connect.git
```

Edit `connect-distributed.properties` replacing the `<BOOTSTRAP_SERVERS>` and `<APIKEY>` placeholders with your Event Streams credentials.

bootstrap server's are a list of kafka brokers we can get it from IBM event stream service credentials and fill it with comma seprated without quotes.
API key will be avaialble with IBM event stream service credentials

***screen shot*** 

```shell
vi splunk-streaming-lite-connect/config/connect-distributed.properties
```

# install splunk-streaming-lite-connect chart

```shell
helm install <release-name> splunk-streaming-lite-connect
```

<release-name> can be any name given to the installation. This will be prefixed to all the Kubernetes artifacts deployed as a part of the installation.

Wait till all the artifacts are deployed. This shows the sample output of the command `<kubectl get pods>` when the helm chart was installed with `<testreleasename>` as the <release-name>:

***screen shot*** 

### Manage Connectors

To manage connectors, port forward to the `splunk-streaming-lite-connect-service` Service on port 8083:

```shell
kubectl port-forward service/<release-name>-splunk-streaming-lite-connect 8083
```

The Connect REST API is then available via `http://localhost:8083`.
The Connect REST API is documented at https://kafka.apache.org/documentation/#connect_rest

### Run Connectors (This step has to be performed after splunk deployment and splunk HEC token created)

When the Kafka Connect runtime is running, see the instructions for running the connectors:

1. prerequisities to create connector tasks.

    `topics`  Kafka topic configured on the IBM event stream
    `splunk.indexes` to set the destination Splunk indexes (which is the index created on the splunk instance for event search)
    `splunk.hec.token` to set your Http Event Collector (HEC) token 
    `splunk.hec.uri` to the URI for your destination Splunk HEC endpoint.
    
    For more information on Splunk HEC configuration refer to [Splunk Documentation.](http://docs.splunk.com/Documentation/SplunkCloud/latest/Data/UsetheHTTPEventCollector)


2. Run the following command to create connector tasks. 


```
  curl localhost:8083/connectors -X POST -H "Content-Type: application/json" -d '{
    "name": "kafka-connect-splunk",
    "config": {
      "connector.class": "com.splunk.kafka.connect.SplunkSinkConnector",
      "tasks.max": "3",
      "splunk.indexes": "<SPLUNK_INDEXES>",
      "topics":"<YOUR_TOPIC>",
      "splunk.hec.uri": "<SPLUNK_HEC_URI:SPLUNK_HEC_PORT>",
      "splunk.hec.token": "<YOUR_TOKEN>"
    }
  }'
```

3. Verify that data is flowing into your Splunk platform instance by searching using the index specified in the configuration.
4. Use the following commands to check status, and manage connectors and tasks:

```
    # List active connectors
    curl http://localhost:8083/connectors

    # Get kafka-connect-splunk connector info
    curl http://localhost:8083/connectors/kafka-connect-splunk

    # Get kafka-connect-splunk connector config info
    curl http://localhost:8083/connectors/kafka-connect-splunk/config

    # Delete kafka-connect-splunk connector
    curl http://localhost:8083/connectors/kafka-connect-splunk -X DELETE

    # Get kafka-connect-splunk connector task info
    curl http://localhost:8083/connectors/kafka-connect-splunk/tasks
```

