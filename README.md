# Monitoring RHPAM's Kie Server with Prometheus and Grafana

## RHPAM's Kie Server

- From OperatorHub, deploy the _Business Automation operator_ into your openshift project.
- To deploy a trial RHPAM environment and to add a service (_kieserver-metrics_) that exposes the metrics from Kie Server to Prometheus, apply the `my-process-svc-trial-prometheus.yaml` file, with this command:

```bash
$ oc apply -f my-process-svc-trial-prometheus.yaml
```

## Prometheus

- Create a new namespace for our Prometheus and Grafana deployment

```bash
$ oc new-project prometheus-grafana
```

- From OperatorHub, deploy the _Prometheus operator_ into your openshift project.
- Apply the `kieserver-prometheus.yaml` file to:
  - Create an OpenShift secret, such as _metrics-secret_, to access the Prometheus metrics on KIE Server. The secret must contain the "username" and "password" elements with KIE Server user credentials.
  - Create a service monitor instance for the (_kieserver-metrics_) service. A ServiceMonitor defines a service endpoint that needs to be monitored by the Prometheus instance.
  - Create a Service Account (_kieserver-prometheus_) with Cluster role and Cluster role binding to ensure you have the permission to get nodes and pods in other namespaces at the cluster scope.
  - Create a Prometheus instance. A Prometheus resource can scrape the targets defined in the ServiceMonitor resource.

```bash
$ oc apply -f kieserver-prometheus.yaml
```

- Verify that the Prometheus services have successfully started.

```bash
$ oc get svc
```

- Check the server logs from one of the target pods to see if the services are running properly.

```bash
$ oc get pods

        NAME                                   READY   STATUS    RESTARTS   AGE
        prometheus-kieserver-prometheus-0      2/2     Running   1          5m45s
        prometheus-kieserver-prometheus-1      2/2     Running   1          5m45s
        prometheus-operator-858466f659-ln7l6   1/1     Running   0          38m

$ oc logs prometheus-kieserver-prometheus-0 -c prometheus
```

- Expose the prometheus-operated service to use the Prometheus console externally.

```bash
$ oc expose svc/prometheus-operated
```

- Visit the Prometheus route and go to the Prometheus targets page. Check to see that the Prometheus targets page is picking up the target endpoints.

```bash
$ echo http://$(oc get route prometheus-operated --template='{{ .spec.host }}')
```

- Use this link to [troubleshoot problems](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/troubleshooting.md#troubleshooting-servicemonitor-changes)

## Grafana

- Choose the same namespace as Prometheus Operator deployment.

```bash
$ oc project prometheus-grafana
```

- From OperatorHub, deploy the _Grafana operator_ into your openshift project.
- You can now deploy the Grafana instance itself by applying the file `kieserver-grafana.yaml`.

```bash
$ oc apply -f kieserver-grafana.yaml
```
