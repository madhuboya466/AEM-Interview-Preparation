# 07 · AEM internals: Sling, OSGi, JCR, HTL

6 modules · Day 11–13 · Long version: [04-aem-internals](../04-aem-internals/java-sling-osgi-jcr.md)

One picture to keep in mind:
```
URL → Sling (find resource, pick script) → HTL → Sling Model → OSGi services → JCR
```

---

## M1 · How Sling processes a request

**In simple words**
Sling turns a URL into a content node, then uses that node's type to choose the script that renders it.

**The flow, step by step**
1. A URL arrives, e.g. `/content/site/en/home.teaser.html`.
2. Sling **splits** it: path `/content/site/en/home`, selector `teaser`, extension `html`.
3. **Resource resolution**: finds the node at that path.
4. Reads the node's **`sling:resourceType`**.
5. **Script resolution**: finds the matching script under `/apps` (then `/libs`), using selectors and extension; if none, follows `sling:resourceSuperType`.
6. The script (HTL) **renders**, including child components.
7. The **response** is returned.

**Diagram script**
> "Sling takes the URL and splits it into path, selectors and extension. It finds the content node at that path, reads its resource type, and uses that to choose the script. The script renders the page, including each component, and returns the HTML."

**Say this**
> Sling is the web framework in AEM, and it is resource-based. When a request comes in, Sling splits the URL into the resource path, selectors, extension and suffix. It finds the content node at that path, then reads its `sling:resourceType` to decide which script renders it. It looks in `/apps` first, then `/libs`, and uses the selectors and extension to pick the most specific script; if none is found, it follows the resource super type. The HTL script renders the content and returns the response.

**My example** [Hands-on] (Story 7)
> "On preview, links without `.html` didn't open the right page, because Sling uses the extension to pick the HTML rendering. I added a Dispatcher rule to append `.html`."

**Keywords:** URL split (path, selector, extension, suffix) · resolve resource · resourceType · script: /apps then /libs · superType · render

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is Sling? | Sentences 1, 3 |
| How does AEM resolve a URL? | All |
| What is a selector? | "Part of the URL between the path and extension, like `.teaser`, used to choose a different rendering." |
| Why is an existing page 404? | "Sling can't find or read the resource: not published, no read permission, or wrong path." |

**Practice:** Explain request processing with an example URL · What is resourceSuperType for?

---

## M2 · Sling Models

**In simple words**
A Sling Model is a Java class that gives data to the HTL script.

**Say this**
> A Sling Model is a Java class that maps content to an object the HTL script can use. It is annotated with `@Model` and adapts from a request or a resource. Annotations inject values: `@ValueMapValue` for properties from the dialog, `@ChildResource` for child nodes, and `@OSGiService` for services. A `@PostConstruct` method runs after injection for extra logic. If a required injection fails, the model is not created and the component shows nothing, so I check the logs and use optional injection where values may be missing.

**My example** [Hands-on] (Story 12)
> "I fixed the List component's 'Display as Teaser' option by updating the teaser delegate property and upgrading the List from version 3 to version 4. Core Components use Sling Models, and we can extend them with the delegation pattern."

**Keywords:** Java class for HTL · `@Model` adapt request/resource · `@ValueMapValue` · `@ChildResource` · `@OSGiService` · `@PostConstruct` · failed injection = empty component

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is a Sling Model? | All |
| How does a component get its data? | "The dialog saves properties on the node, the Sling Model reads them, HTL displays them." |
| How do you extend a Core Component? | "Proxy component plus the delegation pattern: my model uses `@Via(ResourceSuperType)` and overrides only what I need." |
| A component renders nothing. | Last sentence |

**Practice:** Name four Sling Model annotations · Why might a model return null?

---

## M3 · Servlets and filters

**In simple words**
A servlet handles special requests (like JSON). A filter runs code before or after requests.

**Say this**
> A Sling servlet handles a request directly in Java, for example to return JSON. It can be registered by resource type, with selectors and extensions, or by a fixed path. Resource-type registration is preferred, because it respects content permissions and works better with caching; path-based servlets must be allowed in the configuration and the Dispatcher. A Sling filter runs before or after requests, for example to add security headers, and its scope can be request or component level. If a servlet doesn't respond, I check that its component is active and that the Dispatcher allows the URL.

**My example** [Theory]
> "In my platform role I check servlet and filter issues from the outside: whether the URL is allowed in the Dispatcher and whether the bundle is active."

