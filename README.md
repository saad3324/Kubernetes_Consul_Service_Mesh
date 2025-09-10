🌀 Consul Service Mesh on AWS EKS

This project provisions a production-ready Kubernetes cluster on AWS EKS using Terraform, and deploys a microservices-based application secured by HashiCorp Consul Service Mesh.

It demonstrates how to achieve:
✅ Automated Infrastructure with Terraform
✅ Secure Service-to-Service Communication (mTLS)
✅ Dynamic Service Discovery & Traffic Routing
✅ Failover & Multi-Cluster Peering
✅ Zero-Trust Networking with Consul Intentions

📌 Architecture
flowchart LR
    subgraph AWS EKS Cluster
      A[Frontend Service] -->|Consul Upstream| B[Checkout Service]
      B --> C[Payment Service]
      B --> D[Shipping Service]
      B --> E[Email Service]
      B --> F[Currency Service]
      B --> G[Product Catalog]
      B --> H[Cart Service --> Redis]
      I[Recommendation Service] --> G
      A --> I
      A --> J[Ad Service]
    end

    subgraph Consul Service Mesh
      K[Sidecar Proxies]
      L[Mesh Gateway]
      M[Consul UI + API]
    end

🗂️ Project Structure
k8s-consul-service-mesh/
│── terraform/                 # Terraform IaC for AWS VPC + EKS
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
│
│── consul/                    # Consul Helm values & CRDs
│   ├── consul-values.yaml
│   ├── mesh.yaml
│   ├── intentions.yaml
│   ├── resolver.yaml
│   └── exported-services.yaml
│
│── services/                  # Kubernetes microservices manifests
│   ├── frontend.yaml
│   ├── checkoutservice.yaml
│   ├── cartservice.yaml
│   ├── shippingservice.yaml
│   ├── paymentservice.yaml
│   ├── productcatalog.yaml
│   ├── emailservice.yaml
│   ├── adservice.yaml
│   ├── recommendation.yaml
│   └── redis.yaml
│
│── debug/
│   └── curl-debug-pod.yaml    # Debugging pod for testing mesh
│
└── README.md

⚡ Quick Start
1️⃣ Provision Infrastructure (Terraform)
cd terraform
terraform init
terraform apply -auto-approve
aws eks update-kubeconfig --region us-east-1 --name myapp-eks-cluster

2️⃣ Install Consul via Helm
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install consul hashicorp/consul -f consul/consul-values.yaml
kubectl get pods -l app=consul

3️⃣ Deploy Microservices
kubectl apply -f services/
kubectl get svc frontend-external


Access the app via the frontend LoadBalancer IP.

🔐 Service Mesh Features
✅ Sidecar Injection
annotations:
  consul.hashicorp.com/connect-inject: 'true'

✅ Upstream Dependencies
annotations:
  consul.hashicorp.com/connect-service-upstreams: 'productcatalogservice:3550,shippingservice:50052'

✅ Intentions (Zero-Trust Networking)
kind: ServiceIntentions
spec:
  destination:
    name: shippingservice
  sources:
   - name: frontend
     action: allow

✅ Failover with ServiceResolvers
kind: ServiceResolver
spec:
  failover:
    '*':
      targets:
        - peer: 'DOK'

🛠 Debugging

Deploy curl pod:

kubectl apply -f debug/curl-debug-pod.yaml
kubectl exec -it curl-debug-pod -- sh
curl http://checkoutservice:5050

📊 Observability

Consul UI available via LoadBalancer

Visualize service topology, upstreams, and intentions

Extend with Prometheus + Grafana for metrics

🎯 Learning Outcomes

By completing this project, you will learn how to:
✔ Deploy a Kubernetes cluster on AWS using Terraform
✔ Integrate Consul Service Mesh with Kubernetes
✔ Secure traffic between microservices with mTLS
✔ Implement service discovery, failover, and traffic policies
✔ Debug and monitor a service mesh-enabled microservices app

👤 Maintainer

Muhammad Saad
DevOps Engineer
