---
title: Tenant Isolation
parent: Multi-Tenancy
nav_order: 5
has_children: true
---

# Tenant Isolation

Tenant isolation is enforced at the infrastructure level. Tenants can provision
workloads into any namespace name.

Tenants have no direct Kubernetes API access — the cluster is not exposed to
tenants. All tenant interaction with the platform goes through platform APIs
(grammateus, source handler APIs). Direct Kubernetes API access by a tenant is
treated as a security leak.

## Trust Chain

The isolation model is a dependency chain. Each link depends on the one above
it. If any link breaks, everything below it is compromised.

```
1. Kubernetes SA identity (tamper-proof — SA cannot change its own username)
        ↓
2. Label immutability (Gatekeeper — no one can modify katastroma.org/tenant after creation)
        ↓
3. Tenant namespace label (set by grammateus before tenant SAs exist — protected by #2)
        ↓
4. Identity derivation (Gatekeeper — derives tenant identity from SA's namespace label)
        ↓
5. Resource ownership (Gatekeeper — resource label + namespace label must match
   tenant identity; cluster-scoped resources checked by resource label only)
        ↓
6. Resource isolation (RBAC + Gatekeeper — no read, no ClusterRoleBindings,
   escalation prevention)
```

### Link 1: SA Identity

A Kubernetes SA's username (`system:serviceaccount:<namespace>:<name>`) is
assigned by the API server and cannot be modified by the SA itself. This is the
root of the trust chain — the one thing a tenant cannot tamper with.

