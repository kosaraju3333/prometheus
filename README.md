# prometheus
Monitoring repo



Prometheus Set Up on EKS Cluster:

Installation :'
Using Helm:
Doc: https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack
```
helm install [RELEASE_NAME] oci://ghcr.io/prometheus-community/charts/kube-prometheus-stack
```

- Expose Prometheus Service to external world using AWS Load balancer Controller
- Expose Grafana service to external world by using AWS Load balancer Controller 
Prometheus, Grafana Deployments and exposed to external world:

<img width="1770" height="975" alt="Screenshot 2025-11-19 at 6 57 41 PM" src="https://github.com/user-attachments/assets/242a114b-d5d0-45ff-9d3a-5eb1565076d8" />


<img width="1918" height="202" alt="Screenshot 2025-11-19 at 6 58 45 PM" src="https://github.com/user-attachments/assets/aae9cb27-4fe6-407d-a931-70be3b3687c1" />


Prometheus UI:

<img width="1916" height="855" alt="Screenshot 2025-11-19 at 6 55 48 PM" src="https://github.com/user-attachments/assets/09898b77-a8a4-4c84-896e-2bc1875a0308" />


Ran basic query to show that data is fetched from EKS cluster: 

<img width="1912" height="534" alt="Screenshot 2025-11-19 at 7 06 53 PM" src="https://github.com/user-attachments/assets/8b83d505-d422-4fe7-b5c3-3acde664d68b" />


Grafana Dashboard:

<img width="1916" height="1038" alt="Screenshot 2025-11-19 at 6 56 49 PM" src="https://github.com/user-attachments/assets/94defe78-eb5a-4866-98d1-621db725a654" />


Grafana Kubernates Dashboard:

<img width="1918" height="1035" alt="Screenshot 2025-11-19 at 7 00 23 PM" src="https://github.com/user-attachments/assets/354f3b39-e4b6-4427-86a8-1f3ec18c07ce" />


To add New Target to Prometheus (App inside the EKS):
Need to follow steps:
- Create ServiceMonitor on top  of Service 
```
This could be your POD/Deployment and etc..

apiVersion: v1
kind: Pod
metadata:
  name: sample-app
  namespace: default
  labels:
    app: sample-app
spec:
  containers:
    - name: sample-app
      image: nginx   # you can replace this with your app image
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /metrics
          port: 8080
      livenessProbe:
        httpGet:
          path: /metrics
          port: 8080
---

Service to expose internally

apiVersion: v1
kind: Service
metadata:
  name: sample-app
  namespace: default
  labels:
    app: sample-app
spec:
  selector:
    app: sample-app
  ports:
    - name: metrics
      port: 8080
      targetPort: 8080
---
ServiceMonitor On top of Service to add app as target in prometheus 

apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: sample-app-servicemonitor
  namespace: default
  labels:
    release: prom     # IMPORTANT — must match your Prometheus Helm release
spec:
  selector:
    matchLabels:
      app: sample-app
  endpoints:
    - port: metrics
      path: /metrics
      interval: 30s
```

To add or scrape New Target to Prometheus (App Outside  EKS or add monitor any EC2 vm with node exporter):
Follow below steps:
- Create Service with no selector
- Create Endpoint object pointing to external VM IP
- Create ServiceMonitor 
```
Step 1: Create Service with no selector
---
apiVersion: v1
kind: Service
metadata:
  name: external-node
  namespace: monitoring
  labels:
    app: external-node
spec:
  ports:
    - name: metrics
      port: 9100
      targetPort: 9100
---

Step 2: Create Endpoint object pointing to external VM IP

apiVersion: v1
kind: Endpoints
metadata:
  name: external-node
  namespace: monitoring
subsets:
  - addresses:
      - ip: <EXTERNAL_NODE_IP>
    ports:
      - port: 9100
        name: metrics
---

Step 3: Create ServiceMonitor

apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: external-node-monitor
  namespace: monitoring
  labels:
    release: prom
spec:
  selector:
    matchLabels:
      app: external-node
  endpoints:
    - port: metrics
      interval: 30s
---
```

- Added or scraped new target to prometheus 
- Its only test purpose, I did not exposed any metric to /metric that why its fails .
  
<img width="1918" height="1039" alt="Screenshot 2025-11-19 at 7 21 18 PM" src="https://github.com/user-attachments/assets/0057016c-56b7-4d75-933f-2922661a77fd" />

