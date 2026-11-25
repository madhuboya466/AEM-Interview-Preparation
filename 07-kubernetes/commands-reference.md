# Kubernetes and container command reference

> Companion to [kubernetes-for-aem.md](kubernetes-for-aem.md). Everything you type or write when working with containers and Kubernetes: Docker, kubectl, YAML manifests, Helm, Kustomize, and the debugging recipes that combine them. Examples use the namespace `site-prod` and illustrative names.

---

## Contents

1. [Docker](#1-docker)
2. [kubectl basics: contexts, namespaces, output](#2-kubectl-basics-contexts-namespaces-output)
3. [Reading resources](#3-reading-resources)
4. [Creating and changing resources](#4-creating-and-changing-resources)
5. [Deployments, rollouts and scaling](#5-deployments-rollouts-and-scaling)
6. [Logs, exec, copy and port-forward](#6-logs-exec-copy-and-port-forward)
7. [Config, secrets and storage commands](#7-config-secrets-and-storage-commands)
8. [Networking commands](#8-networking-commands)
9. [Nodes, resources and cluster health](#9-nodes-resources-and-cluster-health)
10. [Access control (RBAC)](#10-access-control-rbac)
11. [YAML manifest anatomy](#11-yaml-manifest-anatomy)
12. [Helm](#12-helm)
13. [Kustomize](#13-kustomize)
14. [Local clusters: kind and minikube](#14-local-clusters-kind-and-minikube)
15. [Debugging recipes](#15-debugging-recipes)
16. [One-page cheat sheet](#16-one-page-cheat-sheet)

---

## 1. Docker

### 1.1 Images
| Command | Does |
|---------|------|
| `docker build -t myorg/dispatcher:1.4.0 .` | Build image from `Dockerfile` in current folder |
| `docker build -f docker/Dockerfile.publish -t myorg/publish:1.0 .` | Build with a specific Dockerfile |
| `docker build --build-arg JAVA_VERSION=21 -t app .` | Pass build arguments |
| `docker images` / `docker image ls` | List local images |
| `docker pull httpd:2.4` | Download image |
| `docker tag app:1.0 registry.example.com/site/app:1.0` | Add a registry name/tag |
| `docker login registry.example.com` | Authenticate to a registry |
| `docker push registry.example.com/site/app:1.0` | Upload image |
| `docker rmi app:1.0` | Remove image |
| `docker image inspect app:1.0` | Layers, env, entrypoint, labels |
| `docker history app:1.0` | Layer-by-layer size |
| `docker image prune -a` | Delete unused images |

### 1.2 Containers
| Command | Does |
|---------|------|
| `docker run -d --name disp -p 8080:80 myorg/dispatcher:1.4.0` | Run detached, map host 8080 → container 80 |
| `docker run -it --rm alpine sh` | Interactive throwaway container |
| `docker run -e AEM_HOST=publish -v $(pwd)/conf:/etc/httpd/conf.d app` | Env variable + bind mount |
| `docker run --memory 2g --cpus 1.5 app` | Resource limits (cgroups) |
| `docker ps` / `docker ps -a` | Running / all containers |
| `docker logs -f --tail 200 disp` | Follow logs |
| `docker exec -it disp sh` | Shell inside running container |
| `docker cp disp:/var/log/httpd/error_log .` | Copy file out |
| `docker stop disp` / `docker start disp` / `docker restart disp` | Lifecycle |
| `docker rm -f disp` | Remove container |
| `docker inspect disp` | Full config: IP, mounts, env, exit code, OOMKilled flag |
| `docker stats` | Live CPU/memory per container |
| `docker top disp` | Processes inside |
| `docker events` | Daemon events (start, die, oom) |
| `docker system df` / `docker system prune` | Disk use / cleanup |

### 1.3 Networks, volumes, compose
| Command | Does |
|---------|------|
| `docker network create aemnet` / `docker network ls` | User network (containers reach each other by name) |
| `docker run --network aemnet --name publish ...` | Attach container |
| `docker volume create repo` / `docker volume ls` | Named volume |
| `docker compose up -d` / `down` / `logs -f` / `ps` | Multi-container app from `compose.yaml` |

### 1.4 Dockerfile instructions
```dockerfile
FROM eclipse-temurin:21-jre        # base image
LABEL owner="platform-team"
ARG APP_VERSION=1.0                # build-time variable
ENV JAVA_OPTS="-XX:MaxRAMPercentage=70"   # runtime env
WORKDIR /opt/app
COPY target/app.jar app.jar        # copy files into image
RUN useradd -r app && chown -R app /opt/app   # run at build time
USER app                           # don't run as root
EXPOSE 8080                        # documented port
HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1
ENTRYPOINT ["sh","-c","java $JAVA_OPTS -jar app.jar"]
```
Multi-stage build: `FROM maven:3.9-eclipse-temurin-21 AS build` → `RUN mvn -B package` → second `FROM` copies only the jar (`COPY --from=build ...`), keeping the final image small.

AEM link: the AEM **Dispatcher SDK** ships scripts (`bin/validate.sh`, `bin/docker_run.sh`) that run Apache + Dispatcher in Docker against your local publish, e.g. `./bin/docker_run.sh src host.docker.internal:4503 8080`.

---

## 2. kubectl basics: contexts, namespaces, output

```bash
kubectl version                          # client and server versions
kubectl cluster-info                     # API server address
kubectl config get-contexts              # clusters/users you can talk to
kubectl config current-context
kubectl config use-context prod-cluster  # switch cluster  (check twice before prod!)
kubectl config set-context --current --namespace=site-prod   # default namespace
kubectl api-resources                    # every resource type + short names
kubectl explain deployment.spec.strategy # built-in documentation of any field
```

**Short names**: `po` pods, `deploy` deployments, `rs` replicasets, `sts` statefulsets, `ds` daemonsets, `svc` services, `ing` ingress, `cm` configmaps, `ns` namespaces, `no` nodes, `pv`/`pvc`, `sa` serviceaccounts, `hpa`, `ep` endpoints, `netpol`.

**Common flags**:
| Flag | Meaning |
|------|---------|
| `-n site-prod` / `--namespace` | Namespace |
| `-A` / `--all-namespaces` | All namespaces |
| `-o wide` | Extra columns (node, IP) |
| `-o yaml` / `-o json` | Full object |
| `-o name` | Just `kind/name` |
| `-o jsonpath='{.items[*].metadata.name}'` | Extract fields |
| `-o custom-columns=NAME:.metadata.name,IMG:.spec.containers[0].image` | Table you design |
| `-l app=publish` / `--selector` | Filter by label |
| `--field-selector status.phase=Running` | Filter by field |
| `-w` / `--watch` | Stream changes |
| `--sort-by=.metadata.creationTimestamp` | Sort |
| `--dry-run=client -o yaml` | Print what would be created, without creating |

---

## 3. Reading resources

```bash
kubectl get pods                              # pods in default namespace
kubectl get pods -n site-prod -o wide         # + node and pod IP
kubectl get pods -l app=publish --show-labels
kubectl get deploy,rs,po,svc,ing -n site-prod # several kinds at once
kubectl get all -n site-prod                  # common kinds (not configmaps/secrets)
kubectl get pod publish-7c9d -o yaml          # full spec + status
kubectl describe pod publish-7c9d             # human summary + EVENTS (most useful)
kubectl describe deploy publish
kubectl get events -n site-prod --sort-by=.lastTimestamp
kubectl get events --field-selector type=Warning -A
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.containerStatuses[*].restartCount}{"\n"}{end}'
kubectl get pods --field-selector=status.phase!=Running -A   # anything unhealthy
```

---

## 4. Creating and changing resources

| Command | Style | Use |
|---------|-------|-----|
| `kubectl apply -f deploy.yaml` | Declarative | **Normal way**: create or update to match the file |
| `kubectl apply -f k8s/` / `-k overlays/prod` | Declarative | A folder / Kustomize |
| `kubectl diff -f deploy.yaml` | Declarative | Preview changes before apply |
| `kubectl create deployment web --image=httpd:2.4 --replicas=2` | Imperative | Quick tests |
| `kubectl create ns site-dev` | Imperative | Namespace |
| `kubectl create deployment web --image=httpd --dry-run=client -o yaml > web.yaml` | Generate | Start a manifest |
| `kubectl edit deploy publish` | Imperative | Edit live object in editor (avoid in prod: drifts from Git) |
| `kubectl patch deploy publish -p '{"spec":{"replicas":4}}'` | Imperative | Small change |
| `kubectl label pod p1 tier=web` / `kubectl annotate ...` | Metadata | Labels/annotations |
| `kubectl replace -f file.yaml` | Imperative | Replace whole object |
| `kubectl delete -f deploy.yaml` / `kubectl delete pod p1` | Delete | Remove (a deleted pod of a Deployment is recreated) |
| `kubectl delete pod p1 --grace-period=0 --force` | Delete | Stuck terminating pod (last resort) |

Good practice: manifests live in **Git**; changes go through PR → CI → `kubectl apply` or a GitOps tool (Argo CD, Flux). That is the Kubernetes version of "changes only through Cloud Manager pipelines".

---

## 5. Deployments, rollouts and scaling

```bash
kubectl set image deploy/publish aem-publish=registry/site/aem-publish:2026.11.2
kubectl rollout status deploy/publish          # wait/follow
kubectl rollout history deploy/publish         # revisions
kubectl rollout history deploy/publish --revision=3
kubectl rollout undo deploy/publish            # previous revision
kubectl rollout undo deploy/publish --to-revision=2
kubectl rollout pause deploy/publish / resume  # batch several changes
kubectl rollout restart deploy/publish         # new pods, same spec (e.g. to reload config)
kubectl scale deploy/publish --replicas=5
kubectl autoscale deploy/publish --min=2 --max=8 --cpu-percent=70   # creates HPA
kubectl get hpa -w
```

---

## 6. Logs, exec, copy and port-forward

```bash
kubectl logs publish-7c9d                       # single-container pod
kubectl logs publish-7c9d -c dispatcher          # one container of several
kubectl logs publish-7c9d --previous             # crashed previous instance  ← CrashLoopBackOff
kubectl logs -f publish-7c9d --since=10m --tail=500 --timestamps
kubectl logs -l app=publish --all-containers --prefix   # all pods with a label
kubectl logs deploy/publish                      # one pod of the deployment

kubectl exec -it publish-7c9d -c aem-publish -- sh      # shell
kubectl exec publish-7c9d -- cat /etc/hosts             # one command
kubectl exec publish-7c9d -- jcmd 1 Thread.print        # thread dump of PID 1 (if JDK in image)
kubectl debug -it publish-7c9d --image=busybox --target=aem-publish   # ephemeral debug container

kubectl cp site-prod/publish-7c9d:/tmp/heap.hprof ./heap.hprof -c aem-publish
kubectl port-forward pod/publish-7c9d 4503:4503        # localhost:4503 → pod
kubectl port-forward svc/publish 8080:80                # → service
```
Linux tools from Module 5 (`top`, `ps`, `jcmd`, `curl`) are what you run **inside** `kubectl exec`, if the image contains them.

---

## 7. Config, secrets and storage commands

```bash
kubectl create configmap publish-config --from-literal=ENV=prod --from-file=app.properties
kubectl create secret generic smtp --from-literal=SMTP_PASSWORD='***'
kubectl create secret docker-registry regcred --docker-server=registry.example.com --docker-username=u --docker-password='***'
kubectl create secret tls site-tls --cert=tls.crt --key=tls.key
kubectl get cm publish-config -o yaml
kubectl get secret smtp -o jsonpath='{.data.SMTP_PASSWORD}' | base64 -d   # decode (only with permission)
kubectl get pv,pvc,storageclass
kubectl describe pvc repo-data                  # Bound? which PV?
```
Remember: changing a ConfigMap/Secret consumed as env needs `kubectl rollout restart`.

---

## 8. Networking commands

```bash
kubectl get svc -n site-prod                       # type, cluster IP, ports
kubectl get endpoints publish -n site-prod         # pod IPs behind the service (empty = no ready pods)
kubectl get ingress -n site-prod                   # hosts, paths, address
kubectl describe ingress site                      # rules, TLS secret, backend errors
kubectl get networkpolicy -n site-prod
# test from inside the cluster
kubectl run tmp --rm -it --image=curlimages/curl -- sh
  curl -sI http://publish.site-prod.svc.cluster.local/content/site/en.html
  nslookup publish.site-prod.svc.cluster.local
```
Service DNS: `<service>.<namespace>.svc.cluster.local`.

---

## 9. Nodes, resources and cluster health

```bash
kubectl get nodes -o wide
kubectl describe node worker-2                 # capacity, allocatable, conditions, pods, pressure
kubectl top nodes / kubectl top pods -n site-prod --containers   # needs metrics-server
kubectl cordon worker-2                        # no new pods
kubectl drain worker-2 --ignore-daemonsets --delete-emptydir-data   # move pods off (maintenance)
kubectl uncordon worker-2
kubectl taint nodes worker-2 dedicated=aem:NoSchedule
kubectl get --raw='/readyz?verbose'            # API server health
```

---

## 10. Access control (RBAC)

| Object | Meaning |
|--------|---------|
| ServiceAccount | Identity for pods/CI |
| Role / ClusterRole | Allowed verbs on resources (namespace / cluster-wide) |
| RoleBinding / ClusterRoleBinding | Give a role to a user/group/service account |

```bash
kubectl auth can-i delete pods -n site-prod
kubectl auth can-i '*' '*' --as=system:serviceaccount:ci:deployer -n site-prod
kubectl get roles,rolebindings -n site-prod
kubectl create rolebinding dev-view --clusterrole=view --group=developers -n site-dev
```
AEM link: same idea as AEM groups + ACLs and least privilege (Module 3): give the CI account only deploy rights in its namespace.

---

## 11. YAML manifest anatomy

```yaml
apiVersion: apps/v1          # API group/version of the kind
kind: Deployment             # object type
metadata:
  name: publish
  namespace: site-prod
  labels: { app: publish, tier: delivery }
  annotations: { owner: platform-team }
spec:                        # desired state (status is written by Kubernetes)
  replicas: 3
  selector: { matchLabels: { app: publish } }     # must match template labels
  template:                  # pod template
    metadata: { labels: { app: publish } }
    spec:
      serviceAccountName: publish
      imagePullSecrets: [{ name: regcred }]
      containers:
        - name: aem-publish
          image: registry.example.com/site/aem-publish:2026.11.1
          imagePullPolicy: IfNotPresent
          ports: [{ name: http, containerPort: 4503 }]
          env:
            - name: RUNMODE
              value: publish,prod
            - name: SMTP_PASSWORD
              valueFrom: { secretKeyRef: { name: smtp, key: SMTP_PASSWORD } }
          envFrom: [{ configMapRef: { name: publish-config } }]
          resources:
            requests: { cpu: "2", memory: 8Gi }
            limits: { memory: 8Gi }
          startupProbe:   { httpGet: { path: /system/health, port: http }, failureThreshold: 60, periodSeconds: 10 }
          readinessProbe: { httpGet: { path: /system/health, port: http }, periodSeconds: 10 }
          livenessProbe:  { httpGet: { path: /system/health, port: http }, periodSeconds: 20, failureThreshold: 3 }
          volumeMounts: [{ name: tmp, mountPath: /tmp }]
      volumes: [{ name: tmp, emptyDir: {} }]
```

Other kinds in one glance:
```yaml
# Service
apiVersion: v1
kind: Service
metadata: { name: publish }
spec: { type: ClusterIP, selector: { app: publish }, ports: [{ port: 80, targetPort: 80 }] }
---
# Ingress with TLS
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata: { name: site }
spec:
  tls: [{ hosts: [www.example.com], secretName: site-tls }]
  rules:
    - host: www.example.com
      http: { paths: [{ path: /, pathType: Prefix, backend: { service: { name: publish, port: { number: 80 } } } }] }
---
# HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: publish }
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: publish }
  minReplicas: 2
  maxReplicas: 8
  metrics: [{ type: Resource, resource: { name: cpu, target: { type: Utilization, averageUtilization: 70 } } }]
---
# CronJob (e.g. nightly report)
apiVersion: batch/v1
kind: CronJob
metadata: { name: nightly-report }
spec:
  schedule: "0 2 * * *"
  jobTemplate: { spec: { template: { spec: { restartPolicy: OnFailure, containers: [{ name: job, image: alpine, command: ["sh","-c","echo run"] }] } } } }
---
# NetworkPolicy: only the ingress controller may reach publish
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: publish-from-ingress }
spec:
  podSelector: { matchLabels: { app: publish } }
  ingress: [{ from: [{ namespaceSelector: { matchLabels: { name: ingress } } }] }]
```

---

## 12. Helm

Helm packages manifests as **charts** with templates and values.
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami && helm repo update
helm search repo nginx
helm install web bitnami/nginx -n site-dev --create-namespace -f values-dev.yaml
helm upgrade --install web ./charts/site -f values-prod.yaml --set image.tag=2026.11.1
helm list -n site-dev
helm status web / helm get values web / helm get manifest web
helm history web
helm rollback web 2
helm uninstall web
helm template ./charts/site -f values-prod.yaml   # render locally, no install
helm lint ./charts/site
helm create mychart                               # skeleton chart
```
Chart layout: `Chart.yaml`, `values.yaml`, `templates/*.yaml` with `{{ .Values.image.tag }}`.

---

## 13. Kustomize

Base manifests + per-environment overlays, no templating:
```
k8s/
├── base/        deployment.yaml service.yaml kustomization.yaml
└── overlays/
    ├── dev/     kustomization.yaml  (replicas 1, image tag dev)
    └── prod/    kustomization.yaml  (replicas 3, image tag release)
```
```bash
kubectl kustomize overlays/prod      # render
kubectl apply -k overlays/prod       # apply
```
AEM link: like OSGi run-mode folders (`config.dev`, `config.prod`): one base, environment-specific differences.

---

## 14. Local clusters: kind and minikube

```bash
kind create cluster --name aem-lab
kind load docker-image myorg/dispatcher:1.4.0 --name aem-lab   # use local image without registry
kind delete cluster --name aem-lab

minikube start --memory 8192 --cpus 4
minikube dashboard
minikube service publish -n site-dev     # open service in browser
minikube addons enable ingress metrics-server
minikube stop / minikube delete
```

---

## 15. Debugging recipes

**Pod not starting**
```bash
kubectl get pod <p> -o wide
kubectl describe pod <p> | sed -n '/Events/,$p'     # read the events first
kubectl logs <p> --previous
```
**Site returns 503 through Ingress**
```bash
kubectl get endpoints publish                       # empty? → no ready pods / wrong selector
kubectl get pods -l app=publish                     # READY 0/1?
kubectl describe pod <p> | grep -A5 Readiness
kubectl port-forward pod/<p> 4503:4503 && curl -I localhost:4503/system/health
```
**Suspected OOMKilled**
```bash
kubectl get pod <p> -o jsonpath='{.status.containerStatuses[*].lastState.terminated}'
# reason: OOMKilled, exitCode: 137  → compare limits with -Xmx / MaxRAMPercentage
kubectl top pod <p> --containers
```
**Bad deployment**
```bash
kubectl rollout status deploy/publish   # stuck?
kubectl rollout history deploy/publish
kubectl rollout undo deploy/publish
```
**Which pod served the error?** Correlate the request time from the access log (CDN/Ingress logs) with pod restarts (`kubectl get pods` RESTARTS + `describe` timestamps), exactly like correlating logs in Module 1.

---

## 16. One-page cheat sheet

```text
CONTEXT    kubectl config get-contexts | use-context X | set-context --current --namespace=N
READ       get <kind> [-o wide|yaml] [-l k=v] [-A] | describe <kind> <name> | get events --sort-by=.lastTimestamp
CHANGE     apply -f file|-k dir | diff -f file | delete -f file | edit | patch | label | annotate
DEPLOY     set image | rollout status|history|undo|restart | scale --replicas=N | autoscale
INSPECT    logs [-f] [--previous] [-c C] | exec -it P -- sh | cp | port-forward | debug
CONFIG     create configmap|secret ... | get secret -o jsonpath | rollout restart to apply
NETWORK    get svc,endpoints,ingress | run tmp --rm -it --image=curlimages/curl -- sh
NODES      get nodes | describe node | top nodes/pods | cordon | drain | uncordon
RBAC       auth can-i VERB RESOURCE [--as=...]
HELP       explain <kind.field> | api-resources
DOCKER     build -t | run -d -p -e -v | ps | logs -f | exec -it | inspect | stats | push
HELM       install|upgrade --install|rollback|uninstall|template|lint
STATUS     Pending→scheduling · ImagePullBackOff→image/creds · CrashLoopBackOff→logs --previous
           OOMKilled(137)→memory limit · 0/1 Ready→readiness · empty endpoints→selector/readiness
```
