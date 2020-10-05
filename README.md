# fantastic-sloth

Attempt to reproduce Concourse state described in the issue [#5404](https://github.com/concourse/concourse/issues/5404).

- Using the [helm chart](https://github.com/concourse/concourse-chart) to deploy Concourse
- Deploying it to `cluster-1` ([See](https://github.com/concourse/hush-house#gathering-acccess-to-the-cluster) for gathering access to the cluster)
- Sending metrics to Datadog (using personal free account)
	* Added the following to [requirements.yaml](https://github.com/concourse/concourse-chart/blob/master/requirements.yaml):

	```yaml
    - name: datadog
      version: 1.39.5
      repository: https://kubernetes-charts.storage.googleapis.com/
	```

- Using [drills pipeline](https://github.com/concourse/drills/blob/master/longevity/lidar-test) to create build history

## Deployment

1. Release/Deployment named `workaround`

    - Chart: concourse-11.4.1
    - Passing the following configuration to override values in the chart: [values-workaround.yml](values-workaround.yml)
    - Datadog dashboard direct [link](https://p.datadoghq.com/sb/2x0hq9m0hhctg8bs-8ea44961896ba6e7f04bb84a172cc88f)
    - Datadog dashboard direct for system stats [link](https://p.datadoghq.com/sb/2x0hq9m0hhctg8bs-5ebd02a1fe0b800b883b05c805807c3a)
    - Using newer version of Postgresql chart:

    ```yaml
    - name: postgresql
      version: 8.6.4
      repository: https://kubernetes-charts.storage.googleapis.com/
      condition: postgresql.enabled
    ```

## Useful commands:

```bash
gcloud container clusters get-credentials cluster-1

kubectl config use-context cluster-1

helm list

# Install release

export RELEASE_NAME="workaround"
export CHART=concourse-chart

helm install --wait --name $RELEASE_NAME --namespace $RELEASE_NAME --values values-$RELEASE_NAME.yml $CHART

helm upgrade --install --namespace $RELEASE_NAME --values values-$RELEASE_NAME.yml $RELEASE_NAME $CHART

export POD_NAME=$(kubectl get pods --namespace $RELEASE_NAME -l "app=$RELEASE_NAME-web" -o jsonpath="{.items[0].metadata.name}") && \
kubectl port-forward --namespace $RELEASE_NAME $POD_NAME 8080:8080

# Delete release

helm delete $RELEASE_NAME
helm del --purge $RELEASE_NAME && \
k delete pvc -l app=$RELEASE_NAME-worker && \
k delete namespace $RELEASE_NAME-main && \
k delete namespace $RELEASE_NAME
 
#according to https://github.com/concourse/concourse-chart/blob/master/README.md#cleanup-orphaned-persistent-volumes

```
