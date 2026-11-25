# 04 · Users, groups and access

4 modules · Day 6–8 · Long version: [03-aem-administration §3](../03-aem-administration/aem-administration.md)

---

## M1 · Two layers of access in AEM as a Cloud Service

**In simple words**
Layer 1 decides if you can log in. Layer 2 decides what you can do once you are in.

**The flow, step by step**
1. User opens the author URL → **CDN IP allowlist** (network check).
2. **Adobe IMS** login → is the user in the right **product profile** in Admin Console for this environment?
3. On first login, AEM creates the **user** and syncs their **groups**.
4. For every action, AEM checks **ACLs** on the content path (read, modify, publish).

**Diagram script**
> "First the network: the IP must be allowed. Then login through Adobe IMS: the user must be in the product profile for that environment. Inside AEM, the user and groups are synced, and finally every action is checked against the permissions on that content path."

**Say this**
> In AEM as a Cloud Service, access works in two layers. The first layer is login: users sign in through Adobe IMS, and the Admin Console product profiles decide which environments they can access. The second layer is inside AEM: users belong to groups, and permissions, called ACLs, on content paths decide what they can read, edit or publish. There can also be a network layer in front, the IP allowlist at the CDN. So when someone has an access issue, I check the network, then the login, then the AEM permissions.

**My example** [Hands-on] (Story 11)
> "I set up access for country authors and approvers: login access through Admin Console, and inside AEM, groups per country and role with permissions on that country's content."

**Keywords:** network (IP) · login (IMS + product profile) · AEM groups · ACLs on paths · check in that order

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How does user management work in AEMaaCS? | All |
| A user can't log in to author. | "I check the IP allowlist, then the product profile in Admin Console…" |
| A user logs in but can't see a site. | "That's the second layer: groups and ACLs…" |
| How did you set up access for authors? | Story 11 |

**Practice:** Explain the two layers · User can log in but can't edit a page: why?

---

## M2 · Permissions (ACLs) and least privilege

**In simple words**
Permissions are rules on content paths. Give them to groups, not people, and give only what is needed.

**Say this**
> Permissions in AEM are access control entries on repository paths: a user or group, allow or deny, and privileges like read, modify, create, delete and replicate. They are inherited down the content tree, and a rule on a deeper path overrides the one above it. At the same node, rules for a user win over rules for a group. Best practice is to give permissions to groups, not individual users, and to follow least privilege: each person gets only what their role needs. For example, authors can edit their own country's content, and only approvers can publish it.

**My example** [Hands-on]
> "Our country authors had edit rights only on their country's content, and approvers handled publishing through the approval workflow." (Story 11)

**Keywords:** ACE = who + allow/deny + privilege + path · inherited · deeper wins · user beats group · groups not users · least privilege

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do AEM permissions work? | All |
| What is least privilege? | Sentence 4 + example |
| Why grant to groups, not users? | "Easier to manage and audit; when someone changes role, you change membership, not permissions." |
| How do you check a user's permissions? | "Tools → Security → Permissions: I can see effective permissions and test a privilege on a path." |

**Practice:** How is a permission evaluated? · Design access for authors and approvers.

---

## M3 · Service users

**In simple words**
Code should not run as admin. It runs as a technical user with only the rights it needs.

**Say this**
> A service user is a technical user that code uses to read or write content, instead of an admin session. We create the service user and its permissions with a Repoinit script, and map our code bundle to it with a service user mapping configuration. Then the code asks for a resource resolver for that sub-service and gets exactly those permissions. This follows least privilege and is required in AEM as a Cloud Service. If the mapping or the user is missing, the code fails with a login exception.

**My example** [Adjacent]
> "I've used Repoinit in my project to create repository structure, like registering a namespace across all environments. Service users are created in the same way." (Story 6)

**Keywords:** technical user · no admin · Repoinit creates · mapping config · resolver per sub-service · LoginException if missing

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| What is a service user? | All |
| Why not use admin in code? | "Security and least privilege, and admin sessions are not allowed in the cloud." |
| Code fails with LoginException. | Last sentence: check the mapping and the user |

**Practice:** How do you create a service user in the cloud? · What happens if the mapping is missing?

---

## M4 · Troubleshooting a 403 or "no access"

**In simple words**
Find out where the request is denied: network, login, or AEM permission.

**The flow, step by step**
1. **Who** is affected: everyone, a group, a region, one user?
2. Denied **before** login? → IP allowlist at the CDN.
3. Login error / environment missing? → Admin Console product profile.
4. Logged in but action fails? → AEM groups and ACLs on that path.
5. POST fails? → CSRF token or servlet permission.
6. Fix, **verify with the user**, prevent.

**Say this**
> For a 403, I first find out who is affected, because that points to the layer. If the user is blocked before even reaching the login, I check the IP allowlist at the CDN. If the login fails or the environment is not available, I check the product profile in Admin Console. If they log in but an action fails, I check their groups and the permissions on that content path. After the fix, I ask the user to test and confirm.

**My example** [Hands-on] (Story 1)
> "Only one region's authors were affected. Their AEM permissions were fine, but the logs showed their IP wasn't in the allowlist. I added their CIDR range and they could log in."

**Keywords:** who? · before login → IP · login → product profile · after login → groups + ACL · POST → CSRF · verify

**Same answer, different questions**
| They ask | Start with |
|----------|-----------|
| How do you troubleshoot a 403 on author? | All + Story 1 |
| Some users can't access AEM. | Sentence 1 + Story 1 |
| An author can't publish. | "That's a permission issue: replicate privilege, or they must use the approval workflow." |

**Practice:** Tell me your 403 story · Author can't publish a page: what do you check?
