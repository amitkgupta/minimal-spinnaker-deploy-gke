### Create and Connect to GKE Cluster

```
$ gcloud container clusters create CLUSTER_NAME \
  --cluster-version=latest \
  --num-nodes=2 \
  --zone=GCP_ZONE \
  --project=GCP_PROJECT \
  --machine-type=n1-standard-2
$ gcloud container clusters get-credentials CLUSTER_NAME \
  --zone GCP_ZONE \
  --project GCP_PROJECT
```

### Deploy Halyard Workstation and Service Account

```
$ kubectl apply -f https://raw.githubusercontent.com/amitkgupta/minimal-spinnaker-deploy-gke/master/spinnaker-ns.yaml
$ kubectl apply -f https://raw.githubusercontent.com/amitkgupta/minimal-spinnaker-deploy-gke/master/halyard.yaml
$ kubectl exec halyard-workstation -nspinnaker -it -- bash
```

### Configure Halyard to use Halyard Service Account (for deploy only)

```
workdir> hal config provider kubernetes account add ACCOUNT_NAME \
         --provider-version v2 \
         --service-account true
workdir> hal config deploy edit \
         --type distributed \
         --account-name ACCOUNT_NAME \
         --bootstrap-only
```

### Deploy Minio

```
$ kubectl apply -f https://raw.githubusercontent.com/amitkgupta/minimal-spinnaker-deploy-gke/master/minio-ns.yaml
$ kubectl apply -f https://raw.githubusercontent.com/amitkgupta/minimal-spinnaker-deploy-gke/master/minio.yaml
```

### Configure Spinnaker to use Minio for Storage

```
workdir> mkdir -p ~/.hal/default/profiles
workdir> echo 'spinnaker.s3.versioning: false' > ~/.hal/default/profiles/front50-local.yml
workdir> hal config storage s3 edit \
         --endpoint http://minio-service.minio:9000 \
         --access-key-id minio \
         --secret-access-key minio123 \
         --path-style-access true
workdir> hal config storage edit --type s3
```

### Deploy Spinnaker

```
workdir> hal config version edit --version 1.15.1
workdir> hal deploy apply
```

### Connect

```
$ kubectl port-forward svc/spin-deck 9000 -nspinnaker &
$ kubectl port-forward svc/spin-gate 8084 -nspinnaker &
$ open http://localhost:9000
```
