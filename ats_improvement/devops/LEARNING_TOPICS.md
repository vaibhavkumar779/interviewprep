# DevOps ATS Improvement — Learning Topics

> After studying each topic, you can add the corresponding bullet points from `README.md` to your resume.

---

## Topic 1: ArgoCD & GitOps

### What to Learn
- ArgoCD architecture: Application CRD, App of Apps pattern, ApplicationSet
- GitOps principles: declarative config, Git as single source of truth, reconciliation loop
- ArgoCD vs Flux CD — when to use which
- ArgoCD sync strategies: auto-sync, manual sync, sync waves, hooks
- ArgoCD with Helm charts and Kustomize
- Multi-environment promotion with ArgoCD (dev → staging → prod)
- ArgoCD RBAC and SSO integration

### Hands-On Practice
```bash
# Install ArgoCD on a local kind/minikube cluster
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Create an Application
argocd app create my-app \
  --repo https://github.com/your-repo/k8s-manifests.git \
  --path dev \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated
```

### Interview Questions to Prepare
1. What is GitOps and how does ArgoCD implement it?
2. Explain ArgoCD sync strategies — auto vs manual, sync waves, hooks
3. How do you handle secrets in a GitOps workflow? (Sealed Secrets, SOPS, External Secrets Operator)
4. ArgoCD App of Apps pattern — when and why?
5. How do you promote changes across environments with ArgoCD?
6. ArgoCD vs FluxCD — differences, pros/cons
7. How does ArgoCD handle drift detection?

### Resume Bullet (add after learning)
> Implemented GitOps-based continuous delivery using ArgoCD for Kubernetes workloads, enabling declarative deployments with automated sync, drift detection, and self-healing reconciliation across 5 namespaces.

---

## Topic 2: HashiCorp Vault

### What to Learn
- Vault architecture: sealing/unsealing, backends, policies
- Secret engines: KV v2, dynamic secrets (database, AWS, Azure)
- Vault auth methods: Kubernetes, AppRole, OIDC
- Vault Agent sidecar injection in Kubernetes
- Vault vs Azure Key Vault — comparison and when to use which
- Vault Helm chart deployment on K8s
- Vault with Terraform (vault provider)

### Hands-On Practice
```bash
# Run Vault in dev mode locally
vault server -dev

# Enable KV v2 engine
vault secrets enable -path=secret kv-v2

# Store and retrieve a secret
vault kv put secret/myapp db_password="s3cret"
vault kv get secret/myapp

# Kubernetes auth method
vault auth enable kubernetes
vault write auth/kubernetes/config \
  kubernetes_host="https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT"
```

### Interview Questions to Prepare
1. How does Vault seal/unseal work? What is auto-unseal?
2. What are dynamic secrets and why are they better than static secrets?
3. How does Vault Agent sidecar injection work in Kubernetes?
4. Vault vs Azure Key Vault vs AWS Secrets Manager — comparison
5. How do you manage Vault policies for least-privilege access?
6. How does Vault integrate with Terraform?

### Resume Bullet (add after learning)
> Evaluated HashiCorp Vault and Azure Key Vault for enterprise secrets management; implemented Azure Key Vault with CSI Driver for Kubernetes workloads while using Vault for dynamic database credentials, eliminating 100% of static secrets.

---

## Topic 3: Istio Service Mesh

### What to Learn
- Istio architecture: Envoy sidecar, istiod (Pilot, Citadel, Galley)
- Traffic management: VirtualService, DestinationRule, Gateway
- mTLS and security policies
- Canary deployments with Istio
- Observability integration: Kiali, Jaeger, Prometheus (you already use these)
- Istio vs Linkerd — when to use which

### Hands-On Practice
```bash
# Install Istio
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled

# Deploy sample app
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/bookinfo/platform/kube/bookinfo.yaml

# Create traffic routing
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 90
    - destination:
        host: reviews
        subset: v2
      weight: 10
EOF
```

### Interview Questions to Prepare
1. What is a service mesh and why do you need one?
2. Explain Istio sidecar injection — how does it work?
3. How does Istio enable mTLS between services?
4. Explain VirtualService vs DestinationRule
5. How do you implement canary deployments with Istio?
6. Istio vs Linkerd — pros/cons
7. How does Istio integrate with Prometheus/Grafana/Jaeger?

### Resume Bullet (add after learning)
> Configured Istio service mesh for 13 microservices enabling mutual TLS, traffic management, and canary deployments with Kiali visualization and Jaeger distributed tracing.

