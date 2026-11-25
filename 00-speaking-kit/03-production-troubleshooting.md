# 03 · Production troubleshooting

6 modules · Day 4–6 · Long version: [01-production-support](../01-production-support/production-support.md)

---

## M1 · How I troubleshoot: find the layer

**In simple words**
Don't guess. Follow the request from the outside in, and find the first layer where it goes wrong.

**The flow, step by step** (outside → in)
1. **Understand**: which URL, which users, since when, what changed?
2. **Reproduce** the issue myself.
3. **Author vs Publish**: does it work on Author? If not → content or code.
4. **Publish vs website**: works on Publish but not on the site? → Dispatcher or CDN.
5. **Some users vs all users**: only some? → permissions or network (IP).
6. **Logs** for that time and URL, to confirm.
7. **Fix, verify with users, prevent.**

**Diagram script**
> "I start from the user and move inwards. First I reproduce it. Then I compare Author and Publish: if Author is also broken, it's content or code. If Publish is fine but the site is not, it's the Dispatcher or CDN. If only some users are affected, I look at permissions or IP access. Then I confirm with the logs, fix it, and verify with the users."

**Say this**
> When I get a production issue, I first understand the scope: which URL, which users, since when, and what changed recently. Then I reproduce it and isolate the layer by following the request path. I compare Author and Publish to separate a content or code problem from a delivery problem, and if Publish works but the site doesn't, I check the Dispatcher and the CDN. If only some users are affected, I check what is different for them, like permissions or IP address. I confirm with the logs, apply the fix, verify with the users, and then work on prevention.

**My example** [Hands-on]
> "For the image 404s, Author and Publish both had the images, so I knew it was the Dispatcher. For the 403, only one region's users were affected, so I compared them and found their IP wasn't allowlisted." (Stories 2, 1)

**Keywords:** scope · reproduce · Author vs Publish · Publish vs site · some vs all users · logs · fix–verify–prevent

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you troubleshoot an AEM issue? | All |
| A page is not loading in production. What do you do? | All, with "page" in sentence 1 |
| What is your approach to production support? | All, then mention RCA (M5) |
| How do you find the root cause quickly? | "By isolating the layer…" sentences 2–4 |

**Practice:** Walk me through how you debug a broken page · Tell me about an issue you isolated.

---

## M2 · 4xx errors: 400, 401, 403, 404

**In simple words**
4xx means the request was refused. The important question is: which layer refused it?

**Say this**
> 4xx errors mean the request was not accepted. A 400 is a bad request, for example wrong URL encoding or headers that are too large. A 401 means the user is not logged in or the session or token has expired. A 403 means access is denied: at the CDN it is usually the IP allowlist, and inside AEM it is a missing permission for that action. A 404 means not found: either the content is not published or the user can't read it, or the Dispatcher filter blocked the request. So for every 4xx I ask which layer produced it.

**Quick table to remember**
| Code | CDN | Dispatcher | AEM |
|------|-----|-----------|-----|
| 401 | – | – | Not logged in / expired |
| 403 | IP allowlist, traffic rules | rare | Missing permission, CSRF on POST |
| 404 | wrong domain mapping | **filter deny**, rewrite | not published, no read access, wrong path |

**My example** [Hands-on]
> "403: one region's authors outside the IP allowlist. 404: `.JPG` blocked by the Dispatcher filter." (Stories 1, 2)

**Keywords:** 400 bad request · 401 login · 403 denied (IP or permission) · 404 missing (publish, read, filter) · which layer?

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you troubleshoot a 403? | 403 sentence + Story 1 + 04-M4 |
| Why would AEM return 404 for a page that exists? | "Three reasons: not published, no read permission, or a Dispatcher filter." |
| Difference between 401 and 403? | "401 is 'who are you?', 403 is 'I know who you are, but you're not allowed'." |

**Practice:** Explain 403 vs 404 in AEM · A page exists but returns 404: what do you check?

---

## M3 · 5xx errors: 500, 502, 503, 504

**In simple words**
5xx means the server side failed: an error in code, a backend not reachable, or too slow.

**Say this**
> 5xx errors are server-side problems. A 500 usually means an exception in AEM, like an error in a component or Sling Model, and I find it in the error log by reading the stack trace. A 502 means a proxy got a bad response from the backend, often while a Publish instance is restarting. A 503 means the service is unavailable, for example no healthy backend or a request that took too long. A 504 is a gateway timeout, when the backend is too slow, for example because of a slow query or a slow external call. For all of them I check the request log for the time taken and the error log for exceptions.

**My example** [Hands-on] (Story 4)
> "A regional team got a 503 after publishing a page. The page had too many containers and was too slow to render. I re-authored it with fewer containers and advised authors to keep pages light."

**Keywords:** 500 exception → stack trace · 502 bad backend · 503 unavailable/slow · 504 timeout · request.log time + error.log

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you troubleshoot a 500? | Sentence 2 + 03-M4 logs |
| The site shows 503. What do you do? | Sentence 4 + Story 4 |
| Difference between 502, 503 and 504? | Sentences 3–5 |
| A page is very slow. | Sentence 5 + 08-M4 |

