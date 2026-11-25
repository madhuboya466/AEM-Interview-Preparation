# My project stories

Your real experience, turned into short stories you can say in about 45 seconds.
Every fact here comes from your own task log. Where a detail is missing, it is left out on purpose: don't add details you can't support.

## The story pattern: 5 sentences

| # | Slot | Question it answers |
|---|------|--------------------|
| 1 | **Situation** | What was the problem and who was affected? |
| 2 | **Check** | What did I check first? |
| 3 | **Cause** | What did I find? |
| 4 | **Fix** | What did I change, and how did I confirm it? |
| 5 | **Prevent** | What did we do so it doesn't happen again? |

Memory hook: **"Some Calm Checks Find Problems"** (Situation, Check, Cause, Fix, Prevent).

Use these stories in two ways:
- As **slot 4 ("My example")** inside a technical answer.
- As the full answer to "Tell me about a production issue you solved" or "Describe a difficult incident".

---

## Story 1: Production author 403 for one region's authors · [Hands-on]
**Use for:** 403 errors, IP allowlisting, CDN, access troubleshooting, isolating the layer.

> Authors from one region were getting a 403 error on the production author, while other authors were fine. I first checked their access in AEM, their groups and permissions, and confirmed the users were migrated to the cloud; all of that was correct. Then I checked the logs and found their requests were coming from an IP address that was not in the Cloud Manager IP allowlist. I got the correct CIDR range for their network, added it to the allowlist, and asked the users to test, and access worked. The lesson was to always check both layers, AEM permissions and network access, when only some users are affected.

**Keywords:** one region · 403 · permissions fine · logs → IP · CIDR → allowlist · verified

---

## Story 2: Images and files returning 404 on the live site · [Hands-on]
**Use for:** 404 errors, Dispatcher filters, "content is on publish but not on the site".

> Some images were not loading on the live site and returned 404. I checked that the image was present in author and on publish, and it was, so the problem had to be in front of publish, in the Dispatcher. I found the images used an uppercase `.JPG` extension, and the Dispatcher filter only allowed lowercase extensions, so Dispatcher blocked them and returned 404. I allowed the extension in the filter rules and deployed the change, and the images loaded; later I fixed the same issue for an `.xlsx` file. This taught me to prove where the content exists at each layer before changing anything.

**Keywords:** images 404 · author yes · publish yes · Dispatcher · `.JPG` case-sensitive · filter updated

---

## Story 3: Content ingestion wiped workflow definitions · [Hands-on]
**Use for:** migration issues, workflows, root cause analysis, prevention, change control.

> After a content ingestion during our migration, workflows and Manage Publication stopped working. I checked the logs and found a `cq:Workflow` content violation exception, then went through the ingestion logs. The CSV file with the migration paths had empty rows, and that caused the ingestion to wipe out the workflow definitions. We reverted to the last good version, ran the ingestion again with a corrected file, and verified that workflows and publishing worked. To prevent it, I informed the team and we started a peer review of the CSV file before every ingestion.

**Keywords:** ingestion · workflows broken · `cq:Workflow` exception · CSV empty rows · revert + re-ingest · peer review

---

## Story 4: 503 after publishing a heavy page · [Hands-on]
**Use for:** 503 errors, performance, slow pages, content-caused incidents.

> A regional team reported a 503 error on the live site after publishing a page. I checked the logs, confirmed the page was published, and checked it on publish, where it was not loading. The page had too many containers, so rendering made too many calls and the request was too slow. I removed the unnecessary containers, re-authored the page with proper styling and published it again, and it worked. Then I advised authors to use the minimum number of containers on a page.

**Keywords:** 503 after publish · published yes · not loading on publish · too many containers · re-authored · author guidance

---

## Story 5: Emails stopped because a token expired · [Hands-on]
**Use for:** OSGi configuration, integrations, credentials and secrets, proactive prevention.

> Emails from AEM stopped being delivered. I found that the SMTP authentication token had expired after 90 days of inactivity. I logged in with the service account, generated a new token and updated it in the configuration, and emails started working again. To stop it happening again, I asked the team to refresh the token every month.

**Keywords:** emails stopped · SMTP token · 90 days inactivity · new token · config updated · monthly refresh

---

## Story 6: Namespace errors during migration, fixed with Repoinit · [Hands-on]
**Use for:** repository, JCR, Repoinit, migration, "configuration as code".

