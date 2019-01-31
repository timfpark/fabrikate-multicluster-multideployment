# fabrikate-multicluster-multideployment

A prototype for how a multi-cluster deployment of a set of microservices (only one for illustration now), with multiple deployments with traffic splitting per microservice, could be specified with a Fabrikate definition within a cluster that utilizes Istio as a service mesh. 

## Design

The services directory houses all of the high level deployment definitions for a set of microservices.  Within each service, the top level component establishes the global routing rules between deployments of the service in `service-chart` (90% stable, 10% canary in this example), and then defines one or more deployments.  In this case there are two, `canary` and `stable`, which utilize a common chart (in `./deployment-chart`).  The top level service chart also defines a common ConfigMap for the two underlying deployments, the assumption here being that the config applied to `canary` and `stable` is invariant within a particular cluster deployment (ie. `prod-east` or `prod-west`).

## Generation

The final resource manifests for the overall deployment are generated in multiple passes, one for each cluster environment.  These are overlaid in the generated directory to compose the final set of resource manifests for both cluster environment. A git-path is then specified to Flux to select the correct environment.

Alternatively, you can generate each environment one by one and checking it into a correspondingly named branch of the manifest repo and pass this branch to Flux. 

## Example

```
$ fab install
$ fab generate prod-east
$ fab install
$ fab generate prod-west
```

which generates:

```
generated
    prod-east
        cloud-native
            elasticsearch-fluentd-kibana
            istio
            kured
            prometheus-grafana
        services
            simple-service
                canary
                stable
    prod-west
        cloud-native
            elasticsearch-fluentd-kibana
            istio
            kured
            prometheus-grafana
        services
            simple-service
                canary
                stable
```

Looking at `generated/prod-west/services/simple-service/simple-service.yaml` we see it has the west storage account:

```
---
# Source: config-map/templates/configMap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: simple-service-configmap
  namespace: services
data:
    STORAGE_ACCOUNT: west-storage
```

while `generated/prod-east/services/simple-service/simple-service.yaml` has the east storage account:

```
---
# Source: config-map/templates/configMap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: simple-service-configmap
  namespace: services
data:
    STORAGE_ACCOUNT: east-storage
```

## Background

* [Canary deployments with Istio](https://istio.io/blog/2017/0.1-canary/)
* [Fabrikate](https://github.com/Microsoft/fabrikate)
