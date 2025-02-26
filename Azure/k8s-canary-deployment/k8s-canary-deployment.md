# 🚀 Canary Deployment in Kubernetes with Load Balancer

## 👨‍💻 Author  

**Vo The Minh**  

- 🌐 **GitHub**: [https://github.com/18120211](https://github.com/18120211)  
- 💼 **LinkedIn**: [https://www.linkedin.com/in/minh-vo-108643118/](https://www.linkedin.com/in/minh-vo-108643118/)  
- 📧 **Email**: minhthevo123@gmail.com 

🚀 Passionate about **Kubernetes, DevOps, and Cloud Engineering**. Always learning and sharing knowledge!  
Feel free to connect! 😊  


## 📖 Table of Contents

- [🚀 Canary Deployment in Kubernetes with Load Balancer](#canary-deployment-in-kubernetes-with-load-balancer)
  - [Overview ](#overview)
  - [📌 Key Concepts ](#key-concepts)
  - [🏗 Architecture and Workflow ](#architecture)
  - [🔧 Technologies Used ](#technologies-used)
  - [🛠 Implementation ](#implementation)
    - [Step 1: Create a Kubernetes Namespace](#step-1-create-a-kubernetes-namespace)
    - [Step 2: Deploy the Stable Version (v1)](#step-2-deploy-the-stable-version-v1)
    - [Step 3: Deploy the Canary Version (v2)](#step-3-deploy-the-canary-version-v2)
    - [Step 4: Load Balancer Service for Traffic Routing](#step-4-load-balancer-service-for-traffic-routing)
    - [Step 5: Implement Traffic Splitting](#step-5-implement-traffic-splitting)
    - [Step 6: Monitoring and Rollout Strategy](#step-6-monitoring-and-rollout-strategy)
  - [📊 Observability (Prometheus \& Grafana) ](#observability-prometheus--grafana)
  - [✅ Benefits of Canary Deployment ](#benefits-of-canary-deployment)
  - [🚀 Future Enhancements ](#future-enhancements)
  - [🎯 Conclusion ](#conclusion)
  - [📌 What’s Included?](#whats-included)
  - [📚 References](#references)


## Overview <a id="overview"></a>
This project demonstrates **Canary Deployment** in **Kubernetes (K8s)** using a **Load Balancer** to gradually roll out new application versions with minimal risk. Canary deployments allow testing new releases with a subset of users before full rollout, ensuring stability and performance.

![k8s-canary-deployments.png](./k8s-canary-deployments.png)


$~$

## 📌 Key Concepts <a id="key-concepts"></a> 
- **Canary Deployment** – A progressive rollout strategy where a new version is released to a small subset of traffic before full deployment.
- **Kubernetes Services & Load Balancer** – Distributes traffic to different versions of the application.
- **Traffic Splitting** – Uses labels, weights, or Istio/Nginx Ingress to route traffic between stable and canary versions.

$~$

## 🏗 Architecture and Workflow <a id="architecture"></a> 
![canary-deployment-workflow.png](./canary-deployment-workflow.png)

1. **Load Balancer Service**
   - Exposes the application externally.
   - Routes traffic to different versions based on deployment strategy.

2. **Canary Deployment**
   - A **small percentage of traffic** is routed to the new version.
   - If the new version is stable, traffic is **gradually increased**.
   - If issues occur, rollback to the stable version.

3. **Progressive Traffic Shifting**
   - Managed via **Kubernetes services, Istio, or Nginx Ingress**.
   - Ensures **controlled exposure** to users.

$~$

## 🔧 Technologies Used <a id="technologies-used"></a>
- **Kubernetes (K8s)** – Container orchestration.
- **Load Balancer Service** – External traffic management.
- **Istio / Nginx Ingress** – For intelligent traffic routing.
- **Helm** – Package management for deployments.
- **Prometheus & Grafana** – Monitoring traffic patterns & performance.

$~$

## 🛠 Implementation <a id="implementation"></a>
![k8s-canary-pipeline.png](./k8s-canary-pipeline.png)
### Step 1: Create a Kubernetes Namespace
```bash
kubectl create namespace canary-demo
```

---

### Step 2: Deploy the Stable Version (v1)
Create the deployment file `stable-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v1
  namespace: canary-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: v1
  template:
    metadata:
      labels:
        app: my-app
        version: v1
    spec:
      containers:
        - name: my-app
          image: my-app:v1
          ports:
            - containerPort: 80
```

Apply the deployment:
```bash
kubectl apply -f stable-deployment.yaml
```

---

### Step 3: Deploy the Canary Version (v2)
Create the deployment file `canary-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-canary
  namespace: canary-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: v2
  template:
    metadata:
      labels:
        app: my-app
        version: v2
    spec:
      containers:
        - name: my-app
          image: my-app:v2
          ports:
            - containerPort: 80
```
Apply the deployment:

```bash
kubectl apply -f canary-deployment.yaml
```

---

### Step 4: Load Balancer Service for Traffic Routing
Create the service file `service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
  namespace: canary-demo
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

Apply the service:
```bash
kubectl apply -f service.yaml
```

---

### Step 5: Implement Traffic Splitting
**Option 1: Using Kubernetes Services**
Modify the service selector in `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
  namespace: canary-demo
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
  sessionAffinity: None
```

By default, traffic is randomly distributed based on replica count.
To control weights, use Istio or Nginx Ingress.

**Option 2: Using Istio Virtual Service for Weighted Traffic Routing**
Create the Istio configuration file `istio-routing.yaml`:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-app-routing
  namespace: canary-demo
spec:
  hosts:
    - my-app-service
  http:
    - route:
        - destination:
            host: my-app-v1
          weight: 90
        - destination:
            host: my-app-canary
          weight: 10
```

Apply the Istio rule:

```bash
kubectl apply -f istio-routing.yaml
```
- 90% traffic → Stable version (v1)
- 10% traffic → Canary version (v2)
Increase v2's weight gradually based on monitoring.

---

### Step 6: Monitoring and Rollout Strategy
Check Pod Status
```bash
kubectl get pods -n canary-demo
kubectl logs -f my-app-canary-xyz -n canary-demo
```

Get Load Balancer External IP
```bash
kubectl get svc -n canary-demo
```

Rollback to v1 (if issues detected)
```bash
kubectl delete deployment my-app-canary -n canary-demo
```

$~$

## 📊 Observability (Prometheus & Grafana) <a id="observability-prometheus-grafana"></a>
Set up Prometheus and Grafana to visualize:
- **Traffic Distribution**
- **Response Times**
- **Error Rates**

Helm Install:
```bash
helm install prometheus prometheus-community/kube-prometheus-stack
helm install grafana grafana/grafana
```

$~$

## ✅ Benefits of Canary Deployment <a id="benefits-of-canary-deployment"></a> 
- **Reduces Risk** – Gradual rollout ensures fewer failures.
- **Quick Rollbacks** – Instantly revert to the previous version if issues arise.
- **Real User Testing** – Test new features with a subset of users.
- **Optimized Performance** – Monitor real-time impact before full release.

$~$

## 🚀 Future Enhancements <a id="future-enhancements"></a>
- **Automate Traffic Shifting** – Use Kubernetes `HPA` (Horizontal Pod Autoscaler) or Argo Rollouts.
- **Advanced Traffic Routing** – Integrate with Linkerd or **Azure Application Gateway**.
- **A/B Testing Integration** – Dynamically serve versions based on user segments.

## 🎯 Conclusion <a id="conclusion"></a>
This project showcases a **robust, scalable Canary Deployment strategy in Kubernetes**, ensuring **smooth rollouts with real-time monitoring and rollback capabilities**. 🚀

📌 **Canary Deployment + Load Balancer = Safer & Smarter Deployments** 🔥

$~$

## 📌 **What’s Included?** <a id="whats-included"></a>
✔ **Step-by-step implementation**  
✔ **Kubernetes manifests for Canary Deployment**  
✔ **Traffic splitting via LoadBalancer, Istio, or Nginx**  
✔ **Monitoring setup with Prometheus & Grafana**  
✔ **Rollback strategy for quick recovery**  

This **README.md** is **production-ready** and can be used in a **GitHub repository** or **portfolio showcase**! 🎯  

Let me know if you need **customizations** or **extra details**! 🚀

$~$

## 📚 References <a id="references"></a>  

Below are useful references for implementing **Canary Deployment in Kubernetes** and **Multi-Tenancy on AKS**:  

### 🔹 Kubernetes Documentation  
- [Kubernetes Canary Deployments](https://kubernetes.io/docs/concepts/services-networking/ingress/#canary) – Canary release strategies in Kubernetes.  

### 🔹 Azure Kubernetes Service (AKS)  
- [RBAC in AKS](https://learn.microsoft.com/en-us/azure/aks/azure-ad-rbac) – Role-based access control (RBAC) in AKS.  
- [Networking in AKS](https://learn.microsoft.com/en-us/azure/aks/concepts-network) – How networking and load balancing work in AKS.  

### 🔹 CI/CD & Infrastructure as Code  
- [Terraform for AKS](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/kubernetes_cluster) – Managing AKS using Terraform.  
- [GitOps with ArgoCD](https://argo-cd.readthedocs.io/en/stable/) – Automating Kubernetes deployments using GitOps.  

### 🔹 Monitoring & Logging  
- [Azure Monitor for Containers](https://learn.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-overview) – Monitoring AKS with Azure Monitor.  
- [Prometheus & Grafana](https://prometheus.io/docs/introduction/overview/) – Monitoring Kubernetes clusters with Prometheus.  
- [Fluentd Logging for Kubernetes](https://docs.fluentd.org/container-deployment/kubernetes) – Aggregating logs in Kubernetes using Fluentd.  

These resources provide in-depth knowledge and best practices for **implementing multi-tenancy and Canary deployments** in Kubernetes environments. 🚀  
