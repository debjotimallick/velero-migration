# Velero Migration Process

## This document outlines the steps to migrate Kubernetes resources from one cluster to another using Velero backups, ensuring a smooth transition of backup data and configurations.

### Prerequisites
- Ensure you have access to both the source and destination Kubernetes clusters.
- Install Velero on both clusters.
- Ensure you have access to the backup storage location (e.g., S3 bucket) used by Velero.
- Ensure you have the necessary permissions to create and manage backups in both clusters.
- Ensure you have the Velero CLI installed on your local machine.
- Ensure you have the necessary credentials to access the backup storage location.

### Create an S3 bucket for Velero backups
1. Create an S3 bucket in your AWS account to store Velero backups. Update the same in the `velero-values.yaml` file.

### Install velero on both clusters
1. Create a generic secret named `velero-aws-credentials` in the `velero` namespace using the `aws-iam-creds` file on both the source and destination clusters, using your AWS credentials:
```bash
kubectl create secret generic velero-aws-credentials --namespace velero --from-file=cloud='.\aws-iam-creds'
```
2. Install Velero on both the source and destination clusters using the given values file:
```bash
helm repo add vmware-tanzu https://vmware-tanzu.github.io/helm-charts
helm install velero vmware-tanzu/velero -n velero --create-namespace -f velero-values.yaml
```

### Deploy sample workloads on the source cluster
1. Deploy sample workloads on the source cluster to test the migration process. You can use the following command to deploy a sample application:
```bash
kubectl apply -f workloads.yaml -n velero-poc
```
2. Verify that the sample workloads are running successfully on the source cluster:
```bash
kubectl get all -n velero-poc
```

### Create a backup of the source cluster
1. Create a backup of the source cluster using Velero:
```bash
velero backup create velero-poc-backup --include-namespaces velero-poc
```
2. Verify that the backup was created successfully:
```bash
velero backup get
velero backup describe velero-poc-backup
```

### Restore the backup on the destination cluster
1. Restore the backup on the destination cluster using Velero:
```bash
velero restore create --from-backup velero-poc-backup
```
2. Verify that the restore was successful:
```bash
velero restore get
velero restore describe <restore-name>
```

### Verify the restored workloads on the destination cluster
1. Verify that the restored workloads are running successfully on the destination cluster:
```bash
kubectl get all -n velero-poc
```