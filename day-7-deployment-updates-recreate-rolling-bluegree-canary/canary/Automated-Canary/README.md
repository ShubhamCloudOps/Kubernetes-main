# A complete, clean Argo Rollouts canary setup you can actually apply and test. This includes:

### 1. Rollout (canary strategy)
    apiVersion: argoproj.io/v1alpha1
    kind: Rollout
    metadata:
      name: myapp-rollout
    spec:
      replicas: 5

      selector:
        matchLabels:
          app: myapp

      template:
        metadata:
          labels:
            app: myapp
        spec:
           containers:
            - name: myapp
              image: nginx:1.25   # change version to trigger rollout
              ports:
                - containerPort: 80

      strategy:
        canary:
          canaryService: myapp-canary
          stableService: myapp-stable

          analysis:
                templates:
                  - templateName: success-rate
            
          steps:
                - setWeight: 10
                - pause: {duration: 30s}

            - setWeight: 30
            - pause: {duration: 60s}

            - setWeight: 60
            - pause: {duration: 90s}
    
            - setWeight: 100

      
### 2. Services (stable + canary)
    apiVersion: v1
    kind: Service
    metadata:
      name: myapp-stable
    spec:
      selector:
    app: myapp
      ports:
        - port: 80
          targetPort: 80
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: myapp-canary
    spec:
      selector:
        app: myapp
      ports:
        - port: 80
          targetPort: 80
   👉 Argo automatically manages which pods are “stable” vs “canary”
### 3. Ingress (NGINX traffic split)
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: myapp-ingress
      annotations:
        kubernetes.io/ingress.class: nginx
    spec:
      rules:
        - host: myapp.local
          http:
            paths:
              - path: /
                pathType: Prefix
                backend:
                  service:
                    name: myapp-stable
                    port:
                      number: 80
   👉 Argo updates traffic split automatically behind the scenes
### 4. AnalysisTemplate (auto rollback)
    apiVersion: argoproj.io/v1alpha1
    kind: AnalysisTemplate
    metadata:
      name: success-rate
    spec:
      metrics:
        - name: success-rate
          interval: 30s
          successCondition: result > 95
          failureCondition: result < 90
          provider:
            prometheus:
              address: http://prometheus:9090
              query: |
                sum(rate(http_requests_total{status=~"2.."}[1m]))
                /
                sum(rate(http_requests_total[1m])) * 100
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
