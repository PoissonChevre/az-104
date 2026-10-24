# 1. Entra ID Essentials

⚠️ = exam trap

## Tenant and subscriptions

- **Tenant** = one Entra ID directory (`xxx.onmicrosoft.com`).
- A subscription trusts **one** tenant. A tenant can have **many** subscriptions.
- ⚠️ Move a subscription to another tenant → **all RBAC assignments are lost**, and managed identities must be recreated.
- Free account: $200 credit for 30 days + some services free for 12 months. Low quotas (vCPU per region and VM family).

## Custom domains

1. Add the domain in Entra ID.
2. Create the **TXT** (or MX) record shown by Azure at your DNS registrar.
3. Click **Verify**. You can then make it the primary domain.

## Users

- Types: **Member** or **Guest** (B2B). Source: cloud, synced from on-prem AD (Entra Connect), or external.
- Bulk create / invite / delete: Users → *Bulk operations* → CSV template downloaded from the portal.
- ⚠️ Set the **usage location** before you assign a license.
- ⚠️ A deleted user can be restored for **30 days**.

## Groups

| | |
|---|---|
| Types | **Security** (access, RBAC, licenses) · **Microsoft 365** (mail, Teams, SharePoint) |
| Membership | **Assigned** · **Dynamic user** · **Dynamic device** |

- ⚠️ Dynamic groups need **P1**. You can't add members by hand, and updates take some time.
- ⚠️ Dynamic device → **Security** groups only.
- Rule example: `(user.department -eq "Sales") and (user.country -eq "BE")`
- **Group-based licensing** (P1): errors (no usage location, no licenses left) show in the group → *Licenses*.

## Administrative Units (AU)

- A container of users, groups and devices, to **delegate admin** of a subset (e.g. "Belgium").
- Assign an Entra role (User Administrator, Helpdesk Administrator…) **scoped to the AU**. Needs **P1**.
- ⚠️ No nesting: an AU can't contain another AU.
- *Restricted management AU*: even a Global Admin can't change its objects without a role on the AU.

## App identities

- **App registration** = **application object** (the app definition) + **service principal** (the app in the tenant, under *Enterprise applications*).
- App sign-in: **secret** or **certificate** (preferred).
- API permissions: **delegated** (on behalf of a user) or **application** (the app acts alone → needs **admin consent**).

## Managed identities

No secret to manage: you give the identity an **RBAC role** (e.g. *Storage Blob Data Reader*).

| | System-assigned | User-assigned |
|---|---|---|
| Life cycle | Deleted with the resource | Independent |
| Used by | 1 resource | Many resources |
| Typical use | One VM reads a Key Vault | Same access for a group of VMs |

## Commands

```bash
az ad user create --display-name "Test User" --user-principal-name test@<tenant>.onmicrosoft.com --password '<password>'
az ad user list --query "[].{upn:userPrincipalName, type:userType}" -o table
az ad group create --display-name grp-lab --mail-nickname grp-lab
az ad group member add --group grp-lab --member-id <objectId>
az vm identity assign -g labo-rg1 -n vm0                       # System-assigned identity
az identity create -g labo-rg1 -n id-lab                       # User-assigned identity
az vm identity assign -g labo-rg1 -n vm0 --identities id-lab
```
