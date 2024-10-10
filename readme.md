# Multi-tier application using Kubernetes & HELM

## 1. Install and Deploy Helm Chart

We are using a generic custom Helm template for deploying frontend and backend services. The only part we manage is the values in `.yaml` files for the frontend and backend.

To deploy the services using Helm:

```bash
helm install <release-name> -f <values-file> <chart-name>
```
For the Frontend Service:
```bash
helm install frontend -f values/frontend.yaml generic-helm-template/
```
For the Backend Service:
```bash
helm install backend -f values/backend.yaml generic-helm-template/
```



## 2. Setting up TLS for Ingress (Self-Signed Certificate)
To secure your Ingress, you can use a self-signed certificate by following these steps.

### Generate a Private Key:
```bash
openssl genrsa -out tls.key 2048
```
### Generate a Self-Signed Certificate:
```bash
openssl req -new -x509 -key tls.key -out tls.crt -days 365 -subj "/CN=demo.myapp.com/O=demo.myapp.com"
```
### Create Kubernetes Secret for TLS:
```bash
kubectl create secret tls myapp-tls --cert=tls.crt --key=tls.key
```
### Update Ingress Resource for TLS:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ .Release.Name }}-ingress
  namespace: {{ .Release.Namespace }}
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
spec:
  tls:
    - hosts:
        - demo.myapp.com   # Common Name (Domain Name)
      secretName: myapp-tls    # Name of the secret created in the previous step
  rules:
    - host: demo.myapp.com   # Common Name (Domain Name)
      http:
        paths:
          # Define the paths for routing
```

## 3. Security Contexts, HPA, and RBAC Configurations

### Security Contexts
The security context is defined for containers in the deployment.yaml file.
### The container runs as a non-root user:
- runAsUser: 1000
- runAsGroup: 3000
- fsGroup: 2000
- runAsNonRoot: true

Horizontal Pod Autoscaler (HPA)
### The HPA is configured for the backend service in hpa.yaml:
- Minimum replicas: 1
- Maximum replicas: 10
- CPU utilization target: 60%
- Memory utilization target: 60%

### Role-Based Access Control (RBAC)
RBAC configurations are set in the service account, role, and role binding files.
The role has permissions to:
- get, list, watch, create, and update pods and services in the namespace.
The role binding binds the role to the service account.

## 4. Thought Process, Assumptions, and Considerations
A generic custom Helm template is used for all services, with specific values managed through .yaml files for each service.
Separate values.yaml files are created for each service (e.g., frontend and backend).
### Configuration files for each service include:
- deployment.yaml (Seployment configuration)
- service.yaml (Service configuration)
- ingress.yaml (Ingress configuration)
- hpa.yaml (HPA configuration)
- role.yaml (Role configuration for RBAC)
- rolebinding.yaml (Role Binding configuration for RBAC)
- serviceaccount.yaml (Service Account configuration for RBAC)
- secrets.yaml (Secret Variables, e.g., DB USERNAME and PASSWORD)
- configmap.yaml (Config Variables, e.g., DB HOST and PORT)

### Assumptions
- Kubernetes cluster is up and running.
- Helm is installed.
- Necessary permissions exist to create resources in the namespace.
- Permissions to create Service Accounts, Roles, Role Bindings, Secrets, and ConfigMaps are in place.

### Considerations
- **Security:** Set appropriate security contexts for containers.
- **Scalability:** HPA is configured for backend services.
- **RBAC:** Ensure Service Accounts, Roles, and Role Bindings are configured with the necessary permissions.
- **Resource Management:** Configure Replicas, Secrets, ConfigMaps, Ingress, and Services as required.