**Practice:** Explain 500 vs 503 · Tell me about a 5xx you fixed.

---

## M4 · Logs and correlation

**In simple words**
Each layer has a log. You connect them using the time, the URL and the user's IP.

**The flow, step by step**
1. Get the **exact time** (convert to UTC), URL and user/IP.
2. **CDN log**: did the request arrive? Blocked? Cached?
3. **Dispatcher log**: allowed or filtered? Cache hit or miss?
4. **request.log**: status code and how long AEM took.
5. **error.log**: the exception and stack trace at that time.
6. **Recent changes**: deployments, publishes, config changes.

**Say this**
> AEM has several logs. The error log shows exceptions and warnings, the request log shows every request with its status and response time, and the access log shows who requested what. The Dispatcher has its own logs for filter decisions and cache hits, and the CDN log shows client IPs, status and cache results. In the cloud, I download them from Cloud Manager or tail them with the Adobe I/O CLI. I correlate them using the timestamp in UTC, the URL, the client IP and request IDs, and I always check what changed recently, like a deployment or a publish.

**My example** [Hands-on]
> "For the 403, the logs gave me the client IP, which showed it was outside the allowlist. For the workflow issue, the error log showed a `cq:Workflow` exception and the ingestion logs showed the cause." (Stories 1, 3)

**Keywords:** error.log (exceptions) · request.log (status + time) · access.log (who) · Dispatcher · CDN · Cloud Manager download · correlate: time, URL, IP, request ID · recent changes

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| Which logs do you use in AEM? | Sentences 1–3 |
| How do you get logs in AEMaaCS? | Sentence 4 |
| How do you read a stack trace? | "I go to the last 'Caused by' line, and the first line from our own code shows the class and line that failed." |
| How do you correlate logs across layers? | Sentence 5 |

**Practice:** Name the AEM logs and what each shows · How do you trace one request across layers?

---

## M5 · Root cause analysis

**In simple words**
The symptom is what users see. The root cause is why it was possible. Fix both.

**The flow, step by step**
1. Separate the **symptom** from the **cause**.
2. Ask **"why"** repeatedly (Five Whys) until you reach something you can change.
3. Back every statement with **evidence** (logs, timestamps).
4. Define **corrective** action (fix now) and **preventive** action (stop it happening again).
5. **Document** and share with the team.

**Say this**
> For root cause analysis, I first separate the symptom from the actual cause. Then I use the Five Whys: I keep asking why until I reach a cause we can act on, usually a missing process or check, not just "someone made a mistake". Every conclusion must be supported by evidence, like log entries and timestamps. Then I define a corrective action to fix the issue now and a preventive action to stop it from happening again. Finally I document it and share it with the team.

**My example** [Hands-on] (Story 3)
> "Symptom: workflows broken. Why? Workflow definitions were missing. Why? The ingestion wiped them. Why? The path CSV had empty rows. Why did that reach production? No review of the CSV. Corrective: revert and re-ingest. Preventive: peer review of the CSV before every ingestion."

**Keywords:** symptom vs cause · Five Whys · evidence · corrective · preventive · document

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you do RCA? | All |
| What is the difference between a fix and a root cause fix? | Sentence 4 |
| Tell me about an incident and what you learned. | Story 3 |
| How do you prevent repeat incidents? | Sentences 4–5 + Stories 3, 5 |

**Practice:** Do a Five Whys on one of your stories · What makes a good RCA?

---

## M6 · Incidents, priority and the incident lifecycle

**In simple words**
An incident is "broken now". Priority depends on how many are affected and how urgent it is.

**The flow, step by step**
Detect → log and classify → prioritise → respond → investigate → fix or workaround → verify → close → RCA if needed.

**Say this**
> An incident is an unplanned interruption, and the first goal is to restore service, even with a workaround. A problem is the underlying cause behind one or more incidents, and a change is a planned modification that goes through approval. Priority is based on impact, meaning how many users or sites are affected, and urgency, meaning how fast it must be fixed; production down with no workaround is the highest priority, P1. The lifecycle is: detect, log, prioritise, respond, investigate, fix, verify with users, and close, followed by a root cause analysis for important incidents. During the incident I give regular updates, even when there is no new finding yet.

**My example** [Hands-on]
> "When workflows broke after an ingestion, the first priority was to restore them by reverting; the CSV review came afterwards as the preventive step." (Story 3)

**Keywords:** incident = restore · problem = cause · change = approved · priority = impact × urgency · lifecycle · regular updates

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is the incident lifecycle? | Sentence 4 |
| Incident vs problem? | Sentences 1–2 |
| How do you prioritise issues? | Sentence 3 |
| How do you handle a P1? | "Restore first, update often, RCA after." + Story 3 |

**Practice:** Explain incident, problem and change · How do you decide priority?
