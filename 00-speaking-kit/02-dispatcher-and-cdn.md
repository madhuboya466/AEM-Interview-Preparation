# 02 · Dispatcher and CDN

5 modules · Day 2–4 · Long version: [01-production-support §6](../01-production-support/production-support.md)

---

## M1 · What the Dispatcher does

**In simple words**
Dispatcher is an Apache web server module in front of Publish. It is a security guard, a cache and a URL handler.

**Say this**
> The Dispatcher is Adobe's module for the Apache web server, and it sits in front of AEM Publish. It has three jobs. First, security: filter rules allow only the URLs we expect and block everything else. Second, caching: it stores rendered pages and files, so Publish does less work and pages load faster. Third, URL handling: virtual hosts and rewrite rules map domains to the right site and handle redirects. In AEM as a Cloud Service, the Dispatcher configuration is kept in Git and deployed through Cloud Manager.

**My example** [Hands-on]
> "I worked on our Dispatcher configuration for domain mapping across RDE, QA, Stage and Production, added a rewrite rule to append `.html` to links, and fixed filter rules that blocked `.JPG` images." (Stories 8, 7, 2)

**Keywords:** Apache module · in front of Publish · security (filters) · cache · URL handling (vhosts, rewrites) · Git + pipeline

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is the Dispatcher? | All |
| Why do we need Dispatcher if we have a CDN? | "The CDN caches at the edge, but the Dispatcher also protects Publish with filter rules and handles URL rewrites close to AEM." |
| How is Dispatcher configured in the cloud? | Last sentence + 06-M4 |
| What did you do with Dispatcher? | My example |

**Practice:** What are the three jobs of Dispatcher? · How do you deploy a Dispatcher change in the cloud?

---

## M2 · Filter rules and why blocked requests return 404

**In simple words**
Filters are a list of allow and deny rules. If a request is denied, Dispatcher answers 404 and Publish never sees it.

**The flow, step by step**
1. A request arrives at the Dispatcher.
2. Dispatcher checks it against the filter rules, from top to bottom.
3. **The last rule that matches wins.**
4. If the final result is deny, Dispatcher returns **404**.
5. If allowed, the request goes on to the cache and then Publish.

**Say this**
> Dispatcher filter rules decide which requests are allowed to reach Publish. The best practice is to deny everything first and then allow only the paths, extensions and selectors the site needs. The rules are checked in order and the last matching rule wins. When a request is denied, the Dispatcher returns a 404, so it looks like the page is missing even though it exists on Publish. Also, the patterns are case-sensitive, so `jpg` and `JPG` are different.

**My example** [Hands-on] (Story 2)
> "Images with an uppercase `.JPG` extension were returning 404 on our live site. They existed on Author and Publish, so I checked the Dispatcher and found the filter only allowed lowercase extensions. I allowed the extension, and the images loaded."

**Keywords:** allow/deny · deny all first · last match wins · denied = 404 · case-sensitive

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do Dispatcher filters work? | All |
| Content is on Publish but not on the website. Why? | "One common reason is a Dispatcher filter…" + Story 2 |
| Why would an image return 404? | Story 2 first, then the rule |
| How do you secure AEM Publish? | Sentences 1–2, plus "deny paths like /crx and /system" |

**Practice:** Why does a blocked request show 404, not 403? · Explain a filter issue you fixed.

---

## M3 · Dispatcher caching and invalidation

**In simple words**
Dispatcher saves pages as files. When content is published, it marks old files as stale so the next request gets a fresh copy.

**The flow, step by step**
1. First request: cache miss → Dispatcher asks Publish → saves the response as a file.
2. Next requests: Dispatcher returns the file directly.
3. An author publishes a page.
4. Dispatcher is told to **invalidate**: it touches a `.stat` file.
5. Cached files older than the `.stat` file are treated as stale.
6. The next request fetches a fresh copy from Publish and caches it again.

**Diagram script**
> "The first request misses the cache, so Dispatcher gets the page from Publish and stores it. After that, it serves the stored file. When an author publishes, Dispatcher updates the stat file, which makes the old cached files stale. The next request gets a fresh page from Publish, and the cycle starts again."

