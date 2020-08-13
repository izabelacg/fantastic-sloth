# fantastic-sloth

Attempts to reproduce Concourse state described in the issue [#5404](https://github.com/concourse/concourse/issues/5404).

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

## Deployments

1. Release/Deployment named `concourse-loadtest-icg`

    - Chart: concourse-11.2.2
    - Passing the following configuration to override values in the chart: [values-concourse-loadtest-icg.yml](values-concourse-loadtest-icg.yml)

1. Release/Deployment named `concourse-icg`

    - Chart: concourse-11.2.2
    - Passing the following configuration to override values in the chart: [values-concourse-icg.yml](values-concourse-icg.yml)

1. Release/Deployment named `concourse-aaa`

    - Chart: concourse-11.4.0
    - Passing the following configuration to override values in the chart: [values-concourse-aaa.yml](values-concourse-aaa.yml)
    - Datadog dashboard direct [link](https://p.datadoghq.com/sb/2x0hq9m0hhctg8bs-8ea44961896ba6e7f04bb84a172cc88f)
    - Datadog dashboard direct for system stats [link](https://p.datadoghq.com/sb/2x0hq9m0hhctg8bs-5ebd02a1fe0b800b883b05c805807c3a)

1. Release/Deployment named `workaround`

    - Chart: concourse-11.4.0
    - Passing the following configuration to override values in the chart: [values-workaround.yml](values-workaround.yml)

## Useful commands:

```bash
gcloud container clusters get-credentials cluster-1

kubectl config use-context cluster-1

helm list

# Install release

export RELEASE_NAME="concourse-icg"
export CHART=concourse-chart
export DATADOG_API_KEY=
export DATADOG_APP_KEY=

helm install --wait --name $RELEASE_NAME --namespace $RELEASE_NAME --values values-$RELEASE_NAME.yml $CHART

helm upgrade --install --recreate-pods --namespace $RELEASE_NAME --values values-$RELEASE_NAME.yml $RELEASE_NAME $CHART

export POD_NAME=$(kubectl get pods --namespace $RELEASE_NAME -l "app=$RELEASE_NAME-web" -o jsonpath="{.items[0].metadata.name}") && \
kubectl port-forward --namespace $RELEASE_NAME $POD_NAME 8080:8080

# Delete release

helm delete $RELEASE_NAME
helm del --purge $RELEASE_NAME
k delete namespace $RELEASE_NAME-main
k delete namespace $RELEASE_NAME
k delete pvc -l app=$RELEASE_NAME-worker #according to https://github.com/concourse/concourse-chart/blob/master/README.md#cleanup-orphaned-persistent-volumes

```
