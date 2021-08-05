
# Kafka Connect

```shell script
curl -LO https://raw.githubusercontent.com/liuxiaoyu-git/OpenShift-HOL/master/uber.csv

mkdir data

mv uber.csv data/

oc rsync data/ my-connect-connect-5b978d984c-bslzc:/tmp

~/Dow/amq-online-1.7.0-install ❯ oc rsync data/ my-connect-connect-5b978d984c-bslzc:/tmp
building file list ... done
./
uber.csv

sent 87676 bytes  received 48 bytes  4279.22 bytes/sec
total size is 87558  speedup is 1.00


oc rsh my-connect-connect-5b978d984c-bslzc
ls /tmp/uber.cvs

cat <<EOF >> /tmp/source-plugin.json
{
  "name": "source-test",
  "config": {
    "connector.class": "FileStreamSource",
    "tasks.max": "3",
    "topic": "my-topic-2",
    "file": "/tmp/uber.csv"
  }
}
EOF
```

#### create kafka topic
```
apiVersion: kafka.strimzi.io/v1alpha1
kind: KafkaTopic
metadata:
  name: my-topic-2
  labels:
    strimzi.io/cluster: my-cluster
spec:
  partitions: 3
  replicas: 2
  config:
    retention.ms: 7200000
    segment.bytes: 1073741824

```

```
oc apply -f kafka-connect-consumer.yaml

## 確認 connect-consumer 還沒有收到 log
oc logs $(oc get pod -n my-app-project -l app=connector-consumer -o=jsonpath='{.items[0].metadata.name}')

## 進到 Kafka Connect 把 uber.cvs 發給  Kafka Connect 處理
oc rsh my-connect-connect-5b978d984c-bslzc

curl -X POST -H "Content-Type: application/json" --data @/tmp/source-plugin.json http://localhost:8083/connectors

## 檢查 connect-consumer 是否有持續收到 log
oc logs $(oc get pod -n my-app-project -l app=connector-consumer -o=jsonpath='{.items[0].metadata.name}')
```



