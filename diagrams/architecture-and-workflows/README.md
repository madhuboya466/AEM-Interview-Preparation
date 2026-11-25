# Architecture and workflow diagrams

Mermaid diagrams render directly on GitHub. `aem-route-map.html` is a standalone visual overview (open it in a browser).

| File | Shows |
|------|-------|
| [aem-route-map.html](aem-route-map.html) | Full visual study page: AEMaaCS route map, Dispatcher flow, pipeline, migration, experience map |
| This file, section 1 | AEMaaCS architecture |
| Section 2 | Request route and where each error is produced |
| Section 3 | Layer isolation flowchart |
| Section 4 | Deployment lifecycle |
| Section 5 | Dispatcher decision flow |
| Section 6 | Access layers (IMS + ACL) |
| Section 7 | Support case lifecycle |
| Section 8 | Kubernetes request path and debug order |
| Section 9 | Terraform change flow around AEM |
| Section 10 | JVM diagnosis flow |

## 1. AEM as a Cloud Service architecture

```mermaid
flowchart TB
  V[Visitors] --> CDN[Adobe CDN<br/>SSL, allowlist, WAF, cache]
  AU[Authors] --> CDN
  CDN --> DP[Dispatcher + Publish pods]
  CDN --> DV[Dispatcher + Preview pods]
  CDN --> A[Author pods]
  A -->|content distribution| DP
  A -->|publish to preview| DV
  A --- REPO[(Author repository<br/>document store + blobs)]
  IMS[Adobe IMS / Admin Console] --> A
  GIT[(Git)] --> CM[Cloud Manager pipelines] -->|code + config| A & DP & DV
```

## 2. Request route and error origin

```mermaid
flowchart LR
  B[Browser] -->|DNS| CDN
  CDN -->|403 allowlist/WAF<br/>503/504 origin timeout| B
  CDN --> D[Dispatcher]
  D -->|404 filter deny<br/>301/302 rewrite<br/>502/503 backend| CDN
  D --> P[Publish / Sling]
  P -->|404 no resource or no read<br/>500 exception<br/>403 privilege| D
  P --> J[(JCR)]
```

## 3. Layer isolation

```mermaid
flowchart TD
  S[Symptom] --> A{Works on Author?}
  A -->|no| C1[Content / code: error.log on author]
  A -->|yes| P{Works on Publish default domain?}
  P -->|no| C2{Published?}
  C2 -->|no| C3[Publish / distribution queue]
  C2 -->|yes| C4[Publish render / anonymous ACL / dispatcher filter]
  P -->|yes| D{Custom domain fails?}
  D -->|yes| C5[DNS, certificate, domain mapping, vhost]
  D -->|no| C6[Cache: CDN TTL / dispatcher stat / browser]
```

## 4. Deployment lifecycle

```mermaid
flowchart LR
  F[feature branch] -->|PR + review| DV[develop]
  DV -->|Jenkins build + sync| AG[(Adobe Git)]
  AG --> NP[Non-prod pipeline<br/>dev / QA]
  DV -->|cut| RL[release branch]
  RL --> PP[Prod pipeline]
  PP --> B[Build + quality] --> ST[Stage + tests] --> AP{Approval} --> PR[Production]
  PR --> V[Validate + tag + merge back]
```

## 5. Dispatcher decision flow

```mermaid
flowchart LR
  R[Request] --> VH[vhost] --> RW[rewrite] --> F{filter}
  F -->|deny| X[404]
  F -->|allow| C{cached + fresh?}
  C -->|yes| S[serve file]
  C -->|no| P[publish] --> K{cacheable?} -->|yes| ST[store] --> S
  K -->|no| S
```

## 6. Access layers

```mermaid
flowchart LR
  U[User] --> IP{CDN IP allowlist}
  IP -->|blocked| F403[403]
  IP --> IMS{IMS product profile<br/>for this env}
  IMS -->|missing| NA[No access to env]
  IMS --> G[AEM user + groups]
  G --> ACL{ACL on path}
  ACL -->|deny| NOP[Missing pages / action fails]
  ACL --> OK[Allowed]
```

## 7. Support case lifecycle

```mermaid
flowchart TD
  O[Open] --> T[Triage + priority] --> Q[Qualify] --> I[Investigate]
  I --> S{Outcome}
  S -->|customer side| A[Advise fix / workaround]
  S -->|defect| E[Escalate to engineering] --> A
  S -->|need data| W[Wait on customer] --> I
  A --> V[Customer validates] --> C[Close + KB]
```

## 8. Kubernetes request path and debug order

```mermaid
flowchart LR
  U[User] --> ING[Ingress<br/>host, TLS] --> SVC[Service<br/>ready endpoints] --> POD[Pod<br/>probes, limits] --> APP[Dispatcher → AEM]
```

## 9. Terraform change flow around AEM

```mermaid
flowchart LR
  PR[PR: DNS / CDN / CIDR change] --> PL[terraform plan] --> RV[review + approval] --> AP[terraform apply] --> V[dig, curl, openssl] --> CM[Cloud Manager domain / allowlist]
```

## 10. JVM diagnosis flow

```mermaid
flowchart TD
  S[Slow / unstable JVM] --> G{GC log: long or frequent pauses?}
  G -->|yes| M[Heap: histogram, heap dump, MAT dominator tree]
  G -->|no| C{CPU high?}
  C -->|yes| T[top -H → hex nid → thread dump stack]
  C -->|no| TD[Thread dump series:<br/>BLOCKED, pool exhausted, waiting on I/O]
```
