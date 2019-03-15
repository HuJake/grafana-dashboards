# grafana-dashbords
This repository holds some curated Grafana dashboards that will work against the Sysdig SaaS backend.

In order to get started you need Sysdig agents installed in your k8s cluster as daemonsets. 
Instructions found here:
https://sysdigdocs.atlassian.net/wiki/spaces/Platform/pages/192348345/Agent+Install+Kubernetes+GKE+OpenShift+IBM

The Sysdig backend has a PromQL service endpoint exposed.

# Use Cases
This will allow users to run PromQL queries on Prometheus metrics. For instance if you want to see Cluster CPU Usage, you can run a PromQL query as follows:
```
sum(kube_pod_container_resource_requests_cpu_cores{node=~"$node"}) / sum(kube_node_status_allocatable_cpu_cores{node=~"$node"})
```
Or Number of Nodes in a cluster:
```
sum(kube_node_info{node=~"$node"})
```

This allows for users of Sysdig to perform math on metrics using PromQL

A few caveats:
1. Works for SaaS backend only
2. Can use only Grafana and can only run queries on Prometheus metrics (for now)
3. The service is in Alpha state.
4. Currently we do not support histograms using PromQL + Grafana and the Sysdig backend
5. We do not support Prometheus scrape configs (so you will be missing metrics like job, instance, up)
6. The label names might be a bit different as we do not support relabeling rules or recording rules.
7. You will have to increase the agent metric count limit to 3000

That said we are continually improving the backend PromQL service endpoint.


Once agents are installed they should automatically begin scraping Prometheus metrics and publishing it to the Sysdig backend

In order to get started with using these Grafana dashboards you will need to add a Prometheus data source to your Grafana instance.
In this example I have a Grafana instance running locally in a docker container.

Create a json file with the follwoing contents:
```
{
    "name": "Sysdig PromQL (alpha)",
    "orgId": 1,
    "type": "prometheus",
    "access": "proxy",
    "url": "https://app.sysdigcloud.com/prometheus",
    "basicAuth": false,
    "withCredentials": false,
    "isDefault": false,
    "editable": true,
    "jsonData": {
        "httpHeaderName1": "Authorization"
    },
    "secureJsonData": {
        "httpHeaderValue1": "Bearer <Sysdig monitor API token>"
    }
}
```

The Sysdig Monitor API token can be retrieved from the Sysdig Monitor UI under Settings -> User Profile

Then run the following command to add a new datasource to Grafana:
```
curl -u <grafana admin user>:<grafana admin password> -H "Content-Type: application/json" http://<grafana ip address>:3000/api/datasources -XPOST -d @<filename>.json
```

For the node-exporter dashboard the following exporter needs to be installed:

Node Exporter 0.17.0 – https://github.com/prometheus/node_exporter

For kubernetes cluster dashboard the following exporters need to be installed:

Node Exporter 0.17.0 – https://github.com/prometheus/node_exporter
Kube State Metrics Exporter 1.5.0 – https://github.com/kubernetes/kube-state-metrics


# Add a dashboard
In order to add a dahsboard download the json file and POSTit using the following command:
```
curl -X POST -H "Authorization: Bearer <Sysdig Monitor API token>" -H "Content-Type: application/json" -d @<dashboard_file>.json
```

If you run into any issues please post an issue in this repo
