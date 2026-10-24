# 2. Identity and Access Management (IAM)

⚠️ = exam trap

## Azure RBAC vs Entra ID roles

| | Azure RBAC | Entra ID roles |
|---|---|---|
| Controls | Azure resources (VMs, storage…) | Entra objects (users, groups, apps) |
| Scope | Management group → Subscription → Resource group → Resource | Tenant or Administrative Unit |
| Examples | Owner, Contributor, Reader | Global Admin, User Admin, Helpdesk Admin |

- ⚠️ A **Global Admin has no access** to Azure resources by default. To get it: Entra ID → Properties → *Access management for Azure resources*. This gives User Access Administrator at root scope (`/`).

## Key Azure roles

| Role | Can |
|---|---|
| Owner | Everything + manage access |
| Contributor | Everything **except** manage access |
| Reader | Read only |
| User Access Administrator | Manage access only |
| Role Based Access Control Administrator | Manage access, with conditions that limit which roles can be assigned |

## How RBAC works

- ⚠️ **Additive**: a user gets the sum of all their roles.
- ⚠️ **Inherited** downwards: a role on a subscription applies to all its resource groups and resources.
- ⚠️ **Control plane ≠ data plane**: Reader or Contributor on a storage account does **not** let you read blobs with Entra ID. You need *Storage Blob Data Reader* or *Contributor*.

## Custom roles

- **Azure**: a JSON file with `Actions`, `NotActions`, `DataActions`, `NotDataActions`, `AssignableScopes`.
- ⚠️ `NotActions` is **not a deny**: it only removes the action from this role. Another role can still allow it. Only *deny assignments* block.
- **Entra**: permissions on Entra objects (e.g. `microsoft.directory/users/password/update`). ⚠️ Needs **P1**.

## Commands

```bash
az role assignment create --assignee test@<tenant>.onmicrosoft.com --role "Virtual Machine Contributor" -g labo-rg1
az role assignment list -g labo-rg1 -o table
az role definition list --name "Contributor" --query "[].permissions"
az role definition create --role-definition custom-role.json
```
