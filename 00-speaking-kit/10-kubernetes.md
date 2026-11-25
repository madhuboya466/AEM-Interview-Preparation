# 10 · Kubernetes

5 modules · Day 18–19 · Long version: [07-kubernetes](../07-kubernetes/kubernetes-for-aem.md) · Commands: [07-kubernetes/commands-reference](../07-kubernetes/commands-reference.md)

> **[Theory] subject.** Bridge sentence:
> "AEM as a Cloud Service runs on Kubernetes, but Adobe operates it. I've worked with its effects, like rolling deployments, immutable code and environment variables, and I've studied how Kubernetes works underneath."

---

## M1 · What Kubernetes is and how it is built

**In simple words**
Kubernetes runs containers on many machines and keeps them in the state you asked for.

**The flow, step by step**
1. You **declare** the desired state (e.g. "3 copies of this image") to the **API server**.
2. It is stored in **etcd**.
3. The **scheduler** picks a node for each pod.
4. The **kubelet** on that node starts the containers.
5. **Controllers** keep comparing desired vs actual and fix differences (restart, replace).

**Diagram script**
> "I send the desired state to the API server, which stores it in etcd. The scheduler chooses a node, the kubelet on that node starts the containers, and controllers keep checking that what's running matches what I asked for. If a pod dies, they replace it."

**Say this**
> Kubernetes is a platform that runs containers across a cluster of machines. You declare the desired state, for example three copies of an application, and send it to the API server, which stores it in etcd. The scheduler places each pod on a node, and the kubelet on that node starts the containers. Controllers keep comparing the desired state with the actual state and fix any difference, for example by restarting a failed pod. That is why Kubernetes is self-healing and can scale easily.

**Keywords:** runs containers on a cluster · desired state · API server + etcd · scheduler · kubelet · controllers = self-healing

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is Kubernetes? | Sentences 1, 5 |
| Explain Kubernetes architecture. | All |
| What is a container? | "A packaged application with everything it needs, started from an immutable image." |

**Practice:** Draw the control plane and node · What does "desired state" mean?

---

## M2 · Core objects

**In simple words**
A pod runs containers. A Deployment keeps copies running and updates them. A Service gives them one address. Ingress brings outside traffic in.

**The flow** (outside → in)
```
Ingress → Service → Pods  (managed by a Deployment)
ConfigMap / Secret → injected into pods
```

**Say this**
> The smallest unit is a pod, which runs one or more containers that share a network address. A Deployment manages identical pods: it keeps the right number running and performs rolling updates. A Service gives these pods one stable name and balances traffic across them, and an Ingress routes outside traffic by domain and path and handles TLS. Configuration comes from ConfigMaps, and passwords and tokens from Secrets. Namespaces separate environments or teams.

**AEM link** [Adjacent]
> "In AEM as a Cloud Service terms: a publish instance with its Dispatcher is like a pod, the publish farm is like a Deployment, domain mapping and SSL are like Ingress, and Cloud Manager environment variables and secrets are like ConfigMaps and Secrets."

**Keywords:** pod · Deployment (count + rolling) · Service (stable name) · Ingress (domain, TLS) · ConfigMap · Secret · Namespace

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| Pod vs Deployment? | Sentences 1–2 |
| What is a Service / Ingress? | Sentence 3 |
| How do you pass config and secrets? | Sentence 4 + AEM link |

**Practice:** Explain the path Ingress → Service → Pod · Map Kubernetes objects to AEMaaCS.

---

## M3 · Probes and resource limits

**In simple words**
Probes tell Kubernetes if a pod is ready or alive. Limits stop a pod from using too much CPU or memory.

**Say this**
> Kubernetes uses probes to check pods. The readiness probe decides if a pod can receive traffic, and if it fails, the pod is taken out of the Service but not restarted. The liveness probe decides if a pod is still alive, and if it fails, the container is restarted; slow-starting apps like AEM also need a startup probe. Each container has resource requests, used for scheduling, and limits: above the CPU limit it is slowed down, and above the memory limit it is killed with "OOMKilled", exit code 137. For Java, the heap plus native memory must fit inside the container's memory limit.

