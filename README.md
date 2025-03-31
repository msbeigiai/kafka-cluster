
# Kafka on Kubernetes (Minikube) - 3-Broker Cluster Setup

This guide will help you set up a **Kafka cluster with 3 brokers** on **Minikube** using **Strimzi**. It includes all steps, from setting up Minikube to interacting with Kafka topics, producers, and consumers.

## Prerequisites
Before starting, ensure you have the following installed:
- **Minikube**: [Install Minikube](https://minikube.sigs.k8s.io/docs/)
- **Kubectl**: [Install Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- **Helm** (optional, if needed for installation): [Install Helm](https://helm.sh/docs/intro/install/)

## Step 1: Start Minikube
Run Minikube to start a Kubernetes cluster:
```bash
minikube start --memory=8192 --cpus=4
```

## Step 2: Install Strimzi
Strimzi is used to manage Kafka on Kubernetes. Run the following to install it:
```bash
kubectl create namespace kafka
kubectl apply -f https://strimzi.io/install/latest | kubectl apply -f -
```

## Step 3: Deploy the Kafka Cluster

Create a file called `kafka-cluster-3nodes.yaml` and paste the following YAML:

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: my-cluster
  namespace: kafka
spec:
  kafka:
    version: 3.9.0   
    replicas: 3  
    listeners:
      - name: plain
        port: 9092
        type: internal
        tls: false
      - name: tls
        port: 9093
        type: internal
        tls: true
    config:
      offsets.topic.replication.factor: 3
      transaction.state.log.replication.factor: 3
      transaction.state.log.min.isr: 2
      default.replication.factor: 3
      min.insync.replicas: 2
      log.message.format.version: "3.9"
    storage:
      type: persistent-claim
      size: 5Gi
      deleteClaim: false
  zookeeper:
    replicas: 3 
    storage:
      type: persistent-claim
      size: 5Gi
      deleteClaim: false
  entityOperator:
    topicOperator: {}
    userOperator: {}
```

Apply the YAML to Kubernetes:
```bash
kubectl apply -f kafka-cluster-3nodes.yaml -n kafka
```

Check the status of your pods:
```bash
kubectl get pods -n kafka
```

You should see Kafka and Zookeeper pods running.

## Step 4: Verify Kafka Cluster Status
Check the Kafka custom resource:
```bash
kubectl get kafka -n kafka
```

You should see the cluster listed as **READY**.

## Step 5: Expose the Kafka Service (Optional for External Access)
To connect to Kafka from outside Minikube, expose the Kafka service by port-forwarding:
```bash
kubectl port-forward -n kafka svc/my-cluster-kafka-bootstrap 9092:9092
```

## Step 6: Connect and Interact with Kafka

### Open a Shell Inside a Kafka Pod
You can connect to any of the Kafka brokers to run Kafka commands. For example, connect to `my-cluster-kafka-0`:
```bash
kubectl exec -it my-cluster-kafka-0 -n kafka -- bash
```

### Create a Topic
```bash
kafka-topics.sh --bootstrap-server localhost:9092 --create --topic my-test-topic --partitions 3 --replication-factor 2
```

### List Topics
```bash
kafka-topics.sh --bootstrap-server localhost:9092 --list
```

### Produce Messages to the Topic
```bash
kafka-console-producer.sh --bootstrap-server localhost:9092 --topic my-test-topic
```
Type your messages and hit **Enter**. Press **CTRL+C** to exit.

### Consume Messages from the Topic
```bash
kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-test-topic --from-beginning
```

You should see the messages you produced.

### Delete the Topic
```bash
kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic my-test-topic
```

## Step 7: Clean Up
Once you're done, delete the Kafka cluster:
```bash
kubectl delete kafka my-cluster -n kafka
```

## Conclusion
You now have a **3-broker Kafka cluster** running in **Minikube** with Strimzi. You can interact with it, create topics, produce and consume messages.

Feel free to modify the `kafka-cluster-3nodes.yaml` to scale the number of brokers, change replication settings, or customize your deployment.

Enjoy your Kafka experience! 

