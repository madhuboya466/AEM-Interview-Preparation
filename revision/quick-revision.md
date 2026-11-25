# Quick revision: all six modules

> One page to read the day before the interview. Each line is a fact you should be able to explain in your own words, with an example.

## The big picture
- Route: **Browser → DNS → CDN → Dispatcher → Publish → AEM application → JCR**; **Author → Preview / Publish** by content distribution; **Git → Cloud Manager → all tiers**.
- Troubleshooting = find **which stop produced the response**. Compare working vs failing (env, user, page, time).

## 1. Production support
- Families: application, infrastructure, content. ITIL: incident, problem, change, service request.
- Priority = impact × urgency; P1–P4; SLA = response time + update cadence.
- Lifecycle: detect → log → prioritise → respond → diagnose → fix/workaround → validate → close → RCA.
- 403: CDN allowlist/WAF or AEM privilege. 404: Dispatcher filter (case-sensitive) or not published / no read. 500: exception → last `Caused by`. 502/503/504: backend bad / unavailable / timeout. Redirect loop: read each `Location` hop.
- Logs: error, request (`[n] ->` / `<- status ms`), access, audit + replication (6.5), dispatcher (httpd access/error, dispatcher), CDN, Cloud Manager. Correlate by UTC time, URL, request ID, client IP, user, recent change.
- RCA: symptom ≠ root cause; Five Whys; evidence for each claim; corrective + preventive actions with owners.

## 2. DevOps
- Git: feature → develop → release → main; hotfix from main and merge back; PR + review; resolve conflicts then rebuild.
- Jenkins: controller + agents; Jenkinsfile stages; credentials store; triggers poll/webhook/upstream/manual; **one trigger per change** avoids duplicates.
- Maven: GAV; scopes (AEM SDK API `provided`); validate → compile → test → package → verify → install → deploy; `dependency:tree`, `-U`.
- AEM modules: core, ui.apps (immutable), ui.config (OSGi + Repoinit), ui.content (mutable), ui.frontend, dispatcher, all.
- Cloud Manager prod pipeline: build → quality → image → Stage → tests → approval → Prod (rolling). Types: full-stack, web-tier, front-end, config. RDE via aio CLI.
- Change control: ticket, risk, evidence, window, approvers, rollback plan, validation. Rollback in cloud = redeploy previous commit.

## 3. Administration
- AEMaaCS: Adobe runs infrastructure and updates; customer owns code, content, users, domains, allowlists.
- Users via IMS + Admin Console product profiles → AEM groups → ACLs. ACLs inherit; deeper wins; user beats group at same node.
- Service users: Repoinit + ServiceUserMapper; least privilege.
- Publication: 6.5 replication agents/queues/flush; cloud distribution + Distribution console + automatic invalidation.
- Repository: /apps /libs /conf /content /etc /var /home /oak:index. Namespaces registered before use.
- Dispatcher: vhost → rewrite → filter → cache; `.stat` + statfileslevel; ignoreUrlParams; CDN TTL separate.

## 4. Internals
- Sling: path.selectors.extension/suffix → resource → resourceType → script (resourceSuperType chain, /apps before /libs).
- Sling Models: `@Model`, `@ValueMapValue`, `@ChildResource`, `@OSGiService`, `@ScriptVariable`, `@Self @Via(ResourceSuperType)`; failed required injection → null.
- Servlets: resource-type registration preferred over path. Filters: REQUEST/COMPONENT/INCLUDE/FORWARD/ERROR.
- OSGi: bundle INSTALLED → RESOLVED → ACTIVE; component active / satisfied / unsatisfied / failed; `@Reference`, `@Activate`, configs per run mode, `$[env:]`, `$[secret:]`.
- JCR/Oak: nodes, properties, types, sessions, MVCC, segment vs document store, indexes, traversal warnings.
- HTL: use, test, list, resource, include, template/call. Each component include = sub-request.
- Workflows: model (Sync to /var), payload, steps, launcher, instance, `WorkflowProcess`.

