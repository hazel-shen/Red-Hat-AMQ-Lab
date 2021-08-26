
oc apply -f https://raw.githubusercontent.com/liuxiaoyu-git/amq-streams-foundations-labs/master/04_monitoring/kafka.yaml

oc apply -f https://raw.githubusercontent.com/liuxiaoyu-git/amq-streams-foundations-labs/master/04_monitoring/connect.yaml


cat > hello-world.yaml
apiVersion: kafka.strimzi.io/v1alpha1
kind: KafkaTopic
metadata:
  name: my-topic
  labels:
      strimzi.io/cluster: my-cluster
spec:
  selector:
    matchLabels:
      strimzi.io/cluster: my-cluster
  replicas: 3
  partitions: 12
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello-world-producer
  name: hello-world-producer
spec:
  selector:
    matchLabels:
      app: hello-world-producer
  replicas: 1
  template:
    metadata:
      labels:
        app: hello-world-producer
    spec:
      containers:
      - name: hello-world-producer
        image: strimzici/hello-world-producer:support-training
        env:
          - name: BOOTSTRAP_SERVERS
            value: my-cluster-kafka-bootstrap:9092
          - name: TOPIC
            value: my-topic
          - name: DELAY_MS
            value: "1000"
          - name: LOG_LEVEL
            value: "INFO"
          - name: MESSAGE_COUNT
            value: "1000000"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello-world-consumer
  name: hello-world-consumer
spec:
  selector:
    matchLabels:
      app: hello-world-consumer
  replicas: 1
  template:
    metadata:
      labels:
        app: hello-world-consumer
    spec:
      containers:
      - name: hello-world-consumer
        image: strimzici/hello-world-consumer:support-training
        env:
          - name: BOOTSTRAP_SERVERS
            value: my-cluster-kafka-bootstrap:9092
          - name: TOPIC
            value: my-topic
          - name: GROUP_ID
            value: my-hello-world-consumer
          - name: LOG_LEVEL
            value: "INFO"
          - name: MESSAGE_COUNT
            value: "1000000"

oc apply -f hello-world.yaml


oc logs $(oc get pod -l app=hello-world-producer -o=jsonpath='{.items[0].metadata.name}') -f
oc logs $(oc get pod -l app=hello-world-consumer -o=jsonpath='{.items[0].metadata.name}') -f


oc apply -f https://raw.githubusercontent.com/liuxiaoyu-git/amq-streams-foundations-labs/master/04_monitoring/prometheus/grafana.yaml -n kafka
oc apply -f https://raw.githubusercontent.com/liuxiaoyu-git/amq-streams-foundations-labs/master/04_monitoring/prometheus/prometheus.yaml -n kafka
oc apply -f https://raw.githubusercontent.com/liuxiaoyu-git/amq-streams-foundations-labs/master/04_monitoring/prometheus/alerting-rules.yaml -n kafka

oc expose svc prometheus -n kafka
oc get route grafana prometheus -n kafka



Ref:
https://slacker.ro/2021/04/19/connect-amq-streams-to-your-red-hat-openshift-4-monitoring-stack/
https://github.com/strimzi/strimzi-kafka-operator


cat << EOF | oc apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    prometheusK8s: 
      volumeClaimTemplate:
       spec:
         storageClassName: fast
         volumeMode: Filesystem
         resources:
           requests:
             storage: 40Gi
EOF