---

## Topic 4: ELK Stack / OpenSearch

### What to Learn
- ELK architecture: Elasticsearch, Logstash, Kibana
- Filebeat/Fluentd/Fluent Bit as log shippers
- Log aggregation patterns in Kubernetes (DaemonSet vs sidecar)
- OpenSearch vs Elasticsearch (licensing, features)
- Index management, ILM policies
- Kibana dashboards and alerting
- EFK stack (Elasticsearch + Fluentd + Kibana) on K8s

### Hands-On Practice
```bash
# Deploy EFK on K8s using Helm
helm repo add elastic https://helm.elastic.co
helm install elasticsearch elastic/elasticsearch --set replicas=1
helm install kibana elastic/kibana
helm install filebeat elastic/filebeat

# Or use Fluent Bit (lighter)
helm repo add fluent https://fluent.github.io/helm-charts
helm install fluent-bit fluent/fluent-bit
```

### Interview Questions to Prepare
1. ELK vs EFK — what's the difference and when to use which?
2. How does Filebeat vs Fluentd vs Fluent Bit differ?
3. How do you handle log aggregation in Kubernetes?
4. What are Elasticsearch index lifecycle management (ILM) policies?
5. How do you set up alerts in Kibana?
6. OpenSearch vs Elasticsearch — why did the fork happen?

### Resume Bullet (add after learning)
> Deployed ELK/EFK stack on AKS for centralized log aggregation using Fluent Bit DaemonSets, Elasticsearch, and Kibana — enabling log-based alerting and reducing troubleshooting time by 50%.

---

## Topic 5: Packer (Image Building)

### What to Learn
- Packer HCL2 templates
- Builders: Azure (azure-arm), AWS (amazon-ebs), Docker
- Provisioners: shell, Ansible, file
- Post-processors: manifest, docker-push
- Golden image pipeline: Packer + CI/CD
- Packer with Terraform workflow

### Hands-On Practice
```hcl
# packer/azure-ubuntu.pkr.hcl
source "azure-arm" "ubuntu" {
  subscription_id = var.subscription_id
  managed_image_resource_group_name = "packer-images"
  managed_image_name = "ubuntu-golden-{{timestamp}}"
  os_type = "Linux"
  image_publisher = "Canonical"
  image_offer = "0001-com-ubuntu-server-jammy"
  image_sku = "22_04-lts"
  vm_size = "Standard_B2s"
}

build {
  sources = ["source.azure-arm.ubuntu"]
  provisioner "shell" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y docker.io nginx"
    ]
  }
}
```

### Interview Questions to Prepare
1. What is Packer and how does it differ from Docker?
2. Explain the Packer build workflow (template → build → image)
3. How do you create golden images with Packer for Azure/AWS?
4. How does Packer integrate with Terraform?
5. What provisioners does Packer support?

### Resume Bullet (add after learning)
> Built golden VM image pipeline using Packer with Ansible provisioners and Azure Pipelines, producing hardened Ubuntu images for AKS node pools and VM workloads.

---

## Topic 6: CloudFormation (AWS IaC)

### What to Learn
- CloudFormation template structure: Resources, Parameters, Outputs, Mappings
- Stacks and StackSets
- Change sets and drift detection
- CloudFormation vs Terraform — comparison
- CDK (Cloud Development Kit) basics
- Nested stacks and cross-stack references

### Hands-On Practice
```yaml
# simple-ec2.yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
      ImageId: ami-0c55b159cbfafe1f0
      Tags:
        - Key: Name
          Value: CFN-Demo
Outputs:
  InstanceId:
    Value: !Ref MyEC2Instance
```

### Interview Questions to Prepare
1. CloudFormation vs Terraform — when to use which?
2. What is a Change Set in CloudFormation?
3. How do you handle drift detection?
4. What are nested stacks and when to use them?
5. Explain CloudFormation StackSets for multi-account deployments

### Resume Bullet (add after learning)
> Managed AWS infrastructure using both Terraform and CloudFormation, leveraging StackSets for multi-account deployments and Change Sets for safe production updates.

---

## Study Priority Order
1. **ArgoCD & GitOps** — highest JD frequency, biggest ATS impact
2. **Istio Service Mesh** — you already use Kiali/Jaeger, Istio is the missing link
3. **ELK Stack** — log aggregation is expected in every DevOps role
4. **HashiCorp Vault** — complements your Key Vault experience
5. **CloudFormation** — shows multi-cloud IaC depth
6. **Packer** — nice differentiator
