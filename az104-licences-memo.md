# AZ-104 – Mémo licences Entra ID

> Chaque niveau inclut le précédent : **Free ⊂ P1 ⊂ P2 ⊂ ID Governance**.

| Fonction | Free | P1 | P2 | ID Governance |
|---|:-:|:-:|:-:|:-:|
| **Base** | | | | |
| Users et groupes simples | ✅ | ✅ | ✅ | ✅ |
| B2B (inviter des Guests) | ✅ | ✅ | ✅ | ✅ |
| Security defaults | ✅ | ✅ | ✅ | ✅ |
| **Contrôle d'accès (P1)** | | | | |
| Groupes dynamiques | | ✅ | ✅ | ✅ |
| Conditional Access | | ✅ | ✅ | ✅ |
| SSPR (pour les users du cloud) | | ✅ | ✅ | ✅ |
| Password writeback | | ✅ | ✅ | ✅ |
| App Proxy | | ✅ | ✅ | ✅ |
| Provisioning automatique des users | | ✅ | ✅ | ✅ |
| **Risque + gouvernance de base (P2)** | | | | |
| ID Protection (sign-in risk / user risk) | | | ✅ | ✅ |
| PIM (Privileged Identity Management) | | | ✅ | ✅ |
| Access Reviews (de base) | | | ✅ | ✅ |
| Entitlement Management (Access Packages) | | | ✅ | ✅ |
| **Extras (ID Governance)** | | | | |
| Entitlement Management – options avancées | | | | ✅ |
| Access Reviews – PIM pour les groupes | | | | ✅ |
| Access Reviews – users inactifs | | | | ✅ |
| Access Reviews – suggestions par IA (ML) | | | | ✅ |
| Lifecycle Workflows (arrivée / changement / départ) | | | | ✅ |

## À retenir

- **P1** = **contrôler l'accès** : Conditional Access, SSPR, App Proxy, groupes dynamiques.
- **P2** = **risque** + **gouvernance de base** : ID Protection, PIM, Access Reviews, Access Packages.
- **ID Governance** = les **extras** : Lifecycle Workflows, users inactifs, IA.
- ⚠️ Piège : PIM, Access Reviews et Access Packages = **P2**, pas ID Governance.
