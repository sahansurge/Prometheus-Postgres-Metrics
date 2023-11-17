# Prometheus-Postgres-Metrics

Prometheus metrics collection for Postgres

Before we start we will have to install kube-prometheus-stack which has the prometheus-operator and a Grafana instance.

Then we install the Zalando postgres-operator

This values.yaml file instructs the helm chart, to deploy a configuration for the operator, that will create all PostgreSQL clusters with a sidecar container. This sidecar container runs the postgresql-exporter form the Prometheus community and is configured to connect the local installed cluster and collect various metrics about the individual PostgreSQL-instance.

```
# add repo for postgres-operator
helm repo add postgres-operator-charts https://opensource.zalando.com/postgres-operator/charts/postgres-operator

# install the postgres-operator
helm install --namespace postgres-operator --create-namespace --values values.yaml postgres-operator postgres-operator-charts/postgres-operator
```

Afterwards deploy the deploy a PostgreSQL cluster

```
kubectl create namespace example
kubectl apply -n example -f postgresql.yaml
```

Then we set up the Prometheus monitoring

```
kubectl apply -n example -f postgresql.yaml
```

Port-forward the example-minimal-cluster-0 to port 9187 and see if the metrics are available.

Now port-forward prometheus and access Prometheus http://localhost:9090/ in status go to targets and see the podmonitors there, the work is done.

If you face an issue with the pods not showing in the target then we have to edit the podmonitor.yaml and apply with a small change. We have to add "labels" under metadata and add "release: {kube-prom-stack release name}" for example here my kube-prom-stack release name was "prom-stack" so we edit the podmonitors.yaml and apply with the small change like this and apply.

```
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: postgresql
  labels:
    release: prom-stack
```

Now wait a minute or two and check again in targets the podmonitors must appear. Check with pg_up on the graph and test it out.

References: https://shivering-isles.com/2022/04/postgres-operator-with-monitoring