**Enforced by:** Kubernetes API server authentication
([Kubernetes: Authenticating](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
— "Service accounts authenticate with the username
`system:serviceaccount:(NAMESPACE):(SERVICEACCOUNT)`")

### Link 2: Label Immutability

Gatekeeper enforces that the `katastroma.org/tenant` label is immutable after
creation on all resources. No SA — tenant or platform — can modify or remove it.
Without this, no label-based control downstream holds — any SA with `patch`
could rewrite its own identity or claim another tenant's namespaces.

**Enforced by:** Gatekeeper custom ConstraintTemplate. On UPDATE operations,
Gatekeeper compares `input.review.oldObject.metadata.labels` against
`input.review.object.metadata.labels` and rejects if the tenant label changed.
On CREATE, `oldObject` is null — the initial label is allowed.
([Gatekeeper: Constraint Templates](https://open-policy-agent.github.io/gatekeeper/website/docs/constrainttemplates/))

### Link 3: Tenant Namespace Label

Grammateus creates the tenant namespace and sets the `katastroma.org/tenant`
label **before** creating the tenant SAs. The label exists before the SAs exist.
The SAs never had an opportunity to influence their own identity — and because
the label is immutable (Link 2), they never will.

**Enforced by:** Platform onboarding sequence (grammateus). This is an ordering
guarantee in the onboarding flow, protected by label immutability.

### Link 4: Identity Derivation

When a tenant SA makes any request, Gatekeeper extracts the SA's namespace from
`input.review.userInfo.username`, looks up that namespace in the synced resource
cache, and reads its `katastroma.org/tenant` label. This is the SA's tenant
identity. Because the label is immutable (Link 2) and was set before the SA
existed (Link 3), the tenant SA cannot influence the identity Gatekeeper
derives.

**Enforced by:** Gatekeeper — two capabilities:

- SA identity via `input.review.userInfo.username`
  ([Gatekeeper: Admission Review Input](https://open-policy-agent.github.io/gatekeeper/website/docs/input/))
- Namespace label lookup via `data.inventory` synced cache
  ([Gatekeeper: Replicating Data](https://open-policy-agent.github.io/gatekeeper/website/docs/sync/)
  — synced namespaces are accessible at
  `data.inventory.cluster["v1"]["Namespace"][name]`)

**Caveat:** `input.review.userInfo` is not populated during Gatekeeper audit
(periodic background checks). Constraints relying on SA identity enforce at
admission time only. Drift detection via audit is out of scope — admission is
the security boundary. Any resource that exists in the cluster was admitted
through the constraint at creation time.
([Gatekeeper: Admission Review Input](https://open-policy-agent.github.io/gatekeeper/website/docs/input/))

### Link 5: Resource Ownership

Gatekeeper enforces two checks on every tenant SA operation:

1. **Resource label** — the target resource's own `katastroma.org/tenant` label
   must match the SA's derived identity. Applies to all resources, namespaced
   and cluster-scoped.

2. **Namespace label** — for namespaced resources, the namespace's own
   `katastroma.org/tenant` label must also match the SA's derived identity. This
   enforces namespace containment as the Kubernetes-native security boundary.

Gatekeeper determines which checks apply from
`input.review.object.metadata.namespace`: non-empty means namespaced (both
checks); empty means cluster-scoped (check 1 only). No resource type is treated
specially.

Namespace names are not checked — a tenant can name a namespace anything as long
as it carries their tenant identity label.

**Enforced by:** Gatekeeper custom ConstraintTemplate combining identity
derivation (Link 4) with incoming resource labels
(`input.review.object.metadata.labels`) and namespace lookup (`data.inventory`).
([Gatekeeper: Constraint Templates](https://open-policy-agent.github.io/gatekeeper/website/docs/constrainttemplates/),
[Gatekeeper: Replicating Data](https://open-policy-agent.github.io/gatekeeper/website/docs/sync/))

### Link 6: Resource Isolation

Three controls complete the isolation:

- **No read permissions** — tenant SAs have no `get`, `list`, or `watch`. A
  tenant SA cannot discover or read resources in any namespace, including its
  own. Cross-tenant read isolation is enforced by the absence of read verbs, not
  by namespace boundaries.

  **Enforced by:** Kubernetes RBAC — verbs are explicit, RBAC is additive-only.
  ([Kubernetes: Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/))

- **No ClusterRole or ClusterRoleBinding creation** — Gatekeeper blocks tenant
  SAs from creating these cluster-scoped RBAC resources. A tenant cannot grant
  itself or any SA broader permissions.

  **Enforced by:** Gatekeeper custom ConstraintTemplate matching on resource
  kind + `input.review.userInfo.username`.
  ([Gatekeeper: Constraint Templates](https://open-policy-agent.github.io/gatekeeper/website/docs/constrainttemplates/))

- **RBAC escalation prevention** — Kubernetes prevents any SA from creating
  RoleBindings that grant more permissions than the SA itself has. Even within
  namespaces the tenant owns, it cannot escalate.

  **Enforced by:** Kubernetes RBAC authorizer (enabled by default in all modern
  clusters).
  ([Kubernetes: Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
  — "The RBAC API prevents users from escalating privileges by editing roles or
  role bindings")

## Impersonation

The provisioner and pruner run with their own platform SAs in the platform
namespace. When operating on tenant resources, they impersonate the
corresponding tenant SA using Kubernetes impersonation (Impersonate-User HTTP
headers). The Kubernetes API server authenticates the platform SA first, then
switches to the impersonated identity. Authorization is evaluated against the
impersonated SA's constraints — not the platform SA's.

**Enforced by:** Kubernetes API server impersonation.
([Kubernetes: User Impersonation](https://kubernetes.io/docs/reference/access-authn-authz/user-impersonation/)
— "Request user info is replaced with impersonation values. The request is then
evaluated, with authorization acting on the impersonated user info.")

## Resource Ownership

Every provisioned resource is labeled by the
[labeler](../event-driven/labeler.md). The `katastroma.org/tenant` label is the
ownership record used for isolation enforcement, pruning, and querying.

## Cluster-Scoped Resources

Tenant SA ClusterRoles use `*` for resources so tenants can provision custom
resources from any CRD. Gatekeeper blocks tenant SAs from creating dangerous
cluster-scoped resources:

- ClusterRoles
- ClusterRoleBindings

**Open issue:** CRDs are cluster-scoped. Blocking tenant CRD creation prevents
tenants from installing operators or charts that include CRDs. Allowing it means
CRDs are visible cluster-wide to all tenants.

Candidate mitigations:

- Block tenant CRD creation entirely — tenants request CRDs through the
  platform, which vets and installs them.
- Allow CRD creation but use Gatekeeper to restrict which API groups tenants can
  register.
- Allow CRD creation with labeling — all tenant-created CRDs are labeled with
  the tenant identity for cleanup and auditing.

## What Gatekeeper Prevents

- Any tenant SA operating on a resource not labeled with their tenant identity →
  rejected
- Any tenant SA operating on a namespaced resource in a namespace not labeled
  with their tenant identity → rejected
- Any mutation to the `katastroma.org/tenant` label after resource creation →
  rejected
- Any tenant SA creating ClusterRoles or ClusterRoleBindings → rejected
- Any non-grammateus SA creating ServiceAccounts in tenant namespaces → rejected

## SA Creation in Tenant Namespaces

A rogue SA in a tenant namespace inherits the same tenant identity (derived from
the namespace label) and cannot escape the tenant's isolation boundary. It
cannot obtain a ClusterRoleBinding (blocked by Gatekeeper). The label-based
model contains any SA created within a tenant's namespaces to that tenant's
identity.

As defense in depth, Gatekeeper restricts ServiceAccount creation in tenant
namespaces to grammateus's platform SA. Gatekeeper identifies grammateus by its
SA identity in the platform namespace — the same
`input.review.userInfo.username` mechanism used throughout the trust chain. If
any other SA attempts to create a ServiceAccount in a tenant namespace, the
request is rejected.

**Enforced by:** Gatekeeper custom ConstraintTemplate matching on resource kind
(ServiceAccount) + requesting SA identity.
([Gatekeeper: Constraint Templates](https://open-policy-agent.github.io/gatekeeper/website/docs/constrainttemplates/))

## SA Token Secret Access

Kubernetes warns that any user with write access to Secrets can request a token,
and any user with read access to Secrets can authenticate as the service account
([Kubernetes: Managing Service Accounts](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)).
The tenant provisioner SA has `create` and `patch` on all resources, which
includes Secrets. This risk is contained by the isolation model:

- **Cannot read tokens** — tenant SAs have no `get`, `list`, or `watch`. They
  cannot read any existing SA token Secrets.
- **Cannot create tokens outside their namespaces** —
  `kubernetes.io/service-account-token` Secrets must be in the same namespace as
  the SA they reference. Gatekeeper blocks tenant SAs from creating Secrets in
  any namespace not labeled with their tenant identity. They cannot mint tokens
  for platform SAs or other tenants' SAs.
- **Tokens within tenant namespaces are contained** — any SA in a namespace
  labeled with a tenant's identity inherits that tenant identity. A token
  created for such an SA authenticates within the same tenant boundary. No
  escape.