> While migrating content from on-premise to the cloud, we had namespace issues across RDE, QA, Stage and Production. The migrated content used a custom namespace that did not exist in the cloud repository. In AEM as a Cloud Service you cannot create these manually, so I added a Repoinit script that registers the namespace and deployed it through the pipelines to every environment. After that the content migrated correctly, and every environment had the same repository setup.

**Keywords:** migration · namespace missing · Repoinit · pipeline · all environments

---

## Story 7: Preview links not opening the right page · [Hands-on]
**Use for:** Dispatcher rewrites, Sling extensions, redirects and links.

> On the cloud preview environment, links on a page were not navigating to the correct page. The links did not include the `.html` extension, which AEM needs to render the page. I created a Dispatcher rewrite rule that adds `.html` for the site's content paths. After deploying it, the links worked.

**Keywords:** preview · links wrong · missing `.html` · rewrite rule · deployed

---

## Story 8: Going live with domains, SSL and DNS · [Hands-on]
**Use for:** CDN, custom domains, SSL, DNS, go-live, environments.

> For our sites I handled the domain setup in AEM as a Cloud Service. I added and updated the SSL certificates in Cloud Manager, and worked with the network team to point the internal and external DNS records to Adobe's CDN. Then I configured the domain mapping in Cloud Manager and the Dispatcher configuration, so each domain reached the right site content across RDE, QA, Stage and Production.

**Keywords:** SSL in Cloud Manager · DNS to Adobe CDN · domain mapping · Dispatcher vhosts · all environments

---

## Story 9: Content migration to the cloud · [Hands-on]
**Use for:** migration, Content Transfer Tool, Cloud Acceleration Manager.

> I migrated content from on-premise AEM to AEM as a Cloud Service. I used the Content Transfer Tool and created migration sets in Cloud Acceleration Manager, then ran extraction and ingestion. During this work I also fixed namespace issues with Repoinit and recovered from an ingestion problem caused by a bad path file.

**Keywords:** on-prem → cloud · Content Transfer Tool · migration set in CAM · extraction → ingestion · issues fixed

---

## Story 10: Pipelines running several times for one change · [Hands-on]
**Use for:** CI/CD, Git, Jenkins triggers, pipeline troubleshooting.

> Our dev pipelines were being triggered multiple times for a single change. I found that when the code was synced into the destination repository, that repository treated the synced commit as a new commit and started the pipeline automatically. I updated the dev pipeline setup so one change triggers only one run.

**Keywords:** duplicate runs · sync to destination repo · new commit → auto trigger · pipeline updated

> Before the interview, add one sentence on exactly what you changed (for example, the trigger setting). Keep it true.

---

## Story 11: Users and groups for country authors and approvers · [Hands-on]
**Use for:** user administration, permissions, least privilege, workflows.

> I created users and groups for country-specific authors and approvers across our sites. Users get access to the environment through Adobe's Admin Console, and inside AEM I used groups per country and role, with permissions on that country's content. Approvers are part of the approval workflow, so authors can create content and approvers control what is published.

**Keywords:** country authors + approvers · Admin Console access · AEM groups per country and role · permissions on content · approval workflow

> Check which parts you did in Admin Console and which in AEM, and adjust the sentence to match.

---

## Shorter stories (use as one-line examples)

| # | Story | One line to say | Use for |
|---|-------|----------------|---------|
| 12 | List component | "I fixed the List component's 'Display as Teaser' option by updating the teaser delegate property and upgrading the List component from version 3 to version 4." | Core Components, Sling Models, development |
| 13 | Preview workflow | "I created a workflow so business users can publish content to the preview environment for review without the normal approval." | Workflows, Preview tier |
| 14 | Country site form | "I built a form with the core form container where users get a confirmation email and business users receive submissions by BCC." | Components, email, forms |
| 15 | Pipelines | "I created deployment pipelines for RDE, QA and Stage and worked with Adobe's Cloud Manager pipelines." | DevOps |

---

## Which story for which question

| Interview topic | Best stories |
|-----------------|-------------|
| 403 / access issue | 1, 11 |
| 404 / Dispatcher | 2, 7 |
| 5xx / performance | 4 |
| Root cause analysis, prevention | 3, 1, 5 |
| Migration | 9, 6, 3 |
| Deployment, CI/CD | 10, 15, 6 |
| CDN, domains, SSL | 8, 1 |
| Workflows | 13, 3 |
| Configuration / OSGi / secrets | 5 |
| "Most challenging incident" | 3 (strong RCA + prevention) |
