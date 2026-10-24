# AZ-104 – Résumé CloudLee : Entra ID · IAM · Entra Extended · Virtual Networking

> Suit l'ordre des chapitres du cours CloudLee. ⚠️ = piège classique d'examen.

---

## 1. Entra ID Essentials

### Tenant, abonnements, domaines
- **Tenant** = instance Entra ID (`xxx.onmicrosoft.com`). Un abonnement fait confiance à **un seul** tenant ; un tenant peut avoir **plusieurs** abonnements.
- Changer le tenant d'un abonnement (*Change directory*) → ⚠️ **toutes les attributions RBAC sont perdues**, et les managed identities doivent être recréées.
- **Abonnement gratuit** : 200 $ de crédit pour 30 jours + services gratuits 12 mois, avec des quotas faibles (vCPU par région/famille).
- **Domaine personnalisé** : on l'ajoute, puis on crée un enregistrement **TXT** (ou MX) chez le registrar → *Verify*. On peut ensuite le définir comme domaine principal.

### Identités utilisateurs
- Types : **Member** / **Guest** (B2B). Sources : cloud, synchronisé (Entra Connect), externe.
- ⚠️ L'**usage location** est obligatoire avant d'assigner une licence.
- ⚠️ Un utilisateur supprimé est restaurable **30 jours** (soft delete).
- **Bulk** : Users → *Bulk operations* → create / invite / delete via un **CSV** téléchargé depuis le portail.

```bash
az ad user create --display-name "Test User" --user-principal-name test@<tenant>.onmicrosoft.com --password 'P@ssw0rd123!'
az ad user list --query "[].{upn:userPrincipalName, type:userType}" -o table
```

### Identités applicatives
- **App registration** = **application object** (la définition de l'app) + **service principal** (l'instance dans le tenant, visible dans *Enterprise applications*).
- Authentification de l'app : **secret** ou **certificat** (préféré).
- Les permissions API sont soit **delegated** (au nom d'un utilisateur), soit **application** (l'app agit seule). Ces dernières demandent l'**admin consent**.

### Managed identities
| | **System-assigned** | **User-assigned** |
|---|---|---|
| Cycle de vie | Lié à la ressource (supprimé avec elle) | Indépendant |
| Partage | 1 ressource | Plusieurs ressources |
| Cas typique | Une VM accède à Key Vault | Le même accès pour toute une flotte, ou pré-provisionné |

- Aucun secret à gérer. On donne ensuite un **rôle RBAC** à l'identité, par exemple *Storage Blob Data Reader*.

```bash
az vm identity assign -g labo-rg1 -n vm0                          # System-assigned
az identity create -g labo-rg1 -n id-lab                          # User-assigned
az vm identity assign -g labo-rg1 -n vm0 --identities id-lab
```

### Groupes, licences, groupes dynamiques
- Types : **Security** (accès, RBAC, licences) ou **Microsoft 365** (mail, Teams, SharePoint).
- Membership : **Assigned**, **Dynamic user**, **Dynamic device**.
- ⚠️ **Dynamic** = licence **P1**. Plus de membres ajoutés à la main, et le calcul prend un délai.
- ⚠️ **Dynamic device** → groupes **Security** uniquement.
- Exemple de règle : `(user.department -eq "Sales") and (user.country -eq "BE")`
- **Licences par groupe** (P1) : en cas d'erreur (usage location manquant, licences épuisées), l'état apparaît dans Groupe → *Licenses*.

```bash
az ad group create --display-name grp-lab --mail-nickname grp-lab
az ad group member add --group grp-lab --member-id <objectId>
```

### Administrative Units (AU)
- Conteneur logique d'users, de groupes et d'appareils, utilisé pour **déléguer l'administration** sur un sous-ensemble (ex. « Belgique »).
- On assigne un rôle Entra (*User Administrator*, *Helpdesk Administrator*…) **scopé à l'AU**. Cela demande **P1** pour les admins de l'AU.
- ⚠️ Une AU n'hérite pas et n'est pas hiérarchique : pas d'AU imbriquées.
- *Restricted management AU* : même un Global Admin du tenant ne peut pas modifier ses objets sans rôle sur l'AU.

---

## 2. Identity and Access Management (IAM)

### Azure RBAC vs Entra ID roles
| | **Azure RBAC** | **Entra ID roles** |
|---|---|---|
| Contrôle | Ressources Azure (VM, storage…) | Objets Entra (users, groupes, apps) |
| Scopes | MG → Subscription → RG → Resource | Tenant ou Administrative Unit |
| Exemples | Owner, Contributor, Reader, User Access Admin | Global Admin, User Admin, Helpdesk Admin |

