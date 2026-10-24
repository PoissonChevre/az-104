# 3. Entra ID Extended Services

**!** = exam trap

Starting point: the company has a directory **in the office** (Active Directory) and a directory **in the cloud** (Entra ID).

## Hybrid identity

**Entra Connect** copies the office accounts (AD) to the cloud (Entra ID), every **30 minutes**. Users keep the same login and password. **Cloud Sync** is the light version.

How the password is checked:

| Method | How | Note |
|---|---|---|
| **PHS** (Password Hash Sync) | A hash of the password is copied to the cloud. The cloud checks it alone. | **Simplest, recommended.** Works even if the office is down. |
| **PTA** (Pass-Through Auth) | An on-prem agent checks each password. | Office down → no sign-in. |
| **Federation** (AD FS) | The office handles the whole sign-in. | Heavy and complex. |

- **Writeback** = cloud → office. Mostly for passwords changed with SSPR.
- **!** A synced account is **edited in on-prem AD**, not in the Azure portal.

**Entra Domain Services**: a **managed AD** in Azure (LDAP, Kerberos, NTLM, GPO) for old apps that need a classic domain. No domain controller to manage.

- **!** It needs its **own subnet**.
- **!** You are **not Domain Admin**: you only get the *AAD DC Administrators* group.
- **!** Cloud users must **change their password once** before they can use it.
- Don't mix them up: Entra Connect **copies** your existing AD. Domain Services **gives you** a new AD in Azure.

## External identities

| | B2B | B2C / External ID |
|---|---|---|
| Who | **Partners** (consultants, suppliers) | **Customers** of your app |
| Where | In **your** directory, as **Guest** (`#EXT#` in the UPN) | In a **separate** directory |
| Account | Their own (company, Microsoft, Google…) | Sign-up, or Google/Facebook |

- B2B settings: *External collaboration settings* (who can invite, allowed or blocked domains). Google federation is possible.

## Access control (P1)

- **Conditional Access**: **IF** user/group, app, location/IP, device, risk… **THEN** block, or allow with MFA, compliant device…
  - **!** Turn off **Security defaults** first (they can't be used together).
  - **Report-only** mode: logs what the policy would do, without blocking. Use it to test.
- **SSPR** (self-service password reset): users reset their own password.
  - Enabled for **None / Selected group / All**. **1 or 2** methods required. Users must **register** first.
  - **!** Synced users → enable **password writeback** (Entra Connect), or the new password won't work on-prem.
- **App Proxy**: publishes an **on-prem web app** to remote users. A **connector** on an office server makes **outbound** connections only: **no inbound port** to open. Works with Entra sign-in, MFA and Conditional Access.

## Governance (P2)

These tools remove access people don't need anymore.

| Tool | What it does |
|---|---|
| **Entitlement management** | **Access packages** (groups + apps + SharePoint sites) in a **catalog**. Request → approval → access, with an **expiry date**. Works for B2B guests. |
| **PIM** (Privileged Identity Management) | Admin roles **just in time**. **Eligible** = can activate the role for a limited time (with MFA, justification, approval). **Active** = always on. |
| **Access reviews** | Regular check: "does this person still need this access?" No or no answer → access can be removed automatically. For groups, apps and roles. |
| **ID Protection** | Detects **sign-in risk** (this sign-in looks suspicious) and **user risk** (this account is probably hacked, e.g. leaked password). Policies: require MFA, password change, or block. |

## Licenses

Each level includes the one before: **Free ⊂ P1 ⊂ P2 ⊂ ID Governance**.

| Feature | Free | P1 | P2 | ID Gov |
|---|:-:|:-:|:-:|:-:|
| Users, assigned groups, B2B, Security defaults | ✓ | ✓ | ✓ | ✓ |
| Dynamic groups, group-based licensing | | ✓ | ✓ | ✓ |
| Conditional Access, SSPR, password writeback | | ✓ | ✓ | ✓ |
| App Proxy, automatic user provisioning | | ✓ | ✓ | ✓ |
| Custom Entra roles, roles scoped to an AU | | ✓ | ✓ | ✓ |
| ID Protection, PIM | | | ✓ | ✓ |
| Access reviews (basic), access packages | | | ✓ | ✓ |
| Lifecycle workflows (joiner / mover / leaver) | | | | ✓ |
| Access reviews: inactive users, AI suggestions, PIM for groups | | | | ✓ |
| Entitlement management: advanced options | | | | ✓ |

- **P1** = **control access**: Conditional Access, SSPR, App Proxy, dynamic groups.
- **P2** = **risk** + **basic governance**: ID Protection, PIM, access reviews, access packages.
- **ID Governance** = the **extras**: lifecycle workflows, inactive users, AI.
- **!** PIM, access reviews and access packages are **P2**, not ID Governance.

## Which tool? (exam style)

| Situation | Answer |
|---|---|
| Office AD accounts must exist in the cloud | **Entra Connect** (PHS) |
| An old app in Azure needs a domain, LDAP, Kerberos | **Entra Domain Services** |
| An external consultant uses their own account | **B2B** (Guest) |
| App customers must create an account | **B2C / External ID** |
| Give a set of accesses that expires automatically | **Access package** |
| An admin gets rights only when needed | **PIM** (eligible role) |
| Check every quarter who still needs an access | **Access review** |
| Block an account whose password leaked | **ID Protection** (user risk) |
| Require MFA outside the office | **Conditional Access** |
| Users reset their password themselves | **SSPR** |
| Password changed with SSPR doesn't work on-prem | Enable **password writeback** |
| Reach the office intranet from home, no VPN, no open port | **App Proxy** |

## Quiz: which license?

Cover the license table above and answer. Each answer is the **lowest** license that includes the feature.

1. Dynamic groups
2. PIM
3. Conditional Access
4. Lifecycle workflows
5. Access packages
6. SSPR
7. ID Protection
8. Access reviews for inactive users
9. B2B guests
10. App Proxy

<details>
<summary>Answers</summary>

1. P1 · 2. P2 · 3. P1 · 4. ID Governance · 5. P2 · 6. P1 · 7. P2 · 8. ID Governance · 9. Free · 10. P1

</details>
