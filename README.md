# Kubernetes NGINX Webapp

A production-style Kubernetes deployment of an NGINX web application using open-source container images. Built and debugged on a real cluster not just copy-pasted manifests.

## Project Overview:

This project is a Kubernetes-based NGINX web application setup that includes namespace isolation with resource quotas and limits, network policies enforcing zero-trust security, ConfigMap-based externalized NGINX configuration and HTML, a deployment with rolling updates, health probes, non-root security and init containers, a ClusterIP service for internal routing, ingress for host-based routing with rate limiting, and an HPA configuration for autoscaling based on CPU and memory with tuning behavior.

## Concepts Covered

Namespace isolation `ns.yaml`
ResourceQuota + LimitRange `ns.yaml`
Zero-trust NetworkPolicy `network-policy.yaml`
Externalized config via ConfigMap `configmap.yaml` 
Rolling update strategy `deployment.yaml` 
Init container for permissions `deployment.yaml`
Non-root container + dropped capabilities `deployment.yaml`
Liveness + Readiness probes `deployment.yaml`
ClusterIP Service `service.yaml` 
Ingress with rate limiting  `ingress.yaml`
HPA CPU + Memory autoscaling `hpa.yaml`


## Prerequisites

- Kubernetes cluster — local ([minikube](https://minikube.sigs.k8s.io/)) or cloud (GKE, EKS, AKS)
- `kubectl` configured and pointing to your cluster
- Ingress Controller:

command: kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml

- Metrics Server (required for HPA):

command:- kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


# Deploy

command: kubectl apply -f KUBERNETES/

# Verify

# Namespace and resource limits
commands: kubectl get namespace webapp
          kubectl describe resourcequota -n webapp
          kubectl describe limitrange -n webapp
# Pods
command: kubectl get pods -n webapp

# Service and Ingress
commands: kubectl get svc -n webapp
          kubectl get ingress -n webapp

# Autoscaler
command: kubectl get hpa -n webapp

# Network policies
command: kubectl get networkpolicy -n webapp

## Access the App

command: kubectl port-forward svc/nginx-webapp-svc 8080:80 -n webapp
Visit: `http://localhost:8080`

## Test Autoscaling

# Generate load
command: kubectl run load-generator --image=busybox --restart=Never -n webapp \
  -- /bin/sh -c "while true; do wget -q -O- http://nginx-webapp-svc; done"

# Watch HPA react in another terminal
command: kubectl get hpa -n webapp -w

# Clean up
kubectl delete pod load-generator -n webapp

## Problems Faced & Fixed

Real issues hit while deploying on an actual cluster:

1. Port 80 permission denied
Non-root containers cannot bind to ports below 1024. nginx crashed with `bind() to 0.0.0.0:80 failed (13: Permission denied)`.
Fixed by setting nginx to listen on port `8080`. Service maps external port `80` → container port `8080`.

2. PID file permission denied
nginx tried to write its PID to `/var/run/nginx.pid` which is owned by root.
Fixed by adding `pid /tmp/nginx.pid;` at the top of `nginx.conf` — `/tmp` is writable by all users.

3. Cache directory chown failed
nginx tried to `chown /var/cache/nginx` at startup but the container lacked root permissions.
Fixed by adding an init container running as root (`uid 0`) that pre-creates and chowns all cache directories before the nginx container starts.

## Teardown

command: kubectl delete namespace webapp

## What I Would Add Next

- TLS via cert-manager + Let's Encrypt
- PodDisruptionBudget for safe maintenance
- VerticalPodAutoscaler (VPA) alongside HPA
- Second service (Redis) to make network policies more meaningful
- Prometheus + Grafana for monitoring
- CI/CD pipeline with Jenkins

Outputs:

  <img width="1536" height="1024" alt="kubernetes-nginx-arhitecture" src="https://github.com/user-attachments/assets/e8ff1fcf-bb15-4af2-bfa7-984665cefa40" />
  
  <img width="1260" height="701" alt="App" src="https://github.com/user-attachments/assets/7160bf6c-f2dd-49a0-bd25-f4ddd2f91fb9" />
  
  <img width="1280" height="960" alt="endpoints" src="https://github.com/user-attachments/assets/085c629e-0670-4bc0-9a99-2f81c8e8cb00" />
  
  <img width="1280" height="960" alt="hpa" src="https://github.com/user-attachments/assets/313521e8-61c6-4411-8f34-07efe812ac82" />
  
  <img width="1280" height="960" alt="port-forward" src="https://github.com/user-attachments/assets/aae015cd-c58c-41b8-b579-b856624f286e" />
  
  <img width="1280" height="960" alt="resources" src="https://github.com/user-attachments/assets/f81ff937-b90a-46c4-9b69-5dde787d33db" />


