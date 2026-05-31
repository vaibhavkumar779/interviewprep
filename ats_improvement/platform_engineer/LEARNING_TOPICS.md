# Platform Engineer ATS Improvement — Learning Topics

> After studying each topic, add the corresponding bullet points from `ATS_ANALYSIS.md` to your resume.

---

## Topic 1: Backstage (Spotify Developer Portal)

### What to Learn
- Backstage architecture: software catalog, scaffolder, TechDocs, plugins
- Service catalog: entities, kinds (Component, API, Resource, System, Domain)
- Software templates (Scaffolder) for golden-path project creation
- TechDocs: docs-as-code integrated into the portal
- Backstage plugins ecosystem (Kubernetes, ArgoCD, Grafana, PagerDuty)
- Setting up Backstage with PostgreSQL backend
- Backstage RBAC and ownership model

### Hands-On Practice
```bash
# Create a Backstage app
npx @backstage/create-app@latest

# Configure catalog
# catalog-info.yaml
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: my-service
  description: My microservice
  annotations:
    backstage.io/kubernetes-id: my-service
    backstage.io/techdocs-ref: dir:.
spec:
  type: service
  lifecycle: production
  owner: team-platform
  system: digital-identity
  providesApis:
    - my-service-api

# Software Template for golden-path
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: microservice-template
  title: Create Microservice
spec:
  owner: team-platform
  type: service
  parameters:
    - title: Service Details
      properties:
        name:
          type: string
        description:
          type: string
  steps:
    - id: fetch-template
      action: fetch:template
      input:
        url: ./skeleton
    - id: publish
      action: publish:github
```

### Interview Questions to Prepare
1. What is Backstage and what problems does it solve for platform teams?
2. Explain the Backstage software catalog — entities, kinds, ownership model
3. How do software templates (Scaffolder) enable golden-path project creation?
4. What is TechDocs and how does it work?
5. How do you integrate Backstage with Kubernetes and ArgoCD?
6. Backstage vs Port vs Cortex — developer portal comparison
7. How do you manage RBAC and ownership in Backstage?

### Resume Bullet (add after learning)
> Built Internal Developer Portal using Backstage with service catalog, golden-path templates, and TechDocs integration, providing self-service discovery for 13+ microservices and reducing developer onboarding from 2 days to 2 hours.

---

## Topic 2: Crossplane (Kubernetes-Native IaC)

### What to Learn
- Crossplane architecture: providers, managed resources, composite resources
- Crossplane vs Terraform — when to use which
- Compositions and CompositeResourceDefinitions (XRDs)
- Crossplane providers: AWS, Azure, GCP, Kubernetes, Helm
- Claims (XRC) — self-service infrastructure requests
- Crossplane with ArgoCD for GitOps-managed infrastructure

### Hands-On Practice
```bash
# Install Crossplane
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm install crossplane crossplane-stable/crossplane --namespace crossplane-system --create-namespace

# Install Azure provider
kubectl apply -f - <<EOF
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-azure
spec:
  package: xpkg.upbound.io/upbound/provider-family-azure:v1.0.0
EOF

# Create a Composition for a "Database" abstraction
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: xdatabases.platform.example.com
spec:
  group: platform.example.com
  names:
    kind: XDatabase
    plural: xdatabases
  claimNames:
    kind: Database
    plural: databases
  versions:
    - name: v1alpha1
      served: true
      referenceable: true
```

### Interview Questions to Prepare
1. What is Crossplane and how does it differ from Terraform?
2. Explain Compositions and CompositeResourceDefinitions (XRDs)
3. How do Claims enable self-service infrastructure for developers?
4. How does Crossplane integrate with ArgoCD?
5. When would you choose Crossplane over Terraform and vice versa?
6. What are Crossplane providers?

### Resume Bullet (add after learning)
> Implemented Crossplane for Kubernetes-native infrastructure provisioning, enabling developers to request cloud resources (databases, storage, caches) via kubectl Claims without direct cloud console access.

---

## Topic 3: OPA Gatekeeper / Kyverno (Policy as Code)

