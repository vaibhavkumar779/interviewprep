# Platform Engineer — ATS Score Analysis

## Current Score: 90/100

| Criteria | Score | Max | Notes |
|---|---|---|---|
| Format/Parsability | 10 | 10 | Clean DOCX, Calibri, no tables/columns/images |
| Standard Sections | 10 | 10 | All standard sections present |
| Contact Info | 10 | 10 | Complete |
| Keyword Match | 16 | 20 | Missing: Backstage, Crossplane, OPA/Gatekeeper, service catalog |
| Quantified Impact | 18 | 20 | Strong metrics |
| Skill-Role Alignment | 16 | 20 | "Platform Engineer" is newer — needs stronger IDP vocabulary |
| Consistency | 10 | 10 | Clean logical progression |

---

## Keywords Missing (High-Impact for ATS)

### Priority 1 — Must Add (core Platform Engineer vocabulary)
| Keyword | Why It Matters | Learn? |
|---|---|---|
| Backstage (Spotify) | THE developer portal — in 90% of Platform Eng JDs | ✅ Yes |
| Internal Developer Portal | Key phrase ATS scans for | ⬜ Already implied — make explicit |
| Service Catalog | Backstage core concept | ✅ Yes (part of Backstage) |
| Developer Experience (DevEx) | Platform Eng mission statement | ⬜ Add to summary |
| Platform as a Product | Key philosophy term | ⬜ Add to summary |

### Priority 2 — Should Add (60%+ of JDs)
| Keyword | Why It Matters | Learn? |
|---|---|---|
| Crossplane | Kubernetes-native IaC — platform eng standard | ✅ Yes |
| OPA / Gatekeeper | Policy-as-code for K8s | ✅ Yes |
| Kyverno | K8s policy alternative | ⬜ Awareness |
| ArgoCD | GitOps — platform eng CI/CD | ✅ Yes |
| Flux CD | GitOps alternative | ⬜ Awareness |
| Tekton | Cloud-native CI/CD pipelines | ⬜ Awareness |

### Priority 3 — Nice to Have
| Keyword | Why It Matters | Learn? |
|---|---|---|
| Port (developer portal) | Backstage alternative | ⬜ Awareness |
| Kratix | Platform orchestrator | ⬜ Awareness |
| Dagger | CI/CD containerized pipelines | ⬜ Optional |
| Score (spec) | Workload spec standard | ⬜ Awareness |
| vCluster | Virtual K8s clusters for dev | ⬜ Optional |
| Istio / Linkerd | Service mesh for platform | ✅ Yes |

---

## Bullet Point Improvements

### Current → Improved (after learning topics above)

1. **Enhance IDP bullet** (add Backstage):
   > Designed and maintained an Internal Developer Platform with Backstage developer portal, enabling self-service infrastructure provisioning and service catalog discovery across 4 product teams, reducing developer onboarding from 2 days to 2 hours.

2. **Add Crossplane bullet** (new):
   > Evaluated and implemented Crossplane for Kubernetes-native infrastructure provisioning, enabling developers to request cloud resources through kubectl without direct cloud console access.

3. **Add OPA/Gatekeeper bullet** (new):
   > Enforced platform guardrails using OPA Gatekeeper policies on AKS clusters — mandatory resource limits, approved container registries, and namespace isolation — preventing 30+ policy violations per sprint.

4. **Add ArgoCD bullet** (new):
   > Implemented ArgoCD-based GitOps delivery for platform services, enabling declarative deployments with automated drift detection and self-healing reconciliation across 5 environments.

5. **Enhance summary** (add DevEx/Platform-as-Product):
   > Platform Engineer... treating platform as a product and improving Developer Experience (DevEx) through self-service infrastructure, golden-path CI/CD, and an Internal Developer Portal.

---

## Target Score After Improvements: 96/100
