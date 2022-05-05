# minikube-grafana
Simple Grafana setup using minikube

# Run Grafana in Minikube (MacOS)

## Before you begin

1. Install minikube with hyperkit
1. Install [helm](https://helm.sh/docs/using_helm/#installing-helm)
1. Get grafana helm chart

    ```helm repo add grafana https://grafana.github.io/helm-charts```

    ```helm repo update```

## Before you fire minikube
1. Mount working directory
    ```minikube mount $(pwd):/host &```
1. Start minikube
    ```minikube start```
1. Start minikube dashboard
    ```minikube dashboard &```

## Start Loki
1. Open new terminal for Loki (optional)
1. Deploy Loki 
    ```helm upgrade --install loki grafana/loki```
1. Port forward 3100
    ```kubectl --namespace default port-forward service/loki 3100 &```

## Start Grafana
1. Open new terminal for Grafana (optional)
1. Install chart using my-release name
    ```helm install my-release grafana/grafana```
1. Get your 'admin' user password by running:

   ```kubectl get secret --namespace default my-release-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo```

1. The Grafana server can be accessed via port 80 on the following DNS name from within your cluster:

   ```my-release-grafana.default.svc.cluster.local```

   Get the Grafana URL to visit by running these commands in the same shell:

    ``` export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=my-release" -o jsonpath="{.items[0].metadata.name}")```

    ```kubectl --namespace default port-forward $POD_NAME 3000 &```

1. Login with the password from step 3 and the username: admin

## Install Promtail on the host machine

1. Open new terminal for Promtail client (optional)
1. Using brew
    ```brew install promtail```
1. Run Promtail
    ```promtail --config.file=./promtail/promtail-config.yaml```

## Troubleshooting

1. ```level=warn ts=2022-05-04T00:55:00.469477Z caller=client.go:349 component=client host=localhost:3100 msg="error sending batch, will retry" status=429 error="server returned HTTP status 429 Too Many Requests (429): Ingestion rate limit exceeded for user fake (limit: 4194304 bytes/sec) while attempting to ingest '9134' lines totaling '1048541' bytes, reduce log volume or contact your Loki administrator to see if the limit can be increased"```

    1. Crate sub-folder
        ```loki```
    1. Create file under the newly created sub-folder
        ```values.yaml```
    1. Add the following entries:
```yaml
config:
    limits_config:
        per_stream_rate_limit: 10MB
```
*
    4. Deploy/reload Loki using overrided values 
        ```helm upgrade --install loki grafana/loki -f loki/values.yaml```

## Install Nginx Ingress Controller

```minikube addons enable ingress```

Create an Ingress for Nginx service

```kubectl create -f baremetal/nginx-ingress.yaml```

Login to Grafana
user: admin
Run the following command to get the admin password
```kubectl get secret --namespace default my-release-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo```
     
## Install Prometheus

```helm repo add prometheus-community https://prometheus-community.github.io/helm-charts```

```helm install prometheus prometheus-community/prometheus```

## Enable NGINX controller metrics

```kubectl apply -f baremetal/nginx-metrics.yaml```


