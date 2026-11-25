# 09 · Support process and customer handling

5 modules · Day 16–17 · Long version: [06-adobe-tse-support](../06-adobe-tse-support/support-and-sla.md)

> Your angle: you handled production incidents from the **customer side** of an AEM as a Cloud Service platform **[Hands-on]**. Adobe's internal tools and exact SLA numbers are **[Theory]**: describe the process, not tool names.

---

## M1 · The support case lifecycle

**In simple words**
A case goes from "reported" to "solved and documented", and the owner keeps it moving.

**The flow, step by step**
1. **Case opened** by the customer.
2. **Triage**: product, environment, priority.
3. **Qualify**: clear problem statement, impact, data needed.
4. **Investigate**: reproduce, isolate, evidence.
5. **Outcome**: solution or workaround, or **escalate** to engineering if it's a product bug.
6. **Customer validates** the fix.
7. **Close** with a summary, and write a **knowledge article** if useful.

**Diagram script**
> "The customer opens a case. I triage it and set the priority, then qualify it into a clear problem statement. I investigate: reproduce, isolate the layer, collect evidence. Then either I give a solution or workaround, or I escalate to engineering if it's a product bug. The customer validates, and I close the case with a summary and, if useful, a knowledge article."

**Say this**
> A support case starts when the customer reports an issue. I triage it, set the priority based on impact, and turn the report into a clear problem statement with the environment, URLs, time and impact. Then I investigate: I reproduce it, isolate the layer and collect evidence like logs. The result is either a solution or workaround for the customer, or an escalation to engineering if it is a product defect. I close the case only after the customer confirms, with a short summary and a knowledge article if it can help others.

**My example** [Hands-on]
> "In my project I followed the same steps for production issues: understand the scope, reproduce, find the layer, fix, confirm with the users, and share what we learned." (Stories 1–3)

**Keywords:** open · triage + priority · problem statement · investigate (reproduce, isolate, evidence) · solve or escalate · customer confirms · close + KB

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| Explain the support lifecycle. | All |
| How do you handle a new case? | Sentences 2–3 |
| When do you close a case? | Last sentence |

**Practice:** Walk through a case from start to end · What is a good problem statement?

---

## M2 · Priority, SLA and ownership

**In simple words**
Priority depends on business impact. SLA is the agreed response time. The owner is responsible until the end.

**Say this**
> Priority is based on business impact and urgency. P1 is critical, for example a production site down with no workaround; P2 is a major function badly affected; P3 is a partial issue with a workaround; and P4 is a question or minor issue. The SLA defines how quickly we must respond and how often we update the customer, based on the priority and the support contract. As the case owner, I am responsible until the case is closed, even when engineering or other teams are working on it. That means I track every open action and keep the customer informed.

**Keywords:** impact × urgency · P1 down · P2 major · P3 workaround · P4 question · SLA = response + updates · owner until closed

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you decide priority? | Sentences 1–2 |
| What is an SLA? | Sentence 3 |
| What does case ownership mean? | Sentences 4–5 |

**Practice:** Give an example of P1 vs P3 in AEM · What does an owner do during escalation?

---

## M3 · Escalating to engineering

**In simple words**
Escalate when it's a real product bug, and give engineering everything they need to start immediately.

**Say this**
> I escalate to engineering when the issue is a product defect, when it needs platform access I don't have, or when a P1 is not progressing. Before escalating, I confirm it is reproducible without the customer's custom code and that it differs from documented behaviour. The escalation includes a clear title, the customer impact, environment and versions, exact steps to reproduce, expected versus actual results, logs with timestamps, and what I have already ruled out. While engineering works on it, I stay the owner, give the customer a workaround if possible, and update them regularly.

**Keywords:** when (defect, platform, stuck P1) · reproduce without custom code · title, impact, env, steps, expected vs actual, logs, ruled out · stay owner + workaround + updates

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| When do you escalate? | Sentence 1 |
| What do you include in an escalation? | Sentence 3 |
| How do you know it's a bug, not customer code? | Sentence 2 |

**Practice:** List what goes into an escalation · How do you keep the customer updated?

---

## M4 · Customer communication

**In simple words**
Be quick, clear, honest and specific. Explain in simple words first, then the technical detail.

**Say this**
> I acknowledge the issue quickly and restate it so the customer knows I understood the impact. In every update I say what I checked, what I found, what I will do next, and when the next update will come. I explain the cause in simple words first and add the technical detail after. I am honest about what I don't know yet, and I never present a guess as a fact. With a frustrated customer I listen first, acknowledge the impact, and focus on the next concrete action.

**Simple explanation example** (your Story 2)
> "The images were published correctly, but the web server's security rules only allowed lowercase file extensions. Your images use uppercase `.JPG`, so they were blocked. We updated the rule and they load now. To avoid this in future, we'll allow both cases."

**Keywords:** acknowledge + restate · checked / found / next / when · simple first, technical after · honest · frustrated: listen, acknowledge, next action

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you communicate during an incident? | Sentences 1–2 |
| How do you explain technical issues to non-technical people? | Sentence 3 + example |
| How do you handle an angry customer? | Last sentence |
| The customer wants a fix today, but it's not possible. | "I explain what is possible now, offer a workaround, and give a clear time for the next update." |

**Practice:** Explain one of your stories to a non-technical person · What's in a good update?

---

## M5 · Knowledge and prevention

**In simple words**
Every solved problem should help the next person: a KB article, notes, or a runbook.

**Say this**
> After solving a case, I check whether it can help others. A knowledge base article describes the symptom in the customer's words, the cause, the resolution and any workaround, so the next person finds it quickly. For major incidents, a root cause report explains what happened, why, and the preventive actions. I also keep clear case notes, so a colleague can take over at any time. And I share lessons with the team, which is how repeat incidents go down.

**My example** [Hands-on] (Stories 3, 4, 5)
> "After incidents, I informed the team and added preventive steps: a peer review of the migration CSV, guidance to authors to use fewer containers, and a monthly token refresh."

**Keywords:** KB (symptom, cause, fix, workaround) · RCA report · case notes · share lessons · fewer repeats

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you share knowledge? | All |
| What makes a good KB article? | Sentence 2 |
| How do you prevent repeat issues? | Last sentence + example |

**Practice:** Write a KB title for Story 2 · What do you share after an incident?
