0. Connect to a Kubernetes Cluster with Dynamic Storage Provisioning and Enough Resources
   - gcloud container clusters create CLUSTER_NAME \
     --cluster-version=latest \
     --num-nodes=2 \
     --zone=GCP_ZONE \
     --project=GCP_PROJECT \
     --machine-type=n1-standard-2
   - gcloud container clusters get-credentials CLUSTER_NAME \
     --zone GCP_ZONE \
     --project GCP_PROJECT
1. Setup Halyard Workstation and Service Account
   - reference: https://www.spinnaker.io/setup/install/halyard/
   - kubectl apply -f spinnaker-ns.yaml -f halyard.yaml
   - kubectl exec halyard-workstation -nspinnaker -it -- bash
2. Configure Halyard to use Halyard Service Account
   - reference: https://www.spinnaker.io/setup/install/providers/kubernetes-v2/
   - [from halyard workstation] hal config provider kubernetes account add ACCOUNT_NAME \
                                --provider-version v2 --service-account true
3. Configure Halyard to use Halyard Service Account for Spinnaker Deploy only
   - reference: https://www.spinnaker.io/setup/install/environment/#distributed-installation
   - [from halyard workstation] hal config deploy edit \
                                --type distributed --account-name ACCOUNT_NAME --bootstrap-only
4. Deploy Minio
   - reference: https://docs.min.io/docs/deploy-minio-on-kubernetes.html#minio-distributed-server-deployment
   - kubectl apply -f minio-ns.yaml -f minio.yaml
5. Configure Spinnaker to use Minio for Storage
   - reference: https://www.spinnaker.io/setup/install/storage/minio/
   - [from halyard workstation] mkdir -p ~/.hal/default/profiles
   - [from halyard workstation] echo 'spinnaker.s3.versioning: false' > ~/.hal/default/profiles/front50-local.yml
   - [from halyard workstation] hal config storage s3 edit \
                                --endpoint http://minio-service.minio:9000 \
                                --access-key-id minio \
                                --secret-access-key minio123 \
                                --path-style-access true
   - [from halyard workstation] hal config storage edit --type s3
6. Deploy Spinnaker
   - reference: https://www.spinnaker.io/setup/install/deploy/#pick-a-version
   - [from halyard workstation] hal config version edit --version 1.15.1
   - [from halyard workstation] hal deploy apply
7. Connect
   - kubectl port-forward svc/spin-deck 9000 -nspinnaker &
   - kubectl port-forward svc/spin-gate 8084 -nspinnaker &
   - open http://localhost:9000
