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

Once Zookeeper and Kafka have been deployed double check both services have running pods:

```bash
kubectl get pods --selector=app=zookeeper

NAME          READY   STATUS    RESTARTS   AGE
zookeeper-0   1/1     Running   0          5m
zookeeper-1   1/1     Running   0          5m
zookeeper-2   1/1     Running   0          5m

```

```bash
kubectl get pods --selector=app=kafka-broker

NAME             READY   STATUS    RESTARTS   AGE
kafka-broker-0   1/1     Running   0          5m
kafka-broker-1   1/1     Running   0          5m
kafka-broker-2   1/1     Running   0          5m
```

Check the logs for any obvious errors:

```bash
kubectl logs -l app=zookeeper --all-containers

kubectl logs -l app=kafka-broker --all-containers

```



Importantly, the services drive access to the pods:

```bash
kubectl get services

NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
kafka-service       NodePort    10.100.12.239   <none>        9093:31307/TCP      5m
zookeeper-client    ClusterIP   10.100.12.167   <none>        2181/TCP            5m
zookeeper-service   ClusterIP   None            <none>        2888/TCP,3888/TCP   5m

```

By viewing a pods environment variables, we can see that the services are internally exposed and can be used for access:

```bash
kubectl exec kafka-broker-0 -- env

PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/kafka/bin
HOSTNAME=kafka-broker-0
KAFKA_USER=kafka
KAFKA_DATA_DIR=/var/lib/kafka/data
JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
KAFKA_HOME=/opt/kafka
KAFKA_HEAP_OPTS=-Xmx2G -Xms2G
KAFKA_OPTS=-Dlogging.level=INFO
ZOOKEEPER_CLIENT_PORT=tcp://10.100.12.167:2181
ZOOKEEPER_CLIENT_PORT_2181_TCP=tcp://10.100.12.167:2181
KAFKA_SERVICE_SERVICE_HOST=10.100.12.239
KAFKA_SERVICE_SERVICE_PORT=9093
KUBERNETES_SERVICE_HOST=10.100.0.1
KUBERNETES_PORT_443_TCP_PROTO=tcp
ZOOKEEPER_CLIENT_PORT_2181_TCP_PORT=2181
ZOOKEEPER_CLIENT_PORT_2181_TCP_ADDR=10.100.12.167
KAFKA_SERVICE_SERVICE_PORT_SERVER=9093
KAFKA_SERVICE_PORT=tcp://10.100.12.239:9093
KAFKA_SERVICE_PORT_9093_TCP_PORT=9093
KUBERNETES_PORT_443_TCP=tcp://10.100.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
ZOOKEEPER_CLIENT_SERVICE_HOST=10.100.12.167
ZOOKEEPER_CLIENT_SERVICE_PORT=2181
KAFKA_SERVICE_PORT_9093_TCP=tcp://10.100.12.239:9093
KAFKA_SERVICE_PORT_9093_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.100.0.1
ZOOKEEPER_CLIENT_SERVICE_PORT_CLIENT=2181
ZOOKEEPER_CLIENT_PORT_2181_TCP_PROTO=tcp
KAFKA_SERVICE_PORT_9093_TCP_ADDR=10.100.12.239
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.100.0.1:443
HOME=/home/kafka
```

In the above example of a kafka-broker pod, we can see that that both the ZOOKEEPER_CLIENT_SERVICE_HOST and the ZOOKEEPER_CLIENT_SERVICE_PORT are exposed. These are used in the kafka.yml statefulset to provide access from kafka pods to zookeeper pods via the zookeeper client service.