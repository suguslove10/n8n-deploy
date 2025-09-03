# Deploy n8n with Argo CD + Helm

Simple, readable Argo CD manifest that deploys n8n using the public Helm chart.

## Files
- `argocd-app-n8n.yaml` — Argo CD Application with inline Helm values
- `values.yaml` — Optional values file you can copy into the manifest if desired

## Prereqs
- Kubernetes cluster with storage class
- Argo CD installed (`argocd` namespace)
- Optional: Ingress controller (if enabling ingress)

## Add Helm repo to Argo CD
UI: Settings → Repositories → Connect → Helm
- Name: `n8n-helm-repo`
- URL: `https://8gears.github.io/n8n-helm-chart/`

CLI:
```bash
argocd repo add https://8gears.github.io/n8n-helm-chart/
```

## Deploy
```bash
kubectl apply -n argocd -f argocd-app-n8n.yaml
```
Argo CD will create namespace `n8n` automatically and install the chart.

## Ingress (optional)
Edit `argocd-app-n8n.yaml` values block and set:
```yaml
ingress:
  enabled: true
  className: nginx
  hosts:
    - host: n8n.example.com
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: n8n-tls
      hosts:
        - n8n.example.com
```

## External Postgres (optional)
Disable bundled Postgres and set env:
```yaml
postgresql:
  enabled: false

env:
  - name: DB_TYPE
    value: postgresdb
  - name: DB_POSTGRESDB_HOST
    value: <host>
  - name: DB_POSTGRESDB_PORT
    value: "5432"
  - name: DB_POSTGRESDB_USER
    valueFrom:
      secretKeyRef:
        name: n8n-db
        key: username
  - name: DB_POSTGRESDB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: n8n-db
        key: password
  - name: DB_POSTGRESDB_DATABASE
    value: n8n
```

## Quick access
```bash
kubectl -n n8n port-forward svc/n8n 5678:5678
```

Adjust `persistence.size` and `resources` as needed.
