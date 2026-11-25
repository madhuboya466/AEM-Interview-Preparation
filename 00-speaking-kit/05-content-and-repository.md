# 05 · Publishing, workflows, repository and migration

4 modules · Day 8–9 · Long version: [03-aem-administration §4–5](../03-aem-administration/aem-administration.md)

---

## M1 · Publishing (replication)

**In simple words**
Publishing copies content from Author to Publish. In the cloud, Adobe's distribution service does it.

**The flow, step by step** (cloud)
1. Author clicks **Publish** (Quick Publish, Manage Publication or a workflow step).
2. AEM **packages** the content.
3. **Sling Content Distribution** sends it through Adobe's pipeline service.
4. **Every Publish instance** imports it (or the **Preview** tier, if that was the target).
5. The **Dispatcher cache is invalidated** automatically.
6. The CDN refreshes when its TTL expires.

**Diagram script**
> "The author publishes. AEM packages the content and the distribution service sends it to all publish instances. Each one imports it, the Dispatcher cache is cleared, and visitors get the new content; the CDN updates after its cache time."

**Say this**
> Publishing, also called replication or activation, copies content from Author to Publish. In AEM 6.5 this was done by replication agents with queues. In AEM as a Cloud Service it uses Sling Content Distribution through an Adobe pipeline service, and every Publish instance receives the content. Content can also be published to the Preview tier for review first. After publishing, the Dispatcher cache is invalidated automatically. If publishing fails, I check the distribution queue on Author and the error log.

**My example** [Hands-on] (Story 13)
> "I created a workflow so business users can publish content to Preview for review without the normal approval."

**Keywords:** Author → Publish · 6.5 = agents + queues · cloud = Content Distribution · Preview tier · auto invalidation · queue + error.log

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How does replication work? | All |
| Content isn't showing on Publish after publishing. | "First I check the distribution queue on Author for a blocked item, then the error log. If it is on Publish but not on the site, it's a cache issue." |
| What is a replication agent? | Sentence 2: the 6.5 way, with transport URL, user and queue |
| What is the Preview tier? | Sentence 4 + Story 13 |

**Practice:** Explain publishing in 6.5 vs cloud · Publishing is stuck: what do you check?

---

## M2 · Workflows

**In simple words**
A workflow automates a process, like approval before publishing.

**The flow, step by step**
1. A **model** defines the steps (designed in `/conf`, synced to `/var` to run).
2. Started manually or by a **launcher** (automatic, on content events).
3. A running copy is an **instance**, acting on a **payload** (a page or asset).
4. **Steps**: participant (a person approves) or process (Java code).
5. Ends when the last step completes (for example, publish).

**Say this**
> A workflow automates a business process in AEM, for example review and approval before publishing. A workflow model defines the steps, and it must be synced so the runtime version is updated. A workflow can be started manually or automatically by a launcher when content changes. Each running workflow is an instance that works on a payload, usually a page or an asset. Steps can be participant steps, where a person approves, or process steps, which run Java code.

**My example** [Hands-on] (Stories 13 and 3)
> "I built a preview-to-publish workflow. I also fixed a production issue where workflows stopped working because a content ingestion had wiped the workflow definitions."

**Keywords:** automate process · model (sync) · launcher · instance + payload · participant / process steps

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is an AEM workflow? | All |
| A workflow is stuck. | "I check the instance history and the error log for the failing step, then retry it from the failures view." |
| What is a launcher? | Sentence 3 |
| Tell me about a workflow issue. | Story 3 |

**Practice:** Explain model, launcher, instance, payload · Workflow didn't start: why?

---

## M3 · Repository structure and Repoinit

**In simple words**
All content is a tree of nodes. Each top folder has a purpose. In the cloud, structure is created by code (Repoinit).

**Say this**
> AEM stores everything in the JCR, a tree of nodes and properties. `/apps` holds our code and `/libs` holds Adobe's code, and both are read-only at runtime in the cloud. `/conf` holds templates, policies and workflow models; `/content` holds pages and assets; `/var` holds runtime data like workflow instances; and `/home` holds users and groups. Because we can't change the repository by hand in the cloud, we use Repoinit scripts to create service users, paths, permissions and namespaces. They are deployed through the pipeline, so every environment gets the same setup.

**My example** [Hands-on] (Story 6)
> "During migration we had namespace errors in every environment. I fixed them with a Repoinit script that registers the namespace, deployed through the pipelines."

**Keywords:** JCR tree · /apps + /libs code (read-only) · /conf · /content · /var · /home · Repoinit = structure as code

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| Explain the AEM repository structure. | Sentences 1–3 |
| What is Repoinit? | Sentences 4–5 + Story 6 |
| Where are workflow models / templates stored? | "/conf" sentence |
| Can you edit CRXDE in production? | "Not in the cloud: CRXDE is only on dev. Changes go through code or content tools." |

**Practice:** Name the main repository folders · Why use Repoinit?

---

## M4 · Content migration to the cloud

**In simple words**
Check the old system, extract content, then ingest it into the cloud, then top up the changes.

**The flow, step by step**
1. **Best Practices Analyzer** on the old AEM: what must change.
2. Create a **migration set** in **Cloud Acceleration Manager**.
3. **Extraction** with the **Content Transfer Tool** on the source.
4. **Ingestion** into the cloud environment (wipe or merge).
5. **Top-up** runs for content changed since.
6. Verify, then go live.

**Diagram script**
> "First we analyse the source system. Then we create a migration set, extract the content with the Content Transfer Tool, and ingest it into the cloud. Later, top-up runs bring the latest changes. Finally we verify everything and go live."

**Say this**
> To migrate content from on-premise AEM to AEM as a Cloud Service, we first run the Best Practices Analyzer to see what needs to change. Then we create a migration set in Cloud Acceleration Manager and use the Content Transfer Tool to extract the content from the source. Next we ingest it into the target cloud environment; a wipe ingestion clears existing content first, so the path list must be correct. After that, top-up runs move content that changed in the meantime. Code is migrated separately through Git and Cloud Manager pipelines.

**My example** [Hands-on] (Stories 9, 6, 3)
> "I did this migration using the Content Transfer Tool and migration sets in Cloud Acceleration Manager. I fixed namespace issues with Repoinit, and recovered from an ingestion where empty rows in the path CSV wiped workflow definitions."

**Keywords:** BPA · migration set (CAM) · CTT extraction · ingestion (wipe!) · top-up · code via pipeline

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you migrate to AEMaaCS? | All |
| What problems did you face in migration? | Stories 6 and 3 |
| What is wipe vs non-wipe ingestion? | "Wipe clears the target first; non-wipe adds on top. Wipe is risky if the path list is wrong." |

**Practice:** List the migration steps · Tell me a migration problem you solved.
