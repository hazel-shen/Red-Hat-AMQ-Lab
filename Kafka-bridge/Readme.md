# Kafka Bridge - Access Kafka Topic from HTTP

- 内部客户端是与Kafka Bridge本身在同一Kubernetes集群中运行的基于容器的HTTP客户端。内部客户端可以访问KafkaBridge定制资源中定义的主机和端口上的Kafka Bridge。
- 外部客户端是在Kubernetes群集之外运行并部署Kafka Bridge的HTTP客户端。外部客户端可以通过OpenShift的Route，LoadBalance服务或K8s的Ingress访问KafkaBridge。

![Alt text](https://developers.redhat.com/sites/default/files/styles/article_feature/public/blog/2020/07/kafka-bridge-1-1024x683.png?itok=Sw9Kr66Y)


測試 Kafka Bridge - 發送 Message
```
oc expose svc my-bridge-bridge-service

KAFKA_BRIDGE=$(oc get route my-bridge-bridge-service -o template --template '{{.spec.host}}')
echo $KAFKA_BRIDGE

KAFKA_TOPIC=my-topic
echo $KAFKA_TOPIC

curl -v http://$KAFKA_BRIDGE/healthy
curl -X POST \
  http://$KAFKA_BRIDGE/topics/$KAFKA_TOPIC \
  -H 'content-type: application/vnd.kafka.json.v2+json' \
  -d '{
    "records": [
        {
            "key": "key-1",
            "value": "value-1"
        },
        {
            "key": "key-2",
            "value": "value-2"
        }
    ]
}'

KAFKA_TOPIC=${1:-'my-topic'}
KAFKA_CLUSTER_NS=${2:-'kafka'}
KAFKA_CLUSTER_NAME=${3:-'my-cluster'}

## 暫時建立一個 kafka consumer instance
oc -n my-app-project run kafka-consumer -ti \
    --image=strimzi/kafka:0.15.0-kafka-2.3.1 \
    --rm=true --restart=Never \
    -- bin/kafka-console-consumer.sh \
    --bootstrap-server my-cluster-kafka-bootstrap:9092 \
    --topic $KAFKA_TOPIC --from-beginning


```

測試 Kafka Bridge - 接收 Message

```
KAFKA_BRIDGE=$(oc get route my-bridge-bridge-service -o template --template '{{.spec.host}}')
KAFKA_CONSUMER_GROUP=my-group
curl -X POST http://$KAFKA_BRIDGE/consumers/$KAFKA_CONSUMER_GROUP \
  -H 'content-type: application/vnd.kafka.v2+json' \
  -d '{
    "name": "my-consumer",
    "format": "json",
    "auto.offset.reset": "earliest",
    "enable.auto.commit": false
  }'
```


