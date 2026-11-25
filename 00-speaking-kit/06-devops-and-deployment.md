# 06 · DevOps and deployment

5 modules · Day 10–11 · Long version: [02-devops-deployment](../02-devops-deployment/devops-deployment.md)

---

## M1 · Git and the branching strategy

**In simple words**
Developers work on separate branches, merge through reviews, and a release branch goes to production.

**The flow, step by step**
1. Create a **feature branch** from `develop`.
2. Commit and push; open a **pull request**; get it **reviewed**.
3. Merge into `develop` → deploy to **Dev/QA**.
4. Cut a **release branch** → deploy to **Stage** → test.
5. Release to **Production**; merge into `main` and tag it.
6. Urgent fix: **hotfix branch** from `main`, then merge back to `develop`.

**Say this**
> We use Git with a branching strategy. Developers create a feature branch from develop, push their changes and open a pull request, which is reviewed before merging. Develop is deployed to the dev or QA environment for testing. For a release, we create a release branch, which is deployed to Stage and then to Production after approval. Urgent production fixes go on a hotfix branch from main, and are merged back so nothing is lost.

**My example** [Hands-on]
> "In my project we worked with release branches and promoted changes from QA to Stage and then Production." *(Add your real branch names here.)*

**Keywords:** feature → PR + review → develop (QA) → release (Stage → Prod) → main + tag · hotfix

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What branching strategy do you use? | All |
| How does code move to production? | Sentences 3–4, then M4 |
| What is a merge conflict and how do you fix it? | "Two branches change the same lines. I edit the file to keep the right content, commit, then rebuild and retest." |

**Practice:** Explain your branching strategy · How do you handle a hotfix?

---

## M2 · Jenkins pipelines and triggers

**In simple words**
Jenkins builds and tests code automatically when it changes. A Jenkinsfile describes the steps.

**The flow, step by step**
1. A **trigger** starts the job: webhook, SCM polling, timer or manual.
2. The **controller** gives the job to an **agent**.
3. The agent runs the **Jenkinsfile stages**: checkout → build/test (Maven) → package → sync/deploy.
4. **Credentials** come from the Jenkins credentials store.
5. Results and **console logs** show success or failure.

**Say this**
> Jenkins is a CI/CD server. The controller schedules jobs, and agents run the builds. A pipeline is defined in a Jenkinsfile with stages such as checkout, Maven build and test, and deploy or sync. A job can be triggered by a webhook from Git, by SCM polling on a schedule, by a timer or manually. Passwords and tokens are kept in the Jenkins credentials store, not in the code. When a build fails, I read the console output from the first error, not the last line.

**My example** [Hands-on] (Story 10)
> "We had dev pipelines running several times for one change, because the synced commit in the destination repository started the pipeline again. I updated the pipeline setup so one change triggers one run."

**Keywords:** controller + agents · Jenkinsfile stages · triggers (webhook, polling, timer, manual) · credentials store · console log first error · one trigger per change

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| Explain Jenkins architecture. | Sentences 1–2 |
| How do you trigger a build? | Sentence 4 |
| Polling vs webhook? | "Polling checks Git on a schedule; a webhook is Git telling Jenkins immediately. Use one, not both." |
| A pipeline ran twice. Why? | Story 10 |

**Practice:** What's in a Jenkinsfile? · How do you avoid duplicate triggers?

---

## M3 · Maven and the AEM project structure

**In simple words**
Maven builds the project. An AEM project has modules for Java code, components, configs, content and Dispatcher.

**Say this**
> Maven is the build tool for AEM projects. The `pom.xml` defines dependencies, plugins and modules, and the lifecycle runs compile, test, package, verify and install. An AEM project has several modules: `core` for Java code, `ui.apps` for components, `ui.config` for OSGi configurations and Repoinit, `ui.content` for initial content like templates, `ui.frontend` for CSS and JavaScript, `dispatcher` for the Dispatcher config, and `all`, which packages everything for deployment. In the cloud, code packages are immutable and content packages are mutable, and they must not be mixed. Common build failures are missing dependencies, the wrong Java version or failing tests.

