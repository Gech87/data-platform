# Kubernetes Deployment

## Architecture
- Namespace-per-environment pattern
- Kustomize for environment customization
- Persistent storage for stateful services
- Health checks and resource limits

## Deployment
```bash
# Deploy to dev
kubectl apply -k kubernetes/overlays/dev

# Deploy to prod  
kubectl apply -k kubernetes/overlays/prod