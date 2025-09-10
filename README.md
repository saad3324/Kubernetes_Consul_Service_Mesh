Consul Service Mesh on AWS EKS

This project provisions an AWS EKS cluster with Terraform and deploys Consul as a service mesh to manage microservices communication, security, and observability.

📌 Overview

Infrastructure as Code: Uses Terraform to create VPC, subnets, NAT gateway, and an EKS cluster.

Service Mesh: HashiCorp Consul for service discovery, secure communication (mTLS), failover, and traffic management.

Microservices: Deploys a sample application (based on Google’s microservices demo
) including:

frontend (public entrypoint via LoadBalancer)

checkoutservice, cartservice, productcatalogservice, shippingservice, paymentservice, currencyservice, emailservice, adservice, recommendationservice

redis-cart for cart storage

Consul Add-Ons:

Connect Injector (automatic sidecar proxy injection)

Mesh Gateway for cross-cluster traffic

UI exposed via LoadBalancer

🏗️ Infrastructure Setup with Terraform
1. Configure AWS Credentials

Export AWS credentials or provide via terraform.tfvars:

aws_region           = "us-east-1"
aws_access_key_id    = "<your-access-key>"
aws_secret_access_key = "<your-secret-key>"

vpc_cidr_block             = "10.0.0.0/16"
private_subnet_cidr_blocks  = ["10.0.1.0/24", "10.0.2.0/24"]
public_subnet_cidr_blocks   = ["10.0.101.0/24", "10.0.102.0/24"]

k8s_cluster_name = "myapp-eks-cluster"
k8s_version      = "1.27"

2. Deploy Infra
terraform init
terraform plan
terraform apply -auto-approve

3. Configure kubectl
aws eks update-kubeconfig --region us-east-1 --name myapp-eks-cluster
kubectl get nodes

🚀 Deploy Consul Service Mesh
1. Install Consul with Helm

Add the Helm repo:

helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update


Install Consul:

helm install consul hashicorp/consul -f consul-values.yaml


Where consul-values.yaml includes:

global:
  name: consul
  datacenter: dc1
  tls:
    enabled: true
  acls:
    manageSystemACLs: true

server:
  replicas: 3

ui:
  enabled: true
  service:
    type: LoadBalancer

connectInject:
  enabled: true
  default: true


Verify Consul pods:

kubectl get pods -n default -l app=consul

📦 Deploy Microservices

Apply Kubernetes manifests:

kubectl apply -f services/


Where services/ contains all deployments and services (frontend, checkout, cart, etc.).

Expose frontend:

kubectl get svc frontend-external


Access frontend via the LoadBalancer external IP.

🔐 Consul Service Mesh Features
1. Sidecar Injection

Enable Consul connect injection in services with annotations:

annotations:
  consul.hashicorp.com/connect-inject: 'true'

2. Service Upstreams

Example: checkout service connecting to dependencies

annotations:
  consul.hashicorp.com/connect-service-upstreams: 'productcatalogservice:3550,shippingservice:50052'

3. Service Intentions (Traffic Policy)

Control access between services:

apiVersion: consul.hashicorp.com/v1alpha1
kind: ServiceIntentions
metadata:
  name: shipping-allow-eks
spec:
  destination:
    name: shippingservice
  sources:
   - name: frontend
     action: allow

4. Failover with ServiceResolvers
apiVersion: consul.hashicorp.com/v1alpha1
kind: ServiceResolver
metadata:
  name: shippingservice
spec:
  failover:
    '*':
      targets:
        - peer: 'DOK'

🛠️ Debugging

Deploy a curl debug pod:

kubectl apply -f debug/curl-debug-pod.yaml
kubectl exec -it curl-debug-pod -- sh
curl http://checkoutservice:5050

📊 Observability

Use Consul UI (via LoadBalancer) to visualize services, intentions, and traffic.

Integrate Prometheus/Grafana for deeper metrics.

📌 Next Steps

Enable Consul ACLs in production.

Configure multi-cluster federation using Mesh Gateways.

Add CI/CD with GitHub Actions or Jenkins for automated deployment.

Integrate HashiCorp Vault for secret management.

👤 Maintainer

Muhammad Saad
DevOps Engineer – Innovent.io