**Keywords:** servlet = Java handles request · by resource type (preferred) or path · filter = before/after · scopes · active + Dispatcher allowed

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| Resource type vs path servlet? | Sentence 3 |
| What is a Sling filter? | Sentence 4 |
| My servlet returns 404. | Last sentence |

**Practice:** Why prefer resource-type servlets? · Give a use for a filter.

---

## M4 · OSGi

**In simple words**
OSGi is the module system. Code is deployed as bundles, which offer services and read configurations.

**Say this**
> OSGi is the module system inside AEM, running on Apache Felix. Code is deployed as bundles, which are JAR files that declare which packages they import and export. Components are classes managed by OSGi; they can provide services and use other services through references. Configuration is supplied per run mode, for example `config.author.prod`, and in the cloud we use environment variables and secrets for environment-specific values. A bundle stuck in "installed" has a missing dependency, and a component that is "unsatisfied" is missing a required service or configuration.

**My example** [Hands-on] (Story 5)
> "The email issue was an OSGi configuration: the mail service's token expired. I updated the configuration with a new token. A better long-term setup is to keep such a value as a Cloud Manager secret variable."

**Keywords:** module system (Felix) · bundles import/export · components + services + references · config per run mode · env vars + secrets · installed = missing dependency · unsatisfied = missing service/config

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is OSGi? | All |
| What are run modes? | Sentence 4 |
| A bundle is not active. | Sentence 5 |
| How do you manage config per environment in the cloud? | "Run-mode folders plus Cloud Manager environment variables, using `$[env:NAME]` and `$[secret:NAME]`." |

**Practice:** Explain bundle vs component vs service · Why might a component be unsatisfied?

---

## M5 · JCR, Oak and queries

**In simple words**
JCR is the content database: a tree of nodes. Oak is the implementation. Queries need indexes to be fast.

**Say this**
> The JCR is AEM's content repository: a tree of nodes with properties, and Apache Oak is the implementation. Each node has a type, like `cq:Page` or `dam:Asset`, and names can use namespaces that must be registered first. Binary files like images are stored in a separate blob store. Queries are written in JCR-SQL2 or QueryBuilder, and they need an Oak index; without one, Oak traverses the repository, which is slow and shows a warning in the logs. In the cloud, custom indexes are deployed through the pipeline.

**My example** [Hands-on] (Story 6)
> "The migration namespace errors were a JCR rule: a custom namespace must be registered before content uses it. I registered it with Repoinit."

**Keywords:** tree of nodes + properties · Oak · node types · namespaces registered · blob store · queries need indexes · traversal warning · index via pipeline

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is JCR? | Sentences 1–3 |
| A search is slow. | Sentence 4 |
| How do you add an index in the cloud? | Sentence 5 |

**Practice:** What is traversal and why is it bad? · Where are binaries stored?

---

## M6 · Components and HTL

**In simple words**
A component is a folder with an HTL script, a dialog and a model. Pages are built from components.

**The flow, step by step** (rendering a component)
1. The page script includes each child component.
2. Sling resolves the component's **resource type** → its HTL script.
3. HTL uses **`data-sly-use`** to load the **Sling Model**.
4. The model reads **properties** saved by the **dialog**.
5. HTL outputs the HTML (with automatic XSS escaping).

**Say this**
> A component is a reusable building block with an HTL script, a dialog for authors, and usually a Sling Model. When a page renders, each component is included as a separate sub-request, and Sling finds its script by resource type. HTL loads the Sling Model with `data-sly-use`, and the model reads the properties the author saved in the dialog. HTL then outputs the HTML and escapes it automatically to prevent XSS. Editable templates and policies control which components authors can use on a page.

**My example** [Hands-on] (Story 4, Story 12)
> "Because every component is a sub-request, a page with too many nested containers becomes slow; that caused a 503 in my project. I also upgraded the List component from v3 to v4 and fixed its teaser setting."

**Keywords:** script + dialog + model · each component = sub-request · resourceType → HTL · `data-sly-use` model · dialog properties · XSS escaping · templates + policies

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is an AEM component? | Sentence 1 |
| How does a component render? | The flow |
| Why can too many components slow a page? | Sentence 2 + Story 4 |
| What is HTL? | "AEM's HTML template language; it is secure by default because it escapes output." |

**Practice:** Explain the parts of a component · How do dialogs and models connect?
