# 14-jobs-cronjobs

# MySQL StatefulSet with Backup Job

## 1. Create MySQL Secrets
```bash
# Generate base64 encoded values
echo -n "rootpassword" | base64     # For MYSQL_ROOT_PASSWORD
echo -n "mydb" | base64             # For MYSQL_DATABASE
echo -n "myuser" | base64           # For MYSQL_USER
echo -n "mypassword" | base64       # For MYSQL_PASSWORD

# Create mysql-secret.yaml
kubectl apply -f mysql-secret.yaml
```

# (Alternative) Create mysql-secret from command line
```bash
kubectl create secret generic mysql-secret --from-literal=root-password=rootpassword --from-literal=database=mydb --from-literal=user=myuser --from-literal=password=mypassword
```

## 2. Deploy MySQL StatefulSet
```bash
# Apply StatefulSet, Headless Service and PV
kubectl apply -f mysql-sts.yaml

# Wait for pods to be ready
kubectl get pods -w -l app=mysql

# Verify StatefulSet
kubectl get sts mysql
kubectl get pvc -l app=mysql
kubectl get svc mysql-service

# Verify MySQL is running
kubectl exec -it mysql-0 -- mysql -u root -p
# After it you will prompt for password, use the password you created in the mysql-secret.yaml file

# Select the database you created in the mysql-secret.yaml file
use mydb;

# Create a test table (inside the mysql container)
CREATE TABLE testdb (id INT, name VARCHAR(255));

# Verify the database was created (inside the mysql container)
SHOW TABLES;
```

## 4. Execute Backup Job
```bash
# Create backup job
kubectl apply -f backup-job.yaml

# Monitor job status
kubectl get jobs mysql-backup
kubectl get pods -l job-name=mysql-backup

# Check backup logs
kubectl logs -l job-name=mysql-backup

# Verify backup file
kubectl exec -it $POD_NAME -- ls -l /backup
```

## Cleanup
```bash
# Delete all resources
kubectl delete job mysql-backup
kubectl delete sts mysql
kubectl delete svc mysql-service
kubectl delete pvc mysql-backup-pvc
kubectl delete pv mysql-backup-pv
kubectl delete secret mysql-secrets

# Verify cleanup
kubectl get all,pv,pvc,secrets -l app=mysql
```