**Say this**
> The Dispatcher caches responses as files on disk. On a cache miss it gets the page from Publish and stores it; after that it serves the file directly. When content is published, the Dispatcher cache is invalidated using a `.stat` file, so older cached files become stale and the next request gets a fresh copy. In AEM as a Cloud Service this invalidation happens automatically on publish. Requests with query strings, or without an extension, are usually not cached.

**My example** [Theory, with a link to your work]
> "I haven't tuned cache rules in depth, but when debugging content issues I check whether the problem is a cached copy, for example by adding a query string, which bypasses the Dispatcher cache."

**Keywords:** cache as files · miss → Publish → store · publish → `.stat` → stale · automatic in cloud · query string not cached

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How does Dispatcher caching work? | All |
| What is cache invalidation? | Sentence 3 |
| I published but I still see old content. Why? | "There are two caches…", then 02-M4 CDN TTL |
| What is statfileslevel? | "It sets how deep `.stat` files go, so publishing in one site doesn't invalidate other sites." |

**Practice:** Explain the cache flow · Why might a page with `?id=1` not be cached?

---

## M4 · CDN: caching at the edge, and "old content after publish"

**In simple words**
The CDN is a global cache in front of everything. It keeps pages for a set time (TTL), even after you publish.

**Say this**
> In AEM as a Cloud Service, every environment includes an Adobe-managed CDN. It terminates SSL, applies IP allowlists and traffic rules, and caches content close to users. The CDN caches responses based on the Cache-Control headers, for a time called the TTL. The Dispatcher cache is cleared on publish, but the CDN keeps its copy until the TTL ends. So if content looks old after publishing, I check the response headers such as Age and Cache-Control to see whether it came from the CDN cache.

**My example** [Hands-on]
> "I worked on the CDN side for go-live: SSL certificates, DNS pointing to Adobe's CDN, and IP allowlists." (Story 8)

**Keywords:** Adobe-managed CDN · SSL + IP rules + cache · Cache-Control → TTL · Dispatcher cleared, CDN waits · check Age header

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What does the CDN do in AEMaaCS? | Sentences 1–2 |
| Old content after publishing? | Sentences 3–5 |
| How do you check if a page is cached? | "I check the response headers: Age, Cache-Control, and the cache status; or add a query string to bypass the Dispatcher cache." |

**Practice:** Why can old content appear after publish? · Which headers show caching?

---

## M5 · Going live: domain, SSL, DNS and IP allowlist

**In simple words**
To put a site on a real domain, you add the certificate, add the domain, point DNS to Adobe, and map it in Dispatcher.

**The flow, step by step**
1. **Add the SSL certificate** in Cloud Manager.
2. **Add the custom domain** in Cloud Manager and verify ownership.
3. **Map the domain** to the environment.
4. **Point DNS** (CNAME) to Adobe's CDN; the network team usually does this.
5. **Dispatcher** virtual host and rewrites map the domain to the right site content.
6. **Restrict access** if needed with an IP allowlist (CIDR ranges).

**Diagram script**
> "First the certificate, then the domain, then the mapping to the environment, then DNS pointing to Adobe's CDN, and finally the Dispatcher config that sends the domain to the right content. If an environment should be private, we add an IP allowlist."

**Say this**
> To go live with a custom domain in AEM as a Cloud Service, I first add the SSL certificate in Cloud Manager, then add the domain and map it to the environment. The network team points the DNS record to Adobe's CDN. In the Dispatcher, the virtual host and rewrite rules map the domain to the correct site content. Access can be restricted with an IP allowlist in Cloud Manager using CIDR ranges; a request from an IP outside the list gets a 403 at the CDN, before it reaches AEM.

**My example** [Hands-on] (Stories 8 and 1)
> "I did this setup for our sites, and I also solved a 403 for one region's authors whose IP range was missing from the allowlist."

**Keywords:** certificate → domain → mapping → DNS to CDN → Dispatcher vhost → IP allowlist (403)

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you set up a custom domain? | All |
| What is IP allowlisting? | Last sentence + Story 1 |
| What is CIDR? | "A way to write an IP range: `/24` means 256 addresses, `/32` means one address." |
| Site works on the Adobe URL but not on our domain? | "Then I check DNS, the certificate, the domain mapping and the Dispatcher virtual host." |

**Practice:** List the go-live steps in order · What happens when an IP is not in the allowlist?
