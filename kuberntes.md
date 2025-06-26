Kubernetes Interview Q&A (Clearly Formatted)
========================================

Q1. What is a Namespace?
------------------------
A namespace in Kubernetes is a way to divide cluster resources between multiple users. It helps isolate environments (like dev, test, prod) within the same cluster.

Example:

kubectl create namespace dev

Q2. .kubeconfig usage
---------------------
The .kube/config file contains configuration to connect to Kubernetes clusters (credentials, cluster info, contexts).

Location: ~/.kube/config

Example:

contexts:
- name: dev-context
  context:
    cluster: dev-cluster
    user: dev-user

Q3. Switch the cluster context from CLI
---------------------------------------
kubectl config use-context dev-context

Q4. Check the Pod Logs
----------------------
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>

Q5. Login to Running Container
------------------------------
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh

Q6. What is PV and PVC?
-----------------------
PV (Persistent Volume): Storage in the cluster.
PVC (Persistent Volume Claim): Request for storage by user.

Example:

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

Q7. Different Types of Services
-------------------------------
ClusterIP: Default, accessible only within cluster.
NodePort: Exposes service on a static port on each node.
LoadBalancer: Uses external load balancer.
ExternalName: Maps service to external DNS.

Example:

apiVersion: v1
kind: Service
metadata:
  name: myservice
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
  selector:
    app: myapp

Q8. Configure Application Load Balancer (ALB)
---------------------------------------------
For EKS: Use AWS ALB Ingress Controller.

Example:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  rules:
  - http:
      paths:
      - path: /*
        pathType: ImplementationSpecific
        backend:
          service:
            name: myservice
            port:
              number: 80

Q9. Expose Ports
----------------
kubectl expose deployment myapp --type=NodePort --port=80 --target-port=8080

Q10. Port Forwarding
-------------------
kubectl port-forward svc/myservice 8080:80
# Now access the service via http://localhost:8080

Q11. EKS Versions Worked On
--------------------------
Worked on EKS versions like 1.21, 1.22, and recently upgraded to 1.24.

Q12. EKS Upgrade Experience
--------------------------
Performed in-place upgrades using AWS Console and CLI. Steps included:
- Backing up cluster configs
- Draining nodes
- Upgrading EKS version
- Updating node groups and validating workloads post-upgrade

Q13. What are ConfigMaps?
------------------------
ConfigMaps store non-sensitive key-value pairs (e.g., config settings).

Example:

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: debug

Q14. Secrets vs ConfigMaps
-------------------------
Secrets store sensitive data (e.g., passwords, tokens).

Example:

apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  password: cGFzc3dvcmQ=

Difference:
- ConfigMap: Plaintext data
- Secret: Encoded/encrypted data for sensitive info

Q15. Role-Based Access Control (RBAC)
------------------------------------
RBAC defines permissions for users/groups to perform actions.

Example:

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: dev-reader
rules:
- apiGroups: []
  resources: ["pods"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: dev
subjects:
- kind: User
  name: alice
roleRef:
  kind: Role
  name: dev-reader
  apiGroup: rbac.authorization.k8s.io

==================================================================================================================


## 📌 4. Monitoring

---

### 🔹 a. Any experience with ELK, Grafana?

### ✅ ELK Stack (Elasticsearch, Logstash, Kibana)

Yes — I have worked with the ELK stack for centralized log management.

- **Elasticsearch**: Stores indexed logs
- **Logstash**: Ingests and parses logs
- **Kibana**: Visualizes and queries logs

### ✅ Use Case:
> We used ELK to collect logs from EC2, Lambda, and application containers.
> - Logstash was configured to pull logs from S3 and CloudWatch using input plugins.
> - Elasticsearch hosted on an EC2 instance stored parsed logs.
> - Kibana dashboards helped devs debug 500 errors by filtering logs by timestamp, service, and environment (dev/staging/prod).

---

### 🔹 b. ELK Setup You Worked On?

Yes — here is a simplified **ELK architecture** I implemented:

### ✅ Setup Steps:
1. **Provision EC2 Instances** for:
   - Elasticsearch
   - Logstash
   - Kibana (or use Elastic Cloud for managed service)

2. **Install and configure Logstash**:
```bash
input {
  file {
    path => "/var/log/app/*.log"
    start_position => "beginning"
  }
}
filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}" }
  }
}
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "app-logs-%{+YYYY.MM.dd}"
  }
}
```

3. **Install Kibana**, configure connection to Elasticsearch.
4. Create **dashboards and visualizations** based on log-level, service name, error patterns.

### ✅ Use Case:
> This helped the support team monitor all microservice logs in one place and filter errors by component or environment. It also replaced manual log grepping across instances.

---

### 🔹 c. Any experience with New Relic? Please explain the setup steps.

Yes — I have used New Relic for full-stack observability including **APM (Application Performance Monitoring)**, **infrastructure monitoring**, and **synthetic checks**.

### ✅ New Relic Setup Steps:

#### 🔧 Application (APM) Setup:
1. **Install New Relic agent** in your app:
   - For Python: `pip install newrelic`
   - For Node.js: `npm install newrelic`
2. Configure `newrelic.ini` or `newrelic.js` with your **license key**.
3. Wrap the application with the agent:
   ```bash
   NEW_RELIC_CONFIG_FILE=newrelic.ini python app.py
   ```

#### 🔧 Infrastructure Monitoring:
1. Install New Relic Infrastructure Agent on EC2:
   ```bash
   curl -o - https://download.newrelic.com/infrastructure_agent/gpg/newrelic-infra.gpg | sudo apt-key add -
   echo "deb [arch=amd64] https://download.newrelic.com/infrastructure_agent/linux/apt focal main" | sudo tee /etc/apt/sources.list.d/newrelic-infra.list
   sudo apt-get update
   sudo apt-get install newrelic-infra -y
   ```

2. Add your license key in `/etc/newrelic-infra.yml`.

#### 🔧 Create Dashboards:
- Use New Relic One to build custom dashboards with:
  - **APM traces**
  - **Error rate**
  - **DB query performance**
  - **External services latency**

### ✅ Use Case:
> We used New Relic to identify slow API endpoints in a Django app.
> It helped reduce response times by identifying N+1 queries and slow DB calls.
> New Relic alerts were integrated with Slack to notify the team of high memory usage or 500 error spikes in real time.


=============================================================================================================================




