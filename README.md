# helm-runai


**** Deploy MinIO ****
Run the following Helm command to deploy MinIO:


helm install my-minio-app . -n runai-test --set inferenceServiceUrl=http://triton.runai-test.svc.cluster.local


Expected output:
NAME: my-minio-app
LAST DEPLOYED: Tue Jun 10 17:57:55 2025
NAMESPACE: runai-test
STATUS: deployed
REVISION: 1
TEST SUITE: None


Verify Deployment

Check that the pods are running:

kubectl get pods -n runai-test

Expected output:

NAME                                      READY   STATUS    RESTARTS   AGE
minio-5f89c57d9b-r677m                    1/1     Running   0          80s


Check the services:


bashkubectl get svc -n runai-test


Check the persistent volume claims:

kubectl get pvc -n runai-test

Accessing MinIO

Web Console Access (Recommended for Users)

Port Forward to Local Machine:

kubectl port-forward svc/minio 9090:9090 -n runai-test

Open Browser:
Navigate to: http://localhost:9090
Login Credentials:

Username: minioadmin
Password: minioadmin123



API Access
For programmatic access:
bashkubectl port-forward svc/minio 9000:9000 -n runai-test
API Endpoint: http://localhost:9000
Internal Cluster Access
Other services within the cluster can access MinIO at:

API: http://minio.runai-test.svc.cluster.local:9000
Console: http://minio.runai-test.svc.cluster.local:9090
