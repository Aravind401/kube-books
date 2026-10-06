# Phase 2.3 — ConfigMaps and Secrets

## 1. Why Configuration Matters

Container images should ideally contain application code and dependencies, while environment-specific configuration is supplied separately.

Example:

```text
Same image
   ↓
Development configuration
   ↓
Staging configuration
   ↓
Production configuration
```

This avoids rebuilding an image for every environment.

---

# 2. ConfigMap

A ConfigMap stores non-sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  LOG_LEVEL: info
```

Create:

```bash
kubectl apply -f configmap.yaml
```

Check:

```bash
kubectl get configmap
```

---

# 3. ConfigMap as Environment Variables

Example:

```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV
```

Inside the container:

```text
APP_ENV=production
```

---

# 4. Import an Entire ConfigMap

```yaml
envFrom:
  - configMapRef:
      name: app-config
```

All keys become environment variables.

Use this carefully because adding or changing keys can affect application behavior.

---

# 5. ConfigMap as a File

Example:

```yaml
volumes:
  - name: config
    configMap:
      name: app-config

containers:
  - name: app
    image: nginx
    volumeMounts:
      - name: config
        mountPath: /etc/app-config
```

The keys can appear as files inside the mounted directory.

---

# 6. ConfigMap Limitations

ConfigMap is not intended for confidential information.

Do not store:

- Passwords
- Private keys
- API secrets
- Database credentials

Use Secrets or external secret-management systems for sensitive information.

---

# 7. Secret

A Secret is a Kubernetes object designed to hold sensitive data.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: database-secret
type: Opaque
stringData:
  username: appuser
  password: change-me
```

Apply:

```bash
kubectl apply -f secret.yaml
```

Check:

```bash
kubectl get secrets
```

---

# 8. Secret Data and Base64

If you use:

```yaml
data:
  password: <base64-value>
```

the value is Base64-encoded.

Base64 is encoding, not encryption.

Do not assume:

```text
Base64 = Secure
```

It does not.

Using Kubernetes Secret objects should still be combined with appropriate encryption-at-rest, RBAC, access controls, and external secret management where required.

---

# 9. stringData

For convenience:

```yaml
stringData:
  username: appuser
  password: change-me
```

Kubernetes handles conversion into the Secret representation.

This is often easier when writing manifests.

---

# 10. Secret as Environment Variable

```yaml
env:
  - name: DB_USERNAME
    valueFrom:
      secretKeyRef:
        name: database-secret
        key: username

  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: database-secret
        key: password
```

---

# 11. Secret as Volume

Example:

```yaml
volumes:
  - name: db-secret
    secret:
      secretName: database-secret

containers:
  - name: app
    image: nginx
    volumeMounts:
      - name: db-secret
        mountPath: /etc/secrets
        readOnly: true
```

The Secret data can be exposed as files.

---

# 12. ConfigMap vs Secret

| Feature | ConfigMap | Secret |
|---|---|---|
| Normal configuration | Yes | Possible but not primary purpose |
| Sensitive data | No | Yes |
| Environment variables | Yes | Yes |
| Volume mount | Yes | Yes |
| Base64 representation | Depending on object encoding | Common in `data` |
| External secret manager | Not normally required | Often recommended for production |

---

# 13. External Secret Management

For production environments, consider:

- HashiCorp Vault
- Azure Key Vault
- AWS Secrets Manager
- Google Secret Manager
- External Secrets Operator

Conceptually:

```text
Application
    ↓
Kubernetes
    ↓
External Secret Integration
    ↓
Vault / Cloud Secret Manager
```

This can reduce the need to store long-lived sensitive values directly in Git-managed Kubernetes manifests.

---

# 14. Immutable ConfigMaps and Secrets

Kubernetes supports immutable ConfigMaps and Secrets.

Example:

```yaml
immutable: true
```

Immutable objects can be useful when configuration should not change after creation.

---

# 15. Environment-Specific Configuration

A common model:

```text
Base configuration
        │
   ┌────┼─────┐
   ↓    ↓     ↓
 Dev  Stage  Prod
```

Helm, Kustomize, or GitOps tools can help manage environment-specific configuration.

---

# 16. Security Best Practices

### Do

- Use least-privilege RBAC
- Protect access to Secrets
- Enable appropriate encryption at rest
- Use external secret managers where appropriate
- Avoid committing secrets to Git
- Rotate credentials
- Audit secret access

### Do not

- Put passwords in Dockerfiles
- Commit plaintext credentials
- Treat Base64 as encryption
- Give every ServiceAccount access to every Secret

---

# 17. Hands-On Lab

Create:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_NAME: kubernetes-demo
  LOG_LEVEL: info
---
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  API_KEY: demo-key
```

Apply:

```bash
kubectl apply -f config.yaml
```

Inspect:

```bash
kubectl get configmap app-config
kubectl get secret app-secret
```

---

# 18. Use Them in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-demo
spec:
  containers:
    - name: app
      image: busybox:1.36
      command:
        - sh
        - -c
        - |
          echo "APP_NAME=$APP_NAME"
          echo "LOG_LEVEL=$LOG_LEVEL"
          echo "API_KEY=$API_KEY"
          sleep 3600

      env:
        - name: APP_NAME
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_NAME

        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: LOG_LEVEL

        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: API_KEY
```

Inspect:

```bash
kubectl logs config-demo
```

---

# 19. Troubleshooting

If environment variables are missing:

```bash
kubectl describe pod <pod>
kubectl get configmap
kubectl get secret
```

Check the key names carefully.

For a missing ConfigMap/Secret, inspect Pod events:

```bash
kubectl describe pod <pod>
```

---

# 20. Interview Questions

1. What is a ConfigMap?
2. What is a Secret?
3. ConfigMap vs Secret?
4. Is Base64 encryption?
5. How can a ConfigMap be consumed?
6. How can a Secret be consumed?
7. What happens if a referenced ConfigMap does not exist?
8. How do you prevent secrets from entering Git?
9. What is an external secret manager?
10. How would you manage secrets in AKS/EKS/GKE?

---

# 21. Must Understand

You should be able to explain:

```text
Application Image
       +
ConfigMap
       +
Secret
       ↓
Application Container
```

And:

```text
Git
 ├── Application manifests
 └── No plaintext production secrets

External Secret Manager
            ↓
       Kubernetes
            ↓
       Application
```
