---
title: Gatekeeper Policies
parent: Tenant Isolation
nav_order: 4
---

# What Gatekeeper Prevents

- Any tenant deployer SA operating outside its namespace prefix → rejected
- Any tenant deployer SA creating ClusterRoles or ClusterRoleBindings → rejected
- globex-deployer creating namespace acme-prod → rejected (prefix mismatch)
- globex-deployer creating resources in acme-prod → rejected (prefix mismatch)
