# MAO Metrics

The Machine API Operator(MAO) uses the
[Prometheus project](https://prometheus.io/) to expose metrics. To consume
these metrics directly from the MAO you will need to perform
HTTP GET requests to a specific port and URI of the MAO. The
URI for all metrics is `/metrics`, see the Prometheus documentation for query
parameter options. To find the exposed metrics port for the MAO you can either
inspect the Deployment resource or the
[install manifest](https://github.com/openshift/machine-api-operator/blob/master/install/0000_30_machine-api-operator_11_deployment.yaml)
to find the environment variable `METRICS_PORT`, the default value for this is `8080`.

**Example MAO metrics scrape procedure**
1. Forward the metrics port from the CAO to a local port
   ```
   $ oc port-forward -n openshift-machine-api deployment/machine-api-operator 8080:8080
   ```
2. Perform an HTTP GET request on the local port
   ```
   $ curl http://localhost:8080/metrics
   ```

The Machine API Operator reports the following metrics:

## Metrics about Machine resources

These metrics are based on the number of Machine objects that are currently
observed by the MAO. In this example you can see that there is 1 Machine
currently, see `mapi_machine_items`. For each Machine, there is a corresponding
`mapi_machine_created_timestamp_seconds` entry. These individual entries show
specific information about each Machine, although most of this information will
be static please note that the `phase` variable will be updated to show the
current phase of the Machine.

**Sample metrics**
```
# HELP mapi_machine_items Count of machine objects currently at the apiserver
# TYPE mapi_machine_items gauge
mapi_machine_items 1
# HELP mapi_machine_created_timestamp_seconds Timestamp of the mapi managed Machine creation time
# TYPE mapi_machine_created_timestamp_seconds gauge
mapi_machine_created_timestamp_seconds{api_version="machine.openshift.io/v1beta1",name="machine-name",namespace="openshift-machine-api",node="unique-node-identifier",phase="Running",spec_provider_id="cloud-provider-identifier"} 1.589550152e+09
```

## Metrics about MachineSet resources

Similar to the Machine metrics, these entries show information about the
MachineSets that are currently observed by the MAO. In this example you can
see that there is 1 MachineSet currently, see `mapi_machineset_items`. Each
MachineSet has corresponding `mapi_machine_set_created_timestamp_seconds`,
`mapi_machine_set_status_replicas`, `mapi_machine_set_status_replicas_available`,
and `mapi_machine_set_status_replicas_ready` entries. These individual metric
entries help to provide current information about the state of each MachineSet.

**Sample metrics**
```
# HELP mapi_machineset_items Count of machinesets at the apiserver
# TYPE mapi_machineset_items gauge
mapi_machineset_items 1
# HELP mapi_machine_set_status_replicas Information of the mapi managed Machineset's status for replicas
# TYPE mapi_machine_set_status_replicas gauge
mapi_machine_set_status_replicas{name="machineset-name",namespace="openshift-machine-api"} 1
# HELP mapi_machine_set_status_replicas_available Information of the mapi managed Machineset's status for available replicas
# TYPE mapi_machine_set_status_replicas_available gauge
mapi_machine_set_status_replicas_available{name="machineset-name",namespace="openshift-machine-api"} 1
# HELP mapi_machine_set_status_replicas_ready Information of the mapi managed Machineset's status for ready replicas
# TYPE mapi_machine_set_status_replicas_ready gauge
mapi_machine_set_status_replicas_ready{name="machineset-name",namespace="openshift-machine-api"} 1
# HELP mapi_machineset_created_timestamp_seconds Timestamp of the mapi managed Machineset creation time
# TYPE mapi_machineset_created_timestamp_seconds gauge
mapi_machineset_created_timestamp_seconds{api_version="machine.openshift.io/v1beta1",name="ocp-cluster-rndpg-worker-us-east-2a",namespace="openshift-machine-api"} 1.589550153e+09
```

## Metrics about the Prometheus collectors

These values show the state of the Prometheus collectors internal to the
operator.

**Sample metrics**
```
# HELP mapi_mao_collector_up Machine API Operator metrics are being collected and reported successfully
# TYPE mapi_mao_collector_up gauge
mapi_mao_collector_up{kind="mapi_machine_items"} 1
mapi_mao_collector_up{kind="mapi_machineset_items"} 1
```

In addition, Prometheus provides some default metrics about the internal state
of the running process and the metric collection. You can find more information
about these metric names and their labels through the following links:

* [Prometheus documentation, Standard and runtime collectors](https://prometheus.io/docs/instrumenting/writing_clientlibs/#standard-and-runtime-collectors)
* [Prometheus client Go language collectors](https://github.com/prometheus/client_golang/blob/master/prometheus/go_collector.go)
* [Prometheus client HTTP collectors](https://github.com/prometheus/client_golang/blob/master/prometheus/promhttp/http.go)

## Machine API error rate for provider

These values show errors returned by cloud provider APIs.

For example:
- Invalid provider configurations not identified by Machine API static validation
- Timeouts and internal errors on the server side

**Sample metrics**
```
# Typical format:
# mapi_instance_<operation>_failed{name=<machine-name>,namespace=openshift-machine-api,reason=<failure cause>} <number of occurences>
# Examples:
mapi_instance_create_failed{name=machine-1,namespace=openshift-machine-api,reason="Unknown region 'us-central4'"} 1
mapi_instance_update_failed{name=machine-2,namespace=openshift-machine-api,reason="Unexpected response return code: 500"} 2
mapi_instance_delete_failed{name=machine-3,namespace=openshift-machine-api,reason="Timeout waiting for response"} 5
```

[Demo](https://user-images.githubusercontent.com/32226600/87791648-e72b6900-c842-11ea-90b7-4967b0d06fb5.gif)

## Metrics about MachineHealthCheck resources

When using MachineHealthChecks, metrics are available from the `machine-api-controllers` Pod on the
default metrics port(`8083`) for the `machine-healthcheck-controller` container.

The `mapi_machinehealthcheck_nodes_covered` metric describes the number of Nodes that are currently
being monitored by `machine-healthcheck-controller`.

The `mapi_machinehealthcheck_remediation_success_total` metric gives a total count of the successful
remediation performed by a MachineHealthCheck.

The `name` label in these metric refers to the name of the MachineHealthCheck that is being reported.
The `namespace` label refers to the owning namespace of the MachineHealthCheck.

**Sample metrics**
```
# HELP mapi_machinehealthcheck_nodes_covered Number of nodes covered by MachineHealthChecks
# TYPE mapi_machinehealthcheck_nodes_covered gauge
mapi_machinehealthcheck_nodes_covered{name="machine-api-termination-handler",namespace="openshift-machine-api"} 0
mapi_machinehealthcheck_nodes_covered{name="mhc-1",namespace="openshift-machine-api"} 1
# HELP mapi_machinehealthcheck_remediation_success_total Number of successful remediations performed by MachineHealthChecks
# TYPE mapi_machinehealthcheck_remediation_success_total counter
mapi_machinehealthcheck_remediation_success_total{name="mhc-1",namespace="openshift-machine-api"} 1
```
