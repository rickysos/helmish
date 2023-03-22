# Recursion Generic Web Application Helm Chart

This chart is meant to be a generic helm chart for deploying a standard web application
into Recursion's clusters. You will configure your deployment in your application's repository or in
`eng-infrastructure` by adding a `values.yaml` file specifying information specific to your deployment, and helm will
provision the rest. Below is a list of configurable options:

| Parameter                            | Description                                                                                                                                                                                                                    | Default                                                                                                  |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| `auth.enabled`                       | Should this app provide its own authentication serivce?                                                                                                                                                                        | `false`                                                                                                  |
| `config`                             | A list of non-secret KEY/VALUE environment variables as a configmap                                                                                                                                                            | `{}`                                                                                                     |
| `configome`                          | The contents of the production configome mounted as a configmap                                                                                                                                                                | `""`                                                                                                     |
| `custom.volumes`                     | A list of custom volumes to mount to the main container. A typical use case would be to mount secrets as volumes.                                                                                                              | `[]`                                                                                                     |
| `custom.volumeMounts`                | A list of custom volume mounts to attach to the main container. Often used in conjunction with volumes specifying where they will reside on the host container.                                                                | `[]`                                                                                                     |
| `externalSecrets.relativeVaultPaths` | A list of strings. Within a cluster/app combination path, what paths should be preserved?.                                                                                                                                     | `["/", "/db" ]`                                                                                          |
| `externalSecrets.enabled`            | A boolean flag determining whether `kubernetes-external-secrets` should be used to sync secrets from `vault` to `kubernetes` `secrets`.                                                                                        | `false`                                                                                                  |
| `hpa.enabled`                        | Enable [Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/).                                                                                                               | `false`                                                                                                  |
| `hpa.kind`                           | Set object type for HPA to monitor;                                                                                                                                                                                            | `Deployment`                                                                                             |
| `hpa.minReplicas`                    | Minimum number of Replicas the HPA will scale down to.                                                                                                                                                                         | `""`                                                                                                     |
| `hpa.maxReplicas`                    | Maximum number of Replicas the HPA will scale up to.                                                                                                                                                                           | `""`                                                                                                     |
| `hpa.targetCPUUtilizationPercentage` | Target CPU utilization a pod can hit before scaling up.                                                                                                                                                                        | `null`                                                                                                   |
| `hpa.targetMemoryUtilizationPercentage` | Target Memory utilization a pod can hit before scaling up.                                                                                                                                                                  | `null`                                                                                                   |
| `image.args`                         | Arguments to send to the main container's image                                                                                                                                                                                | `[]`                                                                                                     |
| `image.repository`                   | The repository and name of your image.                                                                                                                                                                                         | `nginx`                                                                                                  |
| `image.tag`                          | The tag of your image version.                                                                                                                                                                                                 | `latest`                                                                                                 |
| `image.pullPolicy`                   | Should your image be pulled everytime?                                                                                                                                                                                         | `always`                                                                                                 |
| `ingress.enabled`                    | Should this app be routable?                                                                                                                                                                                                   | `false`                                                                                                  |
| `ingress.hosts`                      | A list of urls where this service can be found.                                                                                                                                                                                | `example.ops.rxrx.io`                                                                                    |
| `ingress.annotations`                | Additional annotations.                                                                                                                                                                                                        | {}                                                                                                       |
| `ingress.labels`                     | Additional labels.                                                                                                                                                                                                             | {}                                                                                                       |
| `ingress.path`                       | Path where the app will be reachable.                                                                                                                                                                                          | '/'                                                                                                      |
| `service.type`                       | Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them                                                                                                         | `ClusterIP`                                                                                              |
| `service.annotations`                | Additional annotations                                                                                                                                                                                                         | {}                                                                                                       |
| `service.labels`                     | Additional labels                                                                                                                                                                                                              | {}                                                                                                       |
| `service.port`                       | The port the ingress will use.                                                                                                                                                                                                 | `80`                                                                                                     |
| `service.targetPort`                 | The port your application is exposed on.                                                                                                                                                                                       | `5000`                                                                                                   |
| `secrets.enabled`                    | Does this app need access to any secrets?                                                                                                                                                                                      | `true`                                                                                                   |
| `replicas`                           | The number of replicas to spin up                                                                                                                                                                                              | `2`                                                                                                      |
| `replicaSetRetentionLimit`           | The number of historical replicaSet objects to retain                                                                                                                                                                          | `2`                                                                                                      |
| `image.name`                         | The full image name to deploy example "gcr.io/eng-infrastructure/platelet-proxy"                                                                                                                                               | `nginx`                                                                                                  |
| `resources`                          | A dictionary defining the requests & limits of CPU & Mem of the service                                                                                                                                                        | `resources:{ {"limits": "memory": "2Gi", "cpu": "500m"}, "requests": {"memory": "1Gi", "cpu": "100m"} }` |

#### Adding health check probes.

```yaml
livenessProbe:
  httpGet:
    path: /liveness
    port: 80
  initialDelaySeconds: 3
  periodSeconds: 3

readinessProbe:
  httpGet:
    path: /readiness
    port: 80
  initialDelaySeconds: 3
  periodSeconds: 3

startupProbe:
  httpGet:
    path: /startup
    port: 80
  initialDelaySeconds: 3
  periodSeconds: 3
```

#### postStart, This hook is sent immediately after a container is created.

#### preStop, This hook is called immediately before a container is terminated.

```yaml
lifecycle:
  postStart:
    exec:
      command:
        ["/bin/sh", "-c", "echo Hello from the postStart handler > hello.txt"]
  preStop:
    exec:
      command:
        ["/bin/sh", "-c", "echo Hello from the preStop handler > hello.txt"]
```

### Provide a list of initContainers, in order

#### This will share the env variables from the config and secret blocks by default

```yaml
initContainers:
  - name: init-container-0
    image:
      repository: gcr.io/eng-infrastructure/roadie
      tag: latest
  - name: init-container-1
    image:
      repository: busybox
      tag: 2.4.1
    command: "['echo', 'bar']"
    env:
      TEST: "true"
```

### To add a secrect to your app created outside of this chart

#### Example volumeMount

```yaml
volumeMounts:
  name: my-volume-name
  mountPath: /path/to/mount
  readOnly: boolean
  secretName: my-secret-name
```

# Example

TODO create new example
this is the only kubernetes provisioning file that exists for this service, there are no other files needed for deployment.
