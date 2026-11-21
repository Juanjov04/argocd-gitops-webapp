# Despliegue GitOps con Argo CD

Estructura:
- `manifests/deployment.yml` – 2 réplicas, requests/limits, puerto 80
- `manifests/service.yml` – NodePort 30080 -> 80
- `manifests/hpa.yml` – min 2, max 5, CPU 50%
- `application.yml` – Application de Argo CD (Automated Sync, path `manifests`)

## Pasos (Docker Desktop con Kubernetes activado)
```powershell
# 1. Metrics Server (necesario para el HPA)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch deployment metrics-server -n kube-system --type=json -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'

# 2. Argo CD
kubectl create namespace argocd
kubectl apply -n argocd --server-side -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl port-forward svc/argocd-server -n argocd 8080:443
# clave admin:
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | % { [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($_)) }

# 3. Crear la aplicación (editar repoURL antes)
kubectl apply -f application.yml

# 4. Verificar
kubectl get deploy,svc,hpa,pods
# app en http://localhost:30080  (o: kubectl port-forward svc/webapp-service 8081:80)

# 5. Probar HPA con carga
kubectl run carga --rm -it --image=busybox -- /bin/sh -c "while true; do wget -q -O- http://webapp-service; done"
kubectl get hpa -w
```
