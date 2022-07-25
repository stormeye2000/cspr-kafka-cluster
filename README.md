## Multiple Kafka and Zookeeper replicas via StatefulSets



The following scripts provide a highly available resilient Zookeeper/Kafka cluster

Zookeeper runs with three replica pods, which is enough to provide a quorum, if one fails, a new leader will be elected whilst the failed pod is replaced. This is enough for this implementation.

Kafka also runs with three replicas, which again provides a quorom if one replica fails. As the project matures I expect to use more replicas and to also change some of the Kafka start-up server properties

To install the scripts, clone the project and do the following:

```shell
cd multiple-brokers-statefulset
```

Note: the single deployment folder is for reference only

If you haven't already created a 'kafka' namespace in your cluster do the following:

```shell
kubectl apply -f kafka-namespace.yml
```

Set the 'kafka' namespace to the current context:

```shell
kubectl config set-context --current --namespace kafka
```

Now deploy Zookeeper:

```
kubectl apply -f zookeeper.yml
```

Wait for it to deploy all three pods:

```
kubectl get pods -w -l app=zookeeper
```

Now deploy Kafka:

```
kubectl apply -f kafka.yml
```

And again wait for them to finish deployment:

```
kubectl get pods -w -l app=kafka-broker
```