- ⚠️ Un **Global Admin n'a pas** d'accès aux ressources Azure par défaut. Il peut s'élever via *Properties → Access management for Azure resources*, ce qui lui donne User Access Administrator sur `/`.
- ⚠️ RBAC est **additif** (on fait l'union des rôles) et **hérité vers le bas**.

### Rôles Azure clés
| Rôle | Peut |
|---|---|
| **Owner** | Tout + gérer les accès |
| **Contributor** | Tout **sauf** gérer les accès |
| **Reader** | Lecture |
| **User Access Administrator** | Gérer les accès uniquement |
| *Role Based Access Control Administrator* | Gérer les accès, avec possibilité de conditions qui limitent les rôles attribuables |

- ⚠️ **Control plane ≠ data plane** : *Reader* ou *Contributor* sur un storage account ne permet pas de lire les blobs avec Entra ID. Il faut *Storage Blob Data Reader/Contributor*.

### Custom roles
- **Azure** : JSON avec `Actions`, `NotActions`, `DataActions`, `NotDataActions`, `AssignableScopes`.
  ⚠️ `NotActions` **n'est pas un refus** : il retire l'action de ce rôle, mais un autre rôle peut toujours l'accorder.
- **Entra** : permissions sur les objets Entra (ex. `microsoft.directory/users/password/update`). ⚠️ Demande **P1**.

```bash
az role assignment create --assignee test@<tenant>.onmicrosoft.com --role "Virtual Machine Contributor" -g labo-rg1
az role assignment list -g labo-rg1 -o table
az role definition list --name "Contributor" --query "[].permissions"
az role definition create --role-definition custom-role.json
```

---

## 3. Entra ID Extended Services

### Hybride
- **Entra Connect** (ou **Cloud Sync**, plus léger) : synchronise AD DS → Entra ID, toutes les **30 min** par défaut.
  Méthodes d'authentification :
  - **PHS**, Password Hash Sync : le plus simple, recommandé
  - **PTA**, Pass-Through Auth : un agent on-prem valide le mot de passe
  - **Fédération** (AD FS)
- **Entra Domain Services** : domaine **managé** (LDAP, Kerberos, NTLM, GPO) pour les apps legacy dans Azure.
  ⚠️ Il demande son propre subnet. Il n'y a **pas de Domain Admin**, juste le groupe *AAD DC Administrators*. Les users cloud doivent changer leur mot de passe pour que les hash soient générés.

### External Identities
- **B2B** : on invite des partenaires, qui gardent leur propre identité. Leur compte guest a un UPN contenant `#EXT#`.
  Réglages dans *External collaboration settings* : qui peut inviter, domaines autorisés ou bloqués. La **fédération Google** est possible.
- **B2C / External ID for customers** : tenant séparé, destiné aux clients d'une application.

### Gouvernance (licence P2 / Entra ID Governance)
| Fonction | Rôle |
|---|---|
| **Entitlement management** | **Access packages** (groupes + apps + sites) dans un **catalog**, avec demande, approbation et expiration |
| **PIM** | Rôles **eligible** (activés à la demande, limités dans le temps, avec MFA/justification/approbation) vs **active** |
| **Access reviews** | Revue périodique des membres de groupes, accès aux apps ou rôles, avec suppression automatique possible |
| **ID Protection** | Détection du **user risk** et du **sign-in risk** → politiques (MFA, changement de mot de passe, blocage) |

### Contrôle d'accès
- **Conditional Access** (**P1**) : *si* signaux (user/groupe, app, localisation/IP, appareil, risque) *alors* bloquer ou exiger (MFA, appareil conforme…).
  ⚠️ Il est incompatible avec les **Security defaults**, qu'il faut désactiver. Utiliser le mode *Report-only* pour tester.
- **SSPR** (**P1** ; writeback vers AD via Entra Connect) : portée *None / Selected / All*, 1 ou 2 méthodes requises, inscription obligatoire.
- **App Proxy** (**P1**) : publie une app web **on-prem** à l'extérieur via un **connector** (connexions sortantes uniquement, pas de port entrant). On peut le combiner avec la pré-auth Entra et le Conditional Access.

### Récap licences
| Free | P1 | P2 |
|---|---|---|
| Users, groupes assignés, B2B, RBAC, Security defaults | Dynamic groups, licences par groupe, Conditional Access, SSPR, App Proxy, custom Entra roles, rôles sur AU | PIM, ID Protection, Access reviews, Entitlement management |

---

## 4. Virtual Networking

### VNet & subnets
- Un VNet appartient à **une région** et à **un abonnement**. Il a un ou plusieurs espaces d'adresses (RFC 1918 recommandés).
- ⚠️ Azure **réserve 5 IP** par subnet : `.0` (réseau), `.1` (passerelle), `.2` et `.3` (DNS Azure), et la dernière (broadcast). Un /29 n'offre donc que **3 IP** utilisables, et la 1re VM reçoit `.4`.
- ⚠️ Prévoir des espaces d'adresses **sans chevauchement** avec on-prem et les autres VNets, pour pouvoir faire du peering ou du VPN plus tard.

### IP addressing
- **Privée** : dynamique (DHCP Azure, par défaut) ou statique (réservée sur la NIC). ⚠️ On ne configure **jamais** une IP fixe dans l'OS.
- **Publique** : ⚠️ **Standard** uniquement (Basic retirée) → toujours **statique**, *secure by default* (fermée tant qu'un NSG ne l'ouvre pas), zone-redundant possible.
- Elle s'attache à une **NIC**, un **Load Balancer**, un **NAT Gateway**, **Bastion**, une **App Gateway**… ⚠️ jamais à un subnet.
- **Public IP prefix** : un bloc d'IP publiques contiguës.
- **NAT Gateway** : associé à un **subnet**, il ne gère que le **sortant** et il est **prioritaire** sur l'IP publique d'une VM pour le trafic sortant.
  Les nouveaux VNets ont désormais des **subnets privés par défaut**, sans accès sortant implicite → NAT Gateway, IP publique ou LB nécessaire.

```bash
az network vnet create -g labo-rg1 -n vnet1 --address-prefix 10.10.0.0/16 --subnet-name subnet1 --subnet-prefix 10.10.10.0/24
az network public-ip create -g labo-rg1 -n nat-pip --sku Standard
az network nat gateway create -g labo-rg1 -n natgw1 --public-ip-addresses nat-pip
az network vnet subnet update -g labo-rg1 --vnet-name vnet1 -n subnet1 --nat-gateway natgw1
az network nic ip-config update -g labo-rg1 --nic-name vm0-nic -n ipconfig1 --private-ip-address 10.10.10.10   # IP privée statique
```

### Network Security Groups (NSG)
- Règles 5-tuple : source, port source, destination, port destination, protocole → **Allow / Deny**.
- **Priorité 100–4096** : la plus petite est évaluée en premier, et la **première règle qui correspond gagne**.
- Règles par défaut (non supprimables, mais surchargeables) :
  - Inbound : `AllowVnetInBound` 65000 · `AllowAzureLoadBalancerInBound` 65001 · `DenyAllInBound` 65500
  - Outbound : `AllowVnetOutBound` · `AllowInternetOutBound` · `DenyAllOutBound`
- Association : **subnet** et/ou **NIC**. Un seul NSG par subnet ou NIC, et un même NSG peut servir à plusieurs. Il doit être dans la même région.
- ⚠️ Avec des NSG à **deux niveaux**, il faut que **les deux autorisent** le trafic.
  - Entrant : on passe le NSG du **subnet**, puis celui de la **NIC**.
  - Sortant : on passe le NSG de la **NIC**, puis celui du **subnet**.
- NSG **stateful** : le retour d'une connexion autorisée est automatiquement permis.
- Diagnostic : *Effective security rules* sur la NIC, et **IP flow verify** (Network Watcher).

### Augmented security rules
- Une seule règle peut contenir **plusieurs IP/plages**, **plusieurs ports** et des **service tags** (`Internet`, `VirtualNetwork`, `AzureLoadBalancer`, `Storage`, `Sql`…).
- **ASG** (Application Security Group) : on regroupe des NIC par rôle (`asg-web`, `asg-db`) et on les utilise comme source ou destination. Plus besoin de maintenir des listes d'IP. ⚠️ Toutes les NIC d'un ASG doivent être dans le **même VNet**.

```bash
az network nsg rule create -g labo-rg1 --nsg-name nsg1 -n allow-rdp-me --priority 100 \
  --source-address-prefixes <mon-ip> --destination-port-ranges 3389 --access Allow --protocol Tcp
az network asg create -g labo-rg1 -n asg-web
az network nic ip-config update -g labo-rg1 --nic-name vm0-nic -n ipconfig1 --application-security-groups asg-web
az network nsg rule create -g labo-rg1 --nsg-name nsg1 -n allow-web --priority 200 \
  --source-address-prefixes Internet --destination-asgs asg-web --destination-port-ranges 80 443 --access Allow --protocol Tcp
az network nic list-effective-nsg -g labo-rg1 -n vm0-nic
```

---

**Méthode** : après chaque chapitre, faire le **quiz CloudLee** + 1 lab en CLI, puis le **Practice Assessment** Microsoft. Ne retravailler que ce qui est < 80 %.
