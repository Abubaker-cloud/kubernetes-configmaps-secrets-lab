# Kubernetes ConfigMaps & Secrets — Hands-on Lab

## Overview

This lab demonstrates how to manage application configuration and sensitive data in Kubernetes using ConfigMaps and Secrets.

---

## Part 1: ConfigMaps

### 1. Create ConfigMap

```bash
kubectl create configmap app-config \
  --from-literal=DATABASE_URL=mysql://db:3306/mydb \
  --from-literal=LOG_LEVEL=INFO \
  --from-literal=MAX_CONNECTIONS=100
```

### 2. Verify ConfigMap

```bash
kubectl get configmaps
kubectl describe configmap app-config
```

### 3. Update ConfigMap

Updated `LOG_LEVEL` to `DEBUG`.

### 4. Deploy Application

Created a Deployment that consumes ConfigMap values through environment variables and mounts the ConfigMap as files.

- Environment variables using `configMapKeyRef`.
- Configuration files using ConfigMap volumes.

### 5. Verify Mounted Configuration

```bash
kubectl exec -it <pod-name> -- ls /etc/app-config
kubectl exec -it <pod-name> -- cat /etc/app-config/LOG_LEVEL
```

---

## Part 2: Secrets

### 1. Create Secret

```bash
kubectl create secret generic db-secret \
  --from-literal=DB_PASSWORD="supersecretpassword"
```

### 2. Verify Secret

```bash
kubectl get secrets
kubectl describe secret db-secret
```

### 3. Create Deployment

The Deployment uses `secretKeyRef` to inject the Secret into the container as an environment variable.

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_PASSWORD
```

### 4. Apply Deployment

```bash
kubectl apply -f secret-deployment.yaml
```

### 5. Verify Secret Inside Pod

```bash
kubectl exec -it <pod-name> -- env
```

Expected variable:

```text
DB_PASSWORD=<secret-value>
```

### 6. Inspect Pod Configuration

```bash
kubectl get pod <pod-name> -o yaml
```

The Pod references the Secret rather than directly containing the secret value in its specification.

---

## Cleanup

```bash
kubectl delete deployment secret-demo
kubectl delete secret db-secret
```

---

## Key Concepts

| Feature | ConfigMap | Secret |
|---|---|---|
| Purpose | Configuration data | Sensitive data |
| Examples | URLs, settings, log levels | Passwords, tokens, keys |
| Environment injection | `configMapKeyRef` | `secretKeyRef` |
| Volume mounting | Supported | Supported |

## Security Notes

- Base64 is encoding, not encryption.
- Secrets are not automatically encrypted at rest in every Kubernetes configuration.
- Use RBAC to restrict access to Secrets.
- Avoid committing real secrets to GitHub.

---

## Skills Practiced

- Creating ConfigMaps.
- Managing application configuration.
- Updating ConfigMaps.
- Injecting configuration into Pods.
- Mounting ConfigMaps as volumes.
- Creating Kubernetes Secrets.
- Injecting Secrets into containers.
- Inspecting Kubernetes resources.
- Basic Kubernetes secret-management practices.
