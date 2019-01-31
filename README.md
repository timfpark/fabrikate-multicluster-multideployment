# fabrikate-multicluster-multideployment

A prototype for how a multi-cluster deployment of a set of microservices (only one for illustration now), with multiple deployments with traffic splitting per microservice, could be specified with a Fabrikate definition within a cluster that utilizes Istio as a service mesh. 

## Design

The services directory houses all of the high level deployment definitions for a set of microservices.  Within each service, the top level component establishes the global routing rules between deployments of the service in `routing` (90% stable, 10% canary in this example), and then defines one or more deployments.  In this case there are two, `canary` and `stable`, which utilize a common chart (in `./service-chart`).  The top level service component also defines a common ConfigMap for the two underlying deployments, the assumption here being that the config applied to `canary` and `stable` is invariant within a particular cluster deployment (ie. `prod-east` or `prod-west`).

## Generation

The final resource manifests for the overall deployment are generated in multiple passes, one for each cluster environment.  These are overlaid in the generated directory to compose the final set of resource manifests for both cluster environment. A git-path is then specified to Flux to select the correct environment.

Alternatively, you can generate each environment one by one and checking it into a correspondingly named branch of the manifest repo and pass this branch to Flux. 

## Example

```
$ fab generate prod-east
$ fab generate prod-west
```

which generates:

```
generated
    prod-east
        elasticsearch-fluentd-kibana
        prometheus-grafana
        services
            simple-service
                canary
                stable
    prod-west
        canary
            manifests
                simple-service
        stable
            manifests
                simple-service
```

## Background

* [Canary deployments with Istio](https://istio.io/blog/2017/0.1-canary/)
* [Fabrikate](https://github.com/Microsoft/fabrikate)