## 5. JVM
- Heap (objects, `-Xmx`) vs stack (frames, per thread); Young → Old; G1; stop-the-world pauses.
- Pressure recovers; leak = after-GC baseline climbs. OOM types: heap space, GC overhead, Metaspace, native thread.
- Heap dump: `jcmd PID GC.heap_dump`; MAT histogram, dominator tree (retained heap), path to GC roots; compare two dumps.
- Thread dumps: `jcmd PID Thread.print`, series 5–10 s apart; RUNNABLE / BLOCKED / WAITING / TIMED_WAITING; AEM thread name = URL; deadlock printed at end.
- High CPU: `top -H -p PID` → `printf '%x'` → nid in dump.
- Performance factors: cache hit ratio, render cost, queries/indexes, external latency, GC.

## 6. Support process
- Lifecycle: open → triage → own → qualify → investigate → solve / escalate / wait → validate → close → knowledge.
- Defect only if reproducible without customer code, contradicts docs, not known.
- Escalation: title, impact, env, steps, expected/actual, evidence, ruled out, workaround.
- Communication: acknowledge, clear, specific, honest, owned, timed updates; restore first in P1.
- Knowledge: KB with symptom words, case notes, RCA, runbooks.

## 7. Kubernetes
- Image (immutable) → container; control plane (apiserver, etcd, scheduler, controllers) + nodes (kubelet, kube-proxy, runtime); desired state + reconciliation.
- Pod, ReplicaSet, Deployment, StatefulSet, DaemonSet, Job/CronJob, Service, Ingress, ConfigMap/Secret, PV/PVC, HPA, Namespace, RBAC.
- Probes: startup, readiness (traffic), liveness (restart). Limits: CPU throttled, memory OOMKilled (137) ≠ Java OOM.
- Rollouts: rolling, blue/green, canary; `kubectl rollout status|history|undo`.
- AEMaaCS lens: image = your code + AEM release → immutable /apps; env vars restart pods; publish scales horizontally; logs collected centrally; no exec for customers.
- Debug: `get`, `describe` (events), `logs --previous`, `get endpoints`, `top`, `port-forward`. Pending, ImagePullBackOff, CrashLoopBackOff, OOMKilled, 0/1 Ready.

## 8. Terraform
- IaC, declarative; providers, resources, data sources, state, plan (+ ~ - -/+), apply.
- `init` → `fmt`/`validate` → `plan -out` → review → `apply plan` → verify.
- Remote state with locking; never commit state; drift via `plan -refresh-only`; `import`, `moved`, `state mv/rm`.
- Modules + one state per environment; `for_each` over `count` for named things; `prevent_destroy` on prod DNS.
- Around AEMaaCS: CNAME to `cdn.adobeaemcloud.com` + TXT verification, customer CDN, certificates, CIDR lists, Jenkins/GitHub, monitoring. Cloud Manager via UI/API/aio.
- Self-hosted AEM 6.5: VPC, security groups, VMs, load balancer, certificates, blob storage, DNS.

## Your stories (one line each, [Hands-on])
| Story | Concept it proves |
|-------|-------------------|
| Regional authors 403 → IP not in allowlist → CIDR added | Layer isolation, CDN access control |
| `.JPG` / `.xlsx` 404 → filter extension list | Dispatcher filter, case-sensitive regex, deny = 404 |
| Ingestion wiped workflow models (empty CSV rows) → revert, re-ingest, peer review | Migration (wipe), workflow storage, RCA + prevention |
| Namespace errors → Repoinit across RDE/QA/STAGE/PROD | Repository as code, JCR namespaces |
| Heavy-page 503 → too many containers → re-authored | Render cost, timeouts, authoring guidance |
| SMTP token expired after 90 days inactivity → new token, monthly refresh | OSGi mail config, OAuth tokens, secrets |
| `.html` append rule for preview links | Sling extensions, Dispatcher rewrites |
| Domain mapping, SSL, DNS to Adobe CDN | Go-live chain |
| Duplicate pipeline triggers from repo sync | CI/CD triggers |
| List v3 → v4, teaser delegate | Core Components, proxy, delegation, policies |
| Preview-to-publish workflow | Preview tier, workflows |
| Country-site form with email + BCC | Core form container, mail action |
| Country author/approver groups | IMS, groups, ACLs, least privilege |
