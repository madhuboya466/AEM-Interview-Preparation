# AEM Interview Preparation: Adobe AEM Technical Support Engineer (TSE)

A subject-based study set. Each module teaches one technical subject end to end, so any interview question on it can be answered from understanding, not memory. None of it is written as question-and-answer.

## How the material is organised

| # | Module | Subject | File |
|---|--------|---------|------|
| 1 | Production Support and AEM Troubleshooting | Incidents, layers, HTTP errors, logs, Dispatcher/CDN, RCA, your case studies | [01-production-support/production-support.md](01-production-support/production-support.md) |
| 2 | AEM DevOps and Deployment Lifecycle | Git, Jenkins, Maven, Cloud Manager, environments, releases | [02-devops-deployment/devops-deployment.md](02-devops-deployment/devops-deployment.md) |
| 3 | AEM Administration and Platform Architecture | AEMaaCS, users/ACLs, content, replication, repository, Dispatcher | [03-aem-administration/aem-administration.md](03-aem-administration/aem-administration.md) |
| 4 | AEM Internals: Java, Sling, OSGi and JCR | How the frameworks work together inside AEM | [04-aem-internals/java-sling-osgi-jcr.md](04-aem-internals/java-sling-osgi-jcr.md) |
| 5 | Java and JVM Performance Troubleshooting | Memory, GC, heap and thread dumps, Linux, AEM performance | [05-java-jvm-performance/jvm-troubleshooting.md](05-java-jvm-performance/jvm-troubleshooting.md) |
| 6 | Adobe TSE: Case Management and Customer Success | Support lifecycle, SLA, escalation, communication, knowledge | [06-adobe-tse-support/support-and-sla.md](06-adobe-tse-support/support-and-sla.md) |
| – | Diagrams | Mermaid diagrams + an HTML route map of AEMaaCS | [diagrams/architecture-and-workflows/](diagrams/architecture-and-workflows/) |
| – | Quick revision | One-page recap of all six modules | [revision/quick-revision.md](revision/quick-revision.md) |

Suggested order: 3 → 4 → 1 → 2 → 5 → 6. Administration and internals give the vocabulary; production support and DevOps use it; JVM and the support process are the TSE-specific layers on top.

## Experience labels used in every module

| Label | Meaning |
|-------|---------|
| **[Hands-on]** | Taken from your own task log or the experience you described. Speak about it as your work. |
| **[Adjacent]** | You touched the area; the deeper theory is new. Speak about the part you did, then the concept. |
| **[Theory]** | Not in your record. Explain it as knowledge you have studied, not as something you did. |
| **`[fill in]`** | A detail only you know (exact config, branch name, date). It was left blank deliberately rather than invented. Complete it before the interview. |

## Source of your experience (summary)

Your work on an enterprise multi-site AEM platform, on AEM as a Cloud Service, as recorded in your task log:

1. List component: Teaser delegate property, List v3 → v4.
2. Preview-to-publish workflow so business users can publish to the Preview tier without approval.
3. Core form container for one country site: confirmation email to the user, submissions to business users by BCC.
4. Namespace issues across RDE, QA, STAGE, PROD fixed with Repoinit scripts during on-prem → cloud migration.
5. SMTP token expired after 90 days of inactivity: new token from the service account, configuration updated, monthly refresh agreed.
6. Users and groups for country authors and approvers for all the platform's sites.
7. Cloud preview links not navigating correctly: Dispatcher rule appending `.html`.
8. Pipelines for RDE, QA, STAGE and Adobe pipelines.
9. SSL certificates in Cloud Manager, IP allowlisting with CIDR ranges.
10. Network team DNS records to the Adobe CDN; domain mapping in Cloud Manager.
11. Dispatcher configuration for domain mapping across RDE/QA/STAGE/PROD.
12. Content migration with the Content Transfer Tool and migration sets in Cloud Acceleration Manager.
13. Production issue: content ingestion broke workflows and Manage Publication (`cq:Workflow` content violation); empty rows in the path CSV wiped workflow definitions; reverted, re-ingested, peer review introduced.
14. Production author 403 for authors in one region: users fine, IP outside allowlist; CIDR added.
15. Live-site images 404: uppercase `.JPG` not allowed by Dispatcher filter; same for `.xlsx`.
16. 503 for a regional team after publishing: excessive containers; page re-authored; author guidance.
17. Dev pipelines triggering several times: commit arriving in the destination repository treated as a new commit.

Plus your wider background: AEM developer, Jenkins, Git, Maven, release management, user administration, Java, Linux.

Interactive overview: open [diagrams/architecture-and-workflows/aem-route-map.html](diagrams/architecture-and-workflows/aem-route-map.html) in a browser.
