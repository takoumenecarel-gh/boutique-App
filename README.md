# boutique-App
# 🚀 Google Online Boutique on Kubernetes (kind)

## 📌 Overview
This project demonstrates the deployment of **Google Online Boutique**, a production-grade cloud-native e-commerce application composed of **11 microservices + 1 load generator (12 pods total)**.

The application uses **gRPC-based communication** across services written in multiple languages (Go, Python, Java, Node.js, C#).

It is deployed on a **local Kubernetes cluster (kind)** with necessary adjustments to handle networking and resource constraints.

---

## 🎯 Objectives

- Deploy a real-world microservices architecture on Kubernetes
- Understand inter-service communication via gRPC
- Troubleshoot **CrashLoopBackOff** and **OOMKilled**
- Work with **liveness/readiness probes**
- Debug distroless containers using a **debug pod**
- Validate internal DNS and service connectivity

---

## 🏗 Architecture

The application consists of the following services:

| Service | Language | Port | Description |
|--------|----------|------|-------------|
| frontend | Go | 80 | Web UI |
| cartservice | C# | 7070 | Shopping cart |
| productcatalogservice | Go | 3550 | Product listing |
| currencyservice | Node.js | 7000 | Currency conversion |
| paymentservice | Node.js | 50051 | Payment processing |
| shippingservice | Go | 50051 | Shipping cost |
| emailservice | Python | 5000 | Email simulation |
| checkoutservice | Go | 5050 | Order orchestration |
| recommendationservice | Python | 8080 | Recommendations |
| adservice | Java | 9555 | Ads |
| redis-cart | Redis | 6379 | Cart storage |
| loadgenerator | - | - | Traffic simulation |

---

## ⚙️ Prerequisites

- Kubernetes cluster (**kind recommended**)
- kubectl installed and configured
- Docker installed

---

## 🚀 Deployment Steps

### 1. Create Namespace

```bash
kubectl create namespace boutique

## 2.  Deploy the Application

kubectl apply -n boutique -f https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/main/release/kubernetes-manifests.yaml

## 3. Verify pods
kubectl get pods -n boutique -w

# ## 4. Troubleshooting  
# - Issue 1: CrashLoopBackOff (Probe Timeout)
# - Cause: gRPC probe timeout too low (1s) for kind + Calico networking
# - Fix :
    kubectl patch deployment cartservice -n boutique --type='json' -p='[
  {"op":"replace","path":"/spec/template/spec/containers/0/livenessProbe/timeoutSeconds","value":10},
  {"op":"replace","path":"/spec/template/spec/containers/0/readinessProbe/timeoutSeconds","value":10},
  {"op":"replace","path":"/spec/template/spec/containers/0/livenessProbe/initialDelaySeconds","value":30}
]'

# - Issue2: OOMKilled
# - Cause: Memory limit too low (128Mi)
# - Fix:
    kubectl patch deployment cartservice -n boutique --type='json' -p='[
  {"op":"replace","path":"/spec/template/spec/containers/0/resources/limits/memory","value":"256Mi"},
  {"op":"replace","path":"/spec/template/spec/containers/0/resources/requests/memory","value":"128Mi"}
]'

# - Access the Application :
    kubectl port-forward svc/frontend-external 8080:80 -n boutique
# - Open :
    http://localhost:8080

# - Debugging (Distroless Containers)
# Since some containers (like frontend) are distroless, use a debug pod:
    kubectl run debug-shell \
  --image=busybox:1.35 \
  --restart=Never \
  -n boutique \
  -it --rm \
  -- sh

 # - DNS verification: 
  nslookup frontend
  nslookup cartservice

 # - Connectivity Test (Inside Debug Pod)

    for svc in "frontend:80" "productcatalogservice:3550" "currencyservice:7000" \
  "cartservice:7070" "checkoutservice:5050" "shippingservice:50051" \
  "emailservice:5000" "paymentservice:50051" "adservice:9555" "redis-cart:6379"; do
  host=$(echo $svc | cut -d: -f1)
  port=$(echo $svc | cut -d: -f2)
  nc -zv -w 3 $host $port 2>&1 && echo "$svc OPEN" || echo "$svc FAILED"
done

# All services should return OPEN

# - Resource Optimization Example:

    kubectl patch deployment frontend -n boutique --type='json' -p='[
 {"op":"replace","path":"/spec/template/spec/containers/0/resources/requests/cpu","value":"100m"},
 {"op":"replace","path":"/spec/template/spec/containers/0/resources/requests/memory","value":"128Mi"},
 {"op":"replace","path":"/spec/template/spec/containers/0/resources/limits/cpu","value":"400m"},
 {"op":"replace","path":"/spec/template/spec/containers/0/resources/limits/memory","value":"256Mi"}
]'

## - Key Learnings :
'
- Debugging CrashLoopBackOff caused by probe misconfiguration
- Handling OOMKilled due to resource limits
- Working with gRPC health checks
- Using debug pods for distroless containers
- Understanding Kubernetes DNS and service discovery
- Validating microservice communication inside the cluster
'

## Conclusion
'

This project simulates a real production Kubernetes environment with:

- Multi-language microservices
- gRPC communication
- Real traffic via load generator
- Real-world debugging scenarios

''' 

👨‍💻 Author
'
Carel Assonkeng Takoumene - DevSecOps Engineer