**Keywords:** readiness = traffic · liveness = restart · startup = slow apps · requests vs limits · CPU throttled · memory OOMKilled 137 · heap + native < limit

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| Readiness vs liveness? | Sentences 2–3 |
| What is OOMKilled? | Sentence 4 |
| Java OOM vs OOMKilled? | "Java OOM is the heap full inside the JVM; OOMKilled is the whole container above its memory limit." |

**Practice:** What happens when readiness fails? · Why does a Java app get OOMKilled?

---

## M4 · Rolling deployments, and AEMaaCS through a Kubernetes lens

**In simple words**
New pods start, pass checks, get traffic; old pods stop. That is how AEMaaCS deploys with no downtime.

**The flow, step by step**
1. A new image version is set on the Deployment.
2. **One new pod** starts.
3. Its **readiness probe** passes → it gets traffic.
4. **One old pod** stops.
5. Repeat until all pods are new.
6. If new pods never become ready, the rollout stops and **old pods keep serving**.

**Diagram script**
> "Kubernetes starts one new pod, waits until it's ready, sends it traffic, and stops one old pod. It repeats this until all pods are new. If the new version never becomes ready, it stops and the old version keeps running, so users don't see downtime."

**Say this**
> A rolling deployment replaces pods gradually: a new pod starts, passes its readiness check, receives traffic, and then an old pod is stopped. If the new version never becomes ready, the rollout stops and the old pods continue to serve. AEM as a Cloud Service works on the same idea: Cloud Manager builds an image with our code and the AEM release, and replaces the running instances without downtime. Because code is inside the image, `/apps` can't be changed at runtime, and changing an environment variable restarts the service. Publish also scales by adding more instances under load.

**My example** [Adjacent]
> "This explains what I saw in my project: production deployments through Cloud Manager with no downtime, repository setup like namespaces done through Repoinit in code, and configuration through environment variables." (Stories 6, 15)

**Keywords:** new pod → ready → traffic → old pod stops · failed = old keeps serving · AEMaaCS image = code + release · immutable /apps · env change restarts · scale out

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How does a rolling update work? | Sentences 1–2 |
| How is Kubernetes used in AEMaaCS? | Sentences 3–5 |
| Why does an environment variable change restart AEM? | Sentence 4 |
| How do you roll back? | "In Kubernetes, `kubectl rollout undo`; in AEMaaCS, redeploy the previous commit through the pipeline." |

**Practice:** Explain zero-downtime deployment · Connect three AEMaaCS behaviours to Kubernetes.

---

## M5 · Debugging pods

**In simple words**
Check the pod status, read the events, read the logs.

**The flow, step by step**
1. `kubectl get pods` → status and restarts.
2. `kubectl describe pod` → **events** (scheduling, image pull, probes, OOMKilled).
3. `kubectl logs` (and `--previous` for a crashed container).
4. `kubectl get endpoints` → does the Service have ready pods?
5. Fix and watch the rollout.

**Say this**
> To debug a problem in Kubernetes, I start with `kubectl get pods` to see the status and restart count. Then `kubectl describe pod` shows the events, such as scheduling problems, image pull errors, failed probes or OOMKilled. I read the logs with `kubectl logs`, and use `--previous` for a container that crashed. If the Service returns 503, I check `kubectl get endpoints`, because no endpoints means no pod is ready. The common statuses are Pending, ImagePullBackOff, CrashLoopBackOff and OOMKilled.

**Status → meaning**
| Status | Meaning |
|--------|---------|
| Pending | Can't be scheduled (resources) |
| ImagePullBackOff | Image or registry credentials wrong |
| CrashLoopBackOff | App starts and crashes → `logs --previous` |
| OOMKilled | Memory limit |
| 0/1 Ready | Readiness probe failing |

**Keywords:** get pods · describe → events · logs (--previous) · endpoints · Pending · ImagePullBackOff · CrashLoopBackOff · OOMKilled

**Practice:** A pod is in CrashLoopBackOff: what do you do? · Service returns 503: what do you check?
