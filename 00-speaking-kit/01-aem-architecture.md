# 01 · AEM architecture and request flow

4 modules · Day 1–2 · Long version: [03-aem-administration](../03-aem-administration/aem-administration.md)

---

## M1 · What AEM is: Author and Publish

**In simple words**
AEM is a content management system. Authors create pages on one server (Author). Visitors see them on another (Publish).

**Say this**
> Adobe Experience Manager is an enterprise content management system for websites and digital assets. It has two main tiers. Author is where content authors create, edit and approve pages and assets. Publish is where the approved content is delivered to website visitors. When an author publishes a page, AEM copies it from Author to Publish. In front of Publish we have the Dispatcher for caching and security, and a CDN for fast global delivery.

**My example** [Hands-on]
> "In my project we run many country sites on AEM as a Cloud Service. Authors and approvers work on Author, and the live sites are served from Publish through the Dispatcher and Adobe's CDN."

**Keywords:** CMS · Author = create · Publish = deliver · publish copies · Dispatcher · CDN

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is AEM? | Sentence 1, then all |
| What is the difference between Author and Publish? | Sentences 2–4 |
| Explain AEM architecture. | All, then go to M2 (the flow) |
| Why do we need separate Author and Publish? | "Author is for creating and approving, Publish is for delivery. Keeping them separate protects drafts and lets Publish scale for visitors." |

**Practice:** What is AEM? · Why are Author and Publish separate? · Where does a visitor's request go?

---

## M2 · The request flow, end to end

**In simple words**
A visitor's request passes through a chain of stops. Each stop can answer on its own.

**The flow, step by step**
```
Browser → DNS → CDN → Dispatcher → Publish → Repository (JCR)
```
1. **Start: the browser** asks for a page, for example `www.site.com/en.html`.
2. **DNS** turns the domain into an address. In AEM as a Cloud Service it points to Adobe's CDN.
3. **The CDN receives the request first.** It handles SSL, IP rules and caching. If it has a cached copy, it returns it and the flow ends here.
4. **If not cached, the request goes to the Dispatcher.** Dispatcher checks its filter rules (allow or block), then its own cache. If cached, it returns the page.
5. **If not cached, the request reaches Publish.** AEM (Sling) finds the page in the repository and renders the HTML.
6. **The repository (JCR)** holds the content: pages, properties, assets.
7. **End: the response goes back** the same way. Dispatcher and the CDN store a copy so the next visitor gets it faster.

**Diagram script** (say while pointing, left to right)
> "The request starts in the browser. DNS sends it to the CDN. The CDN serves it from cache if it can, otherwise it forwards it to the Dispatcher. The Dispatcher checks the filter rules and its cache, and only on a miss calls Publish. Publish renders the page from the content repository. The response goes back through Dispatcher and the CDN, which cache it for the next visitor."

**Say this** (short version)
> A request starts in the browser and DNS sends it to the CDN. The CDN returns a cached copy if it has one; if not, it forwards the request to the Dispatcher. The Dispatcher checks its filter rules and its cache, and only on a cache miss sends the request to Publish. Publish renders the page using Sling and the content in the JCR repository. On the way back, the Dispatcher and the CDN cache the response, so most visitors never reach Publish.

**My example** [Hands-on]
> "When images were returning 404 on our live site, I followed this chain: the image existed on Author and on Publish, so the problem had to be in front of Publish, in the Dispatcher filter." (Story 2)

**Keywords:** Browser · DNS · CDN (cache) · Dispatcher (filter + cache) · Publish (render) · JCR · cached on return

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What happens when a user opens an AEM page? | "A request starts in the browser…" |
| Explain the delivery architecture. | Same |
| Where is caching done in AEM? | "Caching happens at two places: the CDN and the Dispatcher…" then the flow |
| How do you troubleshoot a page that doesn't load? | "I follow the request path…", then go to 03-M1 |
| What is the role of the CDN / Dispatcher? | Their step, then the full flow briefly |

**Practice:** Draw and explain the request flow · Where can a request be answered without reaching Publish? · What does Publish do?

---

## M3 · AEM as a Cloud Service vs AEM 6.5

**In simple words**
6.5 runs on servers your team manages. The cloud version is run by Adobe as a service, in containers, always up to date.

**Say this**
> AEM 6.5 runs on servers that the customer or Adobe Managed Services maintains, with upgrades done as projects. AEM as a Cloud Service is run by Adobe on containers that scale automatically and receive updates continuously. In the cloud, code is part of an immutable image, so `/apps` and `/libs` cannot be changed at runtime, and the only way to deploy is through Cloud Manager pipelines. Replication agents are replaced by Sling Content Distribution, and the Dispatcher cache is invalidated automatically. Users log in through Adobe IMS, and there is a Preview tier for reviewing content before it goes live.

**Comparison to remember (5 points)**
| | 6.5 | Cloud Service |
|---|---|---|
| Who runs it | Customer / AMS | Adobe |
| Updates | Service packs | Continuous |
| Deploy | Package Manager, Jenkins | Cloud Manager only |
| Replication | Agents and queues | Content Distribution |
| Login | Local, LDAP, SAML | Adobe IMS |

**My example** [Hands-on]
> "My recent work is on AEM as a Cloud Service: we migrated content from on-premise, and I fixed migration issues like namespaces with Repoinit, because in the cloud you can't change the repository structure by hand." (Story 6)

**Keywords:** Adobe-managed · containers + autoscale · continuous updates · immutable `/apps` · Cloud Manager only · Distribution · IMS · Preview

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is the difference between AEM 6.5 and AEMaaCS? | All |
| Why can't I install a package in production through Package Manager? | Sentence 3 |
| How does replication work in the cloud? | Sentence 4, then 05-M1 |
| What are the benefits of the cloud version? | Sentences 2, 3, 5 as benefits |

**Practice:** Compare 6.5 and Cloud Service · Why is `/apps` read-only in the cloud? · What is the Preview tier?

---

## M4 · Cloud Manager and environments

**In simple words**
Cloud Manager is the control panel for an AEM cloud program: pipelines, environments, domains, certificates, IP allowlists, logs.

**Say this**
> Cloud Manager is Adobe's self-service portal for AEM as a Cloud Service. A program has environments: development, stage, production, and Rapid Development Environments for quick developer testing. Cloud Manager runs the CI/CD pipelines that build, test and deploy code. It is also where we manage custom domains, SSL certificates, IP allowlists, environment variables and log downloads. So most platform changes, apart from content, go through Cloud Manager.

**My example** [Hands-on]
> "In my project I used Cloud Manager for SSL certificates, domain mapping, IP allowlisting with CIDR ranges, and pipelines for RDE, QA and Stage." (Stories 8, 1, 15)

**Keywords:** self-service portal · environments (dev, stage, prod, RDE) · pipelines · domains + SSL · IP allowlist · env variables · logs

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is Cloud Manager? | All |
| How do you deploy code in AEMaaCS? | Sentence 3, then 06-M4 |
| Where do you configure SSL or domains? | Sentence 4 + Story 8 |
| How do you get AEM logs in the cloud? | "From Cloud Manager: download per environment and service, or tail them with the Adobe I/O CLI." |

**Practice:** What can you do in Cloud Manager? · What is an RDE? · How do you get production logs?
