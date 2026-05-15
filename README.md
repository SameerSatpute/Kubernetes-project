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
HPA — CPU + Memory autoscaling `hpa.yaml`


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