**My example** [Hands-on]
> "I've worked with Maven builds and Java versions in our pipelines." *(Add one real detail you are sure of.)*

**Keywords:** pom.xml · lifecycle (compile, test, package, verify, install) · core · ui.apps · ui.config · ui.content · ui.frontend · dispatcher · all · immutable vs mutable

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is Maven? | Sentences 1–2 |
| Explain the AEM project structure. | Sentence 3 |
| A Maven build fails. | Sentence 5 + "I check `dependency:tree`, the repository settings and the Java version." |

**Practice:** Name the AEM project modules · What does each lifecycle phase do?

---

## M4 · The Cloud Manager pipeline

**In simple words**
Cloud Manager builds your code once, tests it, deploys to Stage, waits for approval, then deploys to Production without downtime.

**The flow, step by step**
1. **Source**: the pipeline takes a branch from Git (start manually or on Git change).
2. **Build** with Maven and run unit tests.
3. **Code quality** scan: critical issues stop the pipeline.
4. **Build image**: your code + the current AEM release.
5. **Deploy to Stage** and run tests (functional, UI, performance, security).
6. **Approval** (go-live sign-off).
7. **Deploy to Production** with a rolling update: no downtime; if the new version fails, the old one keeps running.

**Diagram script**
> "The pipeline starts from Git. It builds the code and runs unit tests, then a code quality scan. Then it builds the image with our code and the AEM release, and deploys it to Stage, where tests run. After approval, the same build goes to Production with a rolling deployment, so there's no downtime."

**Say this**
> In AEM as a Cloud Service, all code goes through Cloud Manager pipelines. The production pipeline takes code from Git, builds it with Maven and runs unit tests, then runs a code quality scan where critical issues stop the pipeline. It builds an image with our code and the current AEM release and deploys it to Stage, where functional and performance tests run. After approval, the same build is deployed to Production with a rolling deployment, so there is no downtime. There are also pipelines just for Dispatcher config, front-end code and CDN config.

**My example** [Hands-on]
> "I created pipelines for RDE, QA and Stage and worked with Adobe's pipelines. I also deployed Dispatcher and Repoinit changes through them." (Stories 15, 6, 2)

**Keywords:** Git → build + unit tests → code quality → image → Stage + tests → approval → Prod (rolling, no downtime) · pipeline types

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you deploy to AEMaaCS? | All |
| What are the pipeline stages? | The 7 steps |
| What happens if code quality fails? | "Critical issues stop it; important ones can be overridden by an approver." |
| What pipeline types exist? | "Full-stack, web-tier config for Dispatcher only, front-end, and config pipelines for CDN rules." |
| How is RDE deployed? | "With the Adobe I/O CLI, not a normal pipeline, so it takes seconds." |

**Practice:** Draw the production pipeline · Why is the same build deployed to Stage and Prod?

---

## M5 · Release, change control and rollback

**In simple words**
A production release is a controlled change: approved, tested, with a plan to go back.

**Say this**
> A production release goes through change control. We prepare a change request with the scope, the test evidence from QA and Stage, the deployment window, the approvers and a rollback plan. After deployment we validate: key pages, forms, publishing, and the logs for new errors. In AEM as a Cloud Service there is no one-click rollback for code, so we roll back by redeploying the previous version from Git through the pipeline. Content issues can be fixed by restoring or re-running content steps.

**My example** [Hands-on] (Story 3)
> "When an ingestion broke workflows in production, we reverted to the last good state and re-ran it with a corrected file, then added a review step before every ingestion." *(Add your release approval process.)*

**Keywords:** change request (scope, evidence, window, approvers, rollback plan) · validate after deploy · rollback = redeploy previous commit · content restore

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you manage production releases? | All |
| How do you roll back in AEMaaCS? | Sentence 3 |
| QA works but Stage fails. Why? | "I compare what's different: the commit, environment variables, run-mode configs, content and data." |
| How do you validate a deployment? | Sentence 2 |

**Practice:** What's in a change request? · How do you roll back a bad release?
