# Chart-post-install-hook

Are you using Helm Charts to deploy you application in Kubernetes. 
If you have dependency on other services like `mongodb`, `postgresql` or `redis` and you want to modify it to add more users or create multiple database.
You can use Kubernetes Jobs with ```"helm.sh/hook": post-install```

In this example I am creating 2 users in MongoDb deployment using  post-install hook.
See the [create-user.yaml](/templates/hooks/create-user.yaml) and [mongodb-setup-scripts.yaml](/templates/mongodb-setup-scripts.yaml) file.

## Prerequisites Details

* Kubernetes 1.6+

## Chart Details
This chart will do the following:

* Deploy Express-Mongo-CRUD Application
* Deploy a MongoDB database using the stable/mongodb chart

## Installing the Chart

To install the chart with the release name `Node App`:

```bash
$ helm install --name hook ./
```

## To pull docker images from Private Docker Registry
```bash
# Create a kubectl secret to pull from private docker registry
$ kubectl create secret docker-registry regsecret --docker-server=https://my.docker.registry --docker-username=<USER> --docker-password=<API_KEY> --docker-email=<EMAIL>
```

Once created, Use it to install Charts
```bash
$ helm install --name hook --set imagePullSecrets=regsecret ./
```


## Configuration

The following table lists the configurable parameters of the Node App chart and their default values.

|         Parameter         |           Description             |                         Default                          |
|---------------------------|-----------------------------------|----------------------------------------------------------|
| `initContainerImage`           | Init Container Image       |     `alpine:3.6`                                           |
| `replicaCount`            | Replica count for Node App deployment| `1`                                                |
| `imagePullSecrets`        | Docker registry pull secret       |                                                          |
| `ingress.enabled`           | If true, Node App Ingress will be created | `false` |
| `ingress.annotations`       | Node App Ingress annotations     | `{}` |
| `ingress.hosts`             | Node App Ingress hostnames       | `[]` |
| `ingress.tls`               | Node App Ingress TLS configuration (YAML) | `[]` |
| `db.name` | Mongodb Database name | `test`   |
| `db.adminUser`            | Mongodb Database Admin user| `admin`                                                |
| `db.adminPassword`         | Mongodb Database Admin password            | `password`                                           |
| `db.nodeUser`    | Mongodb Database Node user                   | `node`                |
| `db.nodePassword`       | Mongodb Database Node password                     |  `password`                                         |
| `image.repository`| Container repository   | `jainishshah17/express-mongo-crud` |
| `image.tag`| Container tag | `1.0.0` |
| `image.pullPolicy` | Container pull policy | `IfNotPresent`   |
| `service.type` | Node App service type | `LoadBalancer`   |
| `service.internalPort` | Node App service internal port | `3000`   |
| `service.externalPort` | Node App service external port | `80`   |
| `persistence.mountPath` | Node App persistence volume mount path | `"/var/log/app"`   |
| `persistence.enabled` | Node App persistence volume enabled | `true`   |
| `persistence.existingClaim` | Provide an existing PersistentVolumeClaim | `nil`   |
| `persistence.storageClass` | Storage class of backing PVC | `nil (uses alpha storage class annotation)`   |
| `persistence.size` | Node App persistence volume size | `2Gi`   |
| `resources.requests.memory` | Node App initial memory request  |      |
| `resources.requests.cpu`    | Node App initial cpu request     |      |
| `resources.limits.memory`   | Node App memory limit            |      |
| `resources.limits.cpu`      | Node App cpu limit               |      |

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`.

### Ingress and TLS
To get Helm to create an ingress object with a hostname, add these two lines to your Helm command:
```
helm install --name hook \
  --set ingress.enabled=true \
  --set ingress.hosts[0]="jainish.company.com" \
  --set service.type=NodePort \
  --set nginx.enabled=false \
  ./
```

If your cluster allows automatic creation/retrieval of TLS certificates (e.g. [kube-lego](https://github.com/jetstack/kube-lego)), please refer to the documentation for that mechanism.

To manually configure TLS, first create/retrieve a key & certificate pair for the address(es) you wish to protect. Then create a TLS secret in the namespace:

```console
kubectl create secret tls chart-example-tls --cert=path/to/tls.cert --key=path/to/tls.key
```

Include the secret's name, along with the desired hostnames, in the Node App Ingress TLS section of your custom `values.yaml` file:

```
  ingress:
    ## If true, Node Aapp Ingress will be created
    ##
    enabled: true

    ## Node Aapp Ingress hostnames
    ## Must be provided if Ingress is enabled
    ##
    hosts:
      - jainish.domain.com
    annotations:
      kubernetes.io/tls-acme: "true"
    ## Node Aapp Ingress TLS configuration
    ## Secrets must be manually created in the namespace
    ##
    tls:
      - secretName: chart-example-tls
        hosts:
          - jainish.domain.com
```

## Useful links
* [Source of Example Application](https://github.com/jainishshah17/express-mongo-crud) 