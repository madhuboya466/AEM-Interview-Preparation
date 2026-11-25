# 11 · Terraform

4 modules · Day 19–20 · Long version: [08-terraform](../08-terraform/terraform-for-aem.md) · Commands: [08-terraform/commands-reference](../08-terraform/commands-reference.md)

> **[Theory] subject.** Bridge sentence:
> "I haven't used Terraform in my project, but I did the same kind of changes manually: DNS records to Adobe's CDN, SSL certificates and IP allowlists. Terraform is how I'd make those changes repeatable and reviewed."

---

## M1 · What Terraform is and the workflow

**In simple words**
You write the infrastructure you want in files. Terraform shows what it will change, then makes the change.

**The flow, step by step**
1. **Write** `.tf` files describing the desired infrastructure.
2. `terraform init`: download providers, set up the backend.
3. `terraform plan`: show what will be created, changed or destroyed.
4. **Review** the plan (in a pull request).
5. `terraform apply`: make the changes.
6. **State** is updated with what now exists.

**Diagram script**
> "I write the configuration, run init to download the providers, then plan to see exactly what will change. After review, I run apply, which makes the changes through the providers, and Terraform updates its state file."

**Say this**
> Terraform is an Infrastructure as Code tool. We describe the infrastructure we want in configuration files, keep them in Git, and Terraform creates or changes the real resources through providers for each platform, such as AWS, Azure or a DNS provider. The workflow is init, plan and apply: init downloads the providers, plan shows exactly what will be added, changed or destroyed, and apply makes the change. Terraform is declarative, so we describe the end result, not the steps. Because changes go through Git and a reviewed plan, they are repeatable, auditable and the same in every environment.

**Keywords:** Infrastructure as Code · files in Git · providers · init → plan → apply · declarative · repeatable + reviewed

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is Terraform? | All |
| What is Infrastructure as Code? | Sentences 1, 5 |
| Explain the Terraform workflow. | Sentence 3 |
| Declarative vs imperative? | "Declarative says what the end result should be; imperative lists the commands to run." |

**Practice:** Explain init, plan, apply · Why is IaC better than console changes?

---

## M2 · State

**In simple words**
State is Terraform's record of what it created. Keep it remote, locked and secret.

**Say this**
> Terraform keeps a state file that maps the resources in the code to the real objects and their IDs. In a team, the state is stored in a remote backend, like an S3 bucket or Azure storage, with locking so two people can't apply at the same time. The state can contain secrets, so it must be encrypted and never committed to Git. If someone changes a resource manually, that is called drift, and the next plan shows the difference. To bring an existing resource under Terraform, we use import, and to rename a resource safely we use a moved block.

**Keywords:** state = map code → real IDs · remote backend + lock · secrets → encrypt, not in Git · drift · import · moved

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is Terraform state? | Sentence 1 |
| Why use remote state? | Sentence 2 |
| What is drift? | Sentence 4 |
| A resource already exists. | Sentence 5 (import) |

**Practice:** Why lock the state? · What happens if someone edits a resource in the console?

---

## M3 · Modules, environments and pipelines

**In simple words**
Write once as a module, reuse per environment with different values, and apply through a reviewed pipeline.

**The flow, step by step** (in CI)
1. Pull request with a `.tf` change.
2. CI runs `fmt`, `validate` and a security scan.
3. CI runs `plan` and posts it for review.
4. Reviewer checks adds, changes and **destroys**.
5. Merge → `apply` the saved plan.
6. Verify (for DNS: `dig`, `curl`).

**Say this**
> A module is a reusable block of Terraform code with inputs and outputs, for example one module for site DNS records. Each environment, like dev, stage and production, uses the same module with its own variables and its own state, so a mistake in dev can't affect production. In a pipeline, a pull request triggers format and validation checks and a plan that the team reviews, especially anything that will be destroyed. After approval, the saved plan is applied and the change is verified. This is the same change-control idea as our application releases: evidence, approval, deployment and validation.

**My example** [Adjacent]
> "It's the same pattern as AEM run-mode configs and our release process: one definition, different values per environment, and approval before production."

**Keywords:** module (inputs, outputs) · same module per env · own state per env · PR → fmt/validate → plan → review destroys → apply → verify

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is a Terraform module? | Sentence 1 |
| How do you manage multiple environments? | Sentence 2 |
| How do you run Terraform in CI/CD? | Sentences 3–4 |

**Practice:** Describe a Terraform pipeline · Why one state per environment?

---

## M4 · Where Terraform fits around AEM

**In simple words**
Adobe runs AEMaaCS itself. Terraform manages what is around it: DNS, CDN, certificates, IP lists, CI tools.

**Say this**
> In AEM as a Cloud Service, Adobe manages the infrastructure, so we don't build servers or clusters for AEM itself. Terraform is useful for what is around it: DNS records pointing domains to Adobe's CDN, the verification records for domains, a customer-managed CDN if one is used, certificates, and one reviewed list of office IP ranges. It can also manage CI/CD infrastructure such as Jenkins agents and GitHub repositories. Cloud Manager settings themselves are managed through its UI, API or the Adobe I/O CLI. For self-hosted AEM 6.5, Terraform can build the whole setup: network, servers for author, publish and Dispatcher, load balancer, storage and DNS.

**My example** [Adjacent] (Stories 8, 1)
> "In my project I did these changes manually: SSL in Cloud Manager, DNS records to Adobe's CDN with the network team, and an IP allowlist update when one region's authors got a 403. With Terraform, the DNS records and the IP range list would be code, reviewed in a pull request, and the missing range would have been caught earlier."

**Keywords:** Adobe runs AEMaaCS · Terraform around it: DNS, verification, customer CDN, certificates, IP list, CI tools · Cloud Manager via UI/API/aio · 6.5: whole setup

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How can Terraform be used with AEM? | All |
| Can you manage AEMaaCS with Terraform? | Sentences 1, 4 |
| How would you automate your DNS go-live? | My example |
| Terraform for AEM 6.5? | Last sentence |

**Practice:** Name four AEM-related things Terraform can manage · Turn your 403 story into an IaC improvement.
