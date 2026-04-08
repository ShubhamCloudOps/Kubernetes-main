# A complete, clean Argo Rollouts canary setup you can actually apply and test. This includes:

### 1. Rollout (canary strategy)
### 2. Services (stable + canary)
   👉 Argo automatically manages which pods are “stable” vs “canary”
### 3. Ingress (NGINX traffic split)
   👉 Argo updates traffic split automatically behind the scenes
### 4. AnalysisTemplate (auto rollback)
    
## Using Argo Rollouts


# 🧪 How to Deploy
kubectl apply -f rollout.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
kubectl apply -f analysis.yaml

# 🔍 How It Works (Simple)
### 1. New version deployed
### 2. 10% traffic → wait
### 3. Metrics checked (Prometheus)
### 4. If OK → 50% → then 100%
### 5. If errors ❌ → automatic rollback


Users → Ingress → Argo Rollouts
                     │
        ┌────────────┴────────────┐
        │                         │
   Stable Pods              Canary Pods


 #  ⚠️ Requirements
Install Argo Rollouts CRD + controller
NGINX Ingress OR service mesh (for traffic split)
Prometheus (for analysis)


## 👉 “Argo Rollouts enables automated canary deployments by defining traffic weights and analysis checks in YAML, allowing progressive delivery with automatic rollback based on metrics.”