### What to Learn
- OPA (Open Policy Agent) architecture and Rego language
- Gatekeeper: OPA for Kubernetes admission control
- ConstraintTemplates and Constraints
- Common policies: resource limits, allowed registries, required labels
- Kyverno: Kubernetes-native policy engine (YAML-based, no Rego)
- Kyverno vs Gatekeeper — comparison
- Policy testing and CI integration

### Hands-On Practice
```bash
# Install Gatekeeper
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.15/deploy/gatekeeper.yaml

# Require resource limits
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlimits
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLimits
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlimits
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not container.resources.limits
          msg := sprintf("Container %v has no resource limits", [container.name])
        }

---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLimits
metadata:
  name: require-limits
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]

# Kyverno alternative (YAML-based)
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-labels
spec:
  validationFailureAction: Enforce
  rules:
  - name: check-team-label
    match:
      resources:
        kinds: ["Deployment"]
    validate:
      message: "label 'team' is required"
      pattern:
        metadata:
          labels:
            team: "?*"
```

### Interview Questions to Prepare
1. What is OPA and how does Gatekeeper bring it to Kubernetes?
2. Explain ConstraintTemplates and Constraints in Gatekeeper
3. What common platform policies should you enforce? (resource limits, image registries, labels)
4. Kyverno vs Gatekeeper — when to use which?
5. How do you test policies before deploying them?
6. How does policy-as-code fit into the platform engineering workflow?

### Resume Bullet (add after learning)
> Enforced platform guardrails using OPA Gatekeeper policies on AKS — mandatory resource limits, approved container registries, required labels, and namespace isolation — preventing 30+ policy violations per sprint.

---

## Topic 4: ArgoCD & GitOps (Platform Perspective)

### What to Learn
- ArgoCD for platform teams: App of Apps, ApplicationSet
- Multi-tenant ArgoCD: projects, RBAC, SSO
- ArgoCD + Helm + Kustomize for environment management
- ArgoCD Image Updater for automatic image promotions
- ArgoCD Notifications for Slack/Teams integration
- Progressive delivery with ArgoCD Rollouts

### Hands-On Practice
```bash
# App of Apps pattern
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: platform-apps
  namespace: argocd
spec:
  project: platform
  source:
    repoURL: https://github.com/your-org/platform-apps
    path: apps
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true

# ApplicationSet for multi-env
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: my-service
spec:
  generators:
  - list:
      elements:
      - cluster: dev
        namespace: dev
      - cluster: staging
        namespace: staging
      - cluster: prod
        namespace: prod
  template:
    metadata:
      name: 'my-service-{{cluster}}'
    spec:
      source:
        repoURL: https://github.com/your-org/k8s-manifests
        path: 'envs/{{cluster}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{namespace}}'
```

### Interview Questions to Prepare
1. How does ArgoCD App of Apps pattern work for platform teams?
2. What is ApplicationSet and how does it help multi-environment management?
3. How do you implement multi-tenancy in ArgoCD?
4. ArgoCD Image Updater — how does automatic image promotion work?
5. How do you implement progressive delivery with ArgoCD Rollouts?

### Resume Bullet (add after learning)
> Implemented ArgoCD-based GitOps delivery with App of Apps pattern and ApplicationSet for multi-environment promotion, enabling declarative deployments with automated drift detection across 5 environments.

---

## Topic 5: Istio / Linkerd Service Mesh

### What to Learn
- Service mesh concepts: sidecar proxy, data plane, control plane
- Istio: VirtualService, DestinationRule, Gateway, mTLS
- Linkerd: lighter alternative, Rust-based proxy
- Traffic management: canary, blue-green, circuit breaking
- Service mesh observability (you already know Kiali + Jaeger)
- When to use service mesh vs when it's overkill

### Resume Bullet (add after learning)
> Deployed Istio service mesh across AKS clusters for mTLS enforcement, traffic management, and canary deployments, with Kiali and Jaeger providing service-level observability for 13 microservices.

---

## Study Priority Order
1. **Backstage** — #1 differentiator for Platform Engineer roles
2. **ArgoCD & GitOps** — essential for platform delivery
3. **OPA Gatekeeper / Kyverno** — platform guardrails
4. **Crossplane** — Kubernetes-native IaC (hot skill)
5. **Istio** — you already use Kiali/Jaeger, just add the mesh layer
