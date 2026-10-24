# AZ-104 – Résumé complet (structure CloudLee)

> Grands principes + points d'examen + quelques commandes. ⚠️ = piège classique.
> Poids examen : Identité & gouvernance 20–25 % · Stockage 15–20 % · Compute 20–25 % · Réseau 15–20 % · Monitoring & backup 10–15 %.

---

## 1. Entra ID Essentials

**Principes**
- **Tenant** = annuaire Entra ID. Un abonnement fait confiance à **un seul** tenant ; un tenant peut porter plusieurs abonnements.
- ⚠️ Changer le tenant d'un abonnement → **toutes les attributions RBAC sont perdues**, et les managed identities doivent être recréées.
- **Domaine personnalisé** : on l'ajoute, puis on crée un enregistrement **TXT** (ou MX) chez le registrar → *Verify*.
- **Users** : Member / Guest. Bulk via **CSV** (*Bulk operations*). Un user supprimé est restaurable **30 jours**.
- ⚠️ L'**usage location** est obligatoire avant d'assigner une licence.
- **App registration** = application object + **service principal** (*Enterprise apps*). Authentification par secret ou certificat. Permissions *delegated* vs *application* (cette dernière demande l'admin consent).
- **Managed identity** : aucun secret à gérer. On lui donne un rôle RBAC.
  - **System-assigned** : liée à la ressource, supprimée avec elle.
  - **User-assigned** : indépendante, partageable entre ressources.
- **Groupes** : *Security* ou *Microsoft 365*. Membership *Assigned*, *Dynamic user* ou *Dynamic device*.
  ⚠️ Les groupes dynamiques demandent **P1**. ⚠️ Dynamic device → groupes Security uniquement.
- **Administrative Units** : délègue l'administration d'un sous-ensemble d'users, groupes et appareils. Rôle scopé à l'AU (P1). Les AU ne s'imbriquent pas.

```bash
az ad user create --display-name "Test" --user-principal-name test@<tenant>.onmicrosoft.com --password 'P@ssw0rd123!'
az ad group create --display-name grp-lab --mail-nickname grp-lab
az vm identity assign -g labo-rg1 -n vm0                    # Managed identity system-assigned
```

---

## 2. Identity and Access Management (IAM)

| | **Azure RBAC** | **Entra roles** |
|---|---|---|
| Contrôle | Ressources Azure | Objets Entra (users, groupes, apps) |
| Scope | MG → Sub → RG → Resource | Tenant ou AU |
| Exemples | Owner, Contributor, Reader, User Access Admin | Global Admin, User Admin, Helpdesk Admin |

- **Owner** = tout + gérer les accès. **Contributor** = tout sauf les accès. **User Access Admin** = accès seulement.
- ⚠️ RBAC est **additif** et **hérité** vers le bas. `NotActions` **n'est pas un refus** (seules les *deny assignments* refusent).
- ⚠️ **Control plane ≠ data plane** : lire des blobs demande *Storage Blob Data Reader*, pas *Reader*.
- ⚠️ Un Global Admin n'a pas d'accès Azure par défaut. Il doit s'élever via *Access management for Azure resources*.
- Les rôles personnalisés **Azure** se définissent en JSON (`Actions`, `NotActions`, `DataActions`, `AssignableScopes`). Les rôles personnalisés **Entra** demandent P1.

```bash
az role assignment create --assignee <upn> --role "Virtual Machine Contributor" -g labo-rg1
az role definition create --role-definition custom-role.json
```

---

## 3. Entra ID Extended Services

- **Entra Connect / Cloud Sync** : synchronise AD → Entra toutes les 30 min. Méthode d'authentification : **PHS** (recommandé), **PTA** (agent on-prem) ou **Fédération** (AD FS).
- **Entra Domain Services** : domaine managé (LDAP, Kerberos, NTLM, GPO) pour les apps legacy. Il demande un subnet dédié. Pas de Domain Admin : seulement le groupe *AAD DC Administrators*.
- **B2B** : invités externes (`#EXT#`), réglages dans *External collaboration settings*, fédération Google possible. **External ID / B2C** = annuaire des clients d'une app.
- **Conditional Access** (P1) : *si* signaux (user, app, IP, appareil, risque) *alors* bloquer ou exiger (MFA, appareil conforme). ⚠️ Incompatible avec les *Security defaults*. Mode *Report-only* pour tester.
- **SSPR** (P1) : portée None / Selected / All, 1 ou 2 méthodes, writeback vers AD possible via Connect.
- **App Proxy** (P1) : publie une app web on-prem via un connector (connexions sortantes uniquement).
- **Gouvernance (P2)** :
  - **Access packages** (entitlement management)
  - **PIM** : rôles *eligible* vs *active*, juste-à-temps
  - **Access reviews**
  - **ID Protection** : user risk et sign-in risk

| Free | P1 | P2 |
|---|---|---|
| Users, groupes assignés, B2B, RBAC, Security defaults | Dynamic groups, licences par groupe, CA, SSPR, App Proxy, rôles custom Entra, rôles sur AU | PIM, ID Protection, Access reviews, Entitlement mgmt |

---

## 4. Virtual Networking

- Un VNet = **1 région**, **1 abonnement**. ⚠️ Prévoir des espaces d'adresses sans chevauchement (pour peering et VPN).
- ⚠️ **5 IP réservées** par subnet : `.0`, `.1`, `.2`, `.3` et la dernière. Un /29 → 3 IP utilisables, et la 1re VM reçoit `.4`.
- **IP privée** : dynamique ou statique, réglée sur la NIC (jamais dans l'OS).
- **IP publique** : **Standard** uniquement (Basic retirée), statique, *secure by default*. Elle s'attache à une NIC, un LB, un NAT, Bastion ou une App Gateway, ⚠️ jamais à un subnet.
- **NAT Gateway** : sur le **subnet**, pour le **sortant uniquement**. Il est prioritaire sur l'IP publique de la VM. Les nouveaux VNets n'ont plus d'accès sortant implicite.
- **NSG** :
  - Priorité 100–4096 : la plus petite gagne, première correspondance.
  - Règles par défaut à 65000+ (VNet autorisé, LB autorisé, tout le reste refusé en entrée).
  - **Stateful**. Associé au subnet et/ou à la NIC.
  - ⚠️ Avec deux NSG, il faut que **les deux autorisent**. En entrée : subnet puis NIC. En sortie : NIC puis subnet.
- **Règles augmentées** : plusieurs IP et ports dans une même règle, plus **service tags** (`Internet`, `VirtualNetwork`, `Storage`…) et **ASG** (groupes de NIC, dans le même VNet).

```bash
az network vnet create -g labo-rg1 -n vnet1 --address-prefix 10.10.0.0/16 --subnet-name subnet1 --subnet-prefix 10.10.10.0/24
az network nsg rule create -g labo-rg1 --nsg-name nsg1 -n allow-rdp --priority 100 \
  --source-address-prefixes <mon-ip> --destination-port-ranges 3389 --access Allow --protocol Tcp
az network nic list-effective-nsg -g labo-rg1 -n vm0-nic
```

---

## 5. Virtual Machines

**Tailles et disques**
- Familles : **B** burstable · **D** usage général · **E** mémoire · **F** calcul · **L** stockage · **N** GPU.
- ⚠️ Changer de taille redémarre la VM. Si la taille n'est pas dispo sur le cluster actuel → il faut **deallocate**.
- Disques : **OS** · **data** · **temporaire** (`D:` / `/dev/sdb`).
  ⚠️ Le disque temporaire est **perdu** au deallocate ou au redeploy : jamais de données dessus.
- Types de disque : Standard HDD < Standard SSD < Premium SSD < Premium SSD v2 < **Ultra** (IOPS réglables).
  ⚠️ On peut **agrandir** un disque, jamais le réduire. Le nombre max de data disks dépend de la taille de la VM.

**Réseau**
- La NIC doit être dans la même région que la VM. Le nombre de NIC dépend de la taille.
- ⚠️ Ajouter ou retirer une NIC demande que la VM soit **arrêtée (deallocated)**.

**Images**
- On **généralise** la VM (Sysprep / `waagent -deprovision`) → image.
  ⚠️ Une VM généralisée est **inutilisable** après coup.
- **Azure Compute Gallery** : image definitions + versions, réplication multi-région, partage.

**Configuration**
- **Custom Script Extension** : télécharge et exécute un script au déploiement ou après.
- **Run Command** : exécution ponctuelle.
- **cloud-init** : Linux.
- **Automation State Configuration (DSC)** : en cours de remplacement par *Machine Configuration*.

**VMSS**
- Orchestration **Uniform** (VMs identiques) vs **Flexible** (VMs différentes, recommandé).
- Autoscale : basé sur une **métrique** ou un **calendrier**, avec min/max/default, règles *scale out* et *scale in*, et un **cooldown** (5 min par défaut).
- Upgrade policy : Manual / Automatic / Rolling.

**Haute disponibilité (SLA)**
| Config | SLA |
|---|---|
| VM seule (Premium SSD / Ultra) | 99,9 % |
| **Availability Set** : fault domains (max 3) + update domains (max 20, 5 par défaut) | 99,95 % |
| **Availability Zones** (2+ zones) | 99,99 % |
- ⚠️ Une VM ne peut être mise dans un availability set **qu'à sa création**.
- *Proximity placement group* = faible latence entre VMs.

**Chiffrement**
- **SSE** : toujours actif, clés gérées par Microsoft ou par le client (CMK).
- **Encryption at host** : chiffre aussi le disque temporaire et le cache.
- **ADE** : BitLocker (Windows) ou DM-Crypt (Linux), clés dans **Key Vault**, qui doit être dans la **même région** et avoir *enabled for disk encryption*. Son retrait est annoncé : préférer encryption at host.

```bash
az vm create -g labo-rg1 -n vm1 --image Ubuntu2404 --size Standard_D2als_v6 --admin-username azureuser --generate-ssh-keys
az vm resize -g labo-rg1 -n vm1 --size Standard_D4als_v6
az vm disk attach -g labo-rg1 --vm-name vm1 --name data1 --new --size-gb 64
az vm extension set -g labo-rg1 --vm-name vm1 --name CustomScript --publisher Microsoft.Azure.Extensions \
  --settings '{"commandToExecute":"apt-get -y install nginx"}'
az vmss create -g labo-rg1 -n vmss1 --image Ubuntu2404 --instance-count 2 --orchestration-mode Flexible
```

---

## 6. Storage

**Comptes**
- Le plus courant : **Standard GPv2** (blob, file, queue, table). Variantes Premium : block blobs, file shares, page blobs.
- Nom : 3–24 caractères, minuscules et chiffres, **unique au monde**.
- Endpoints : `<compte>.blob.core.windows.net`, `.file.`, `.queue.`, `.table.`, `.dfs.` (ADLS Gen2 = *hierarchical namespace*, à activer à la création).
- Types de blob : **block** (fichiers), **append** (logs), **page** (disques VHD).

**Accès**
| Méthode | Retenir |
|---|---|
| **Access keys** (2) | Accès total. Rotation : utiliser key2 pendant qu'on régénère key1. On peut désactiver *Shared key access*. |
| **SAS** | Accès limité dans le temps, les permissions et les IP. Types : account SAS, service SAS, **user delegation SAS** (signé par Entra, le plus sûr). |
| **Stored access policy** | Sur un container. Permet de **révoquer** les service SAS qui s'y rattachent. |
| **Entra ID + RBAC** | Rôles *Storage Blob Data …* (data plane). Méthode recommandée. |
| **Anonymous** | Niveau container : Private / Blob / Container. Doit aussi être autorisé au niveau du compte. |
- ⚠️ Régénérer une clé **invalide toutes les SAS** signées avec elle.

**Redondance**
| | Copies | Protège contre |
|---|---|---|
| **LRS** | 3 dans 1 datacenter | Panne de disque ou de rack |
| **ZRS** | 3 zones | Panne d'un datacenter |
| **GRS / RA-GRS** | LRS + LRS dans la région **paire** | Panne régionale (RA = lecture du secondaire) |
| **GZRS / RA-GZRS** | ZRS + LRS dans la région paire | Les deux |
- ⚠️ Après un **failover** client, le compte devient **LRS** : il faut reconfigurer la géo-réplication.

**Tiers d'accès**
- **Hot** · **Cool** (30 j min) · **Cold** (90 j) · **Archive** (180 j, **hors ligne**).
- ⚠️ Archive : il faut une **réhydratation** (jusqu'à 15 h, priorité standard ou high) avant de lire. Il n'est disponible qu'au niveau du blob, et pas sur ZRS/GZRS.
- Le tier par défaut du compte ne peut être que Hot ou Cool.

**Fonctions avancées**
- **Lifecycle management** : règles « déplacer vers Cool après X jours / supprimer après Y jours » sur base blobs, snapshots et versions. Exécution ~1 fois par jour.
- **Object replication** : copie asynchrone de block blobs entre comptes. ⚠️ Demande le **versioning** sur la source et la destination, et le **change feed** sur la source.
- **Immutable storage** (WORM) :
  - **time-based retention** : une fois *locked*, on peut seulement l'allonger
  - **legal hold**
- **Chiffrement** : SSE AES-256 toujours actif, clés Microsoft ou **CMK** dans Key Vault. La CMK demande une managed identity et un Key Vault avec soft delete + **purge protection**. Le double chiffrement s'appelle *infrastructure encryption*.
- **Soft delete** (blobs, containers, shares), **versioning**, **snapshots**.

**Azure Files**
- **SMB** (port **445**, souvent bloqué par les FAI) ou **NFS** (Premium).
- Tiers : Transaction optimized / Hot / Cool / Premium.
- **Identité** : AD DS, Entra Domain Services ou Entra Kerberos.
  - **Au niveau du partage** : rôles RBAC *Storage File Data SMB Share Reader / Contributor / Elevated Contributor*.
  - **Au niveau des fichiers** : ACL NTFS.
- **Azure File Sync** : *Storage Sync Service* → **sync group** = **1 cloud endpoint** (un partage) + des **server endpoints** (serveurs avec agent). Le **cloud tiering** garde seulement les fichiers chauds en local.

**Transfert**
- **AzCopy** : CLI, avec SAS ou `azcopy login`.
- **Storage Explorer** : interface graphique.
- **Data Box** : appareil physique, pour des To hors ligne.
- **Import/Export** : envoi de vos propres disques.

```bash
az storage account create -g labo-rg1 -n stlab$RANDOM --sku Standard_LRS --kind StorageV2 --access-tier Hot
az storage container create --account-name <sa> -n data --auth-mode login
az storage blob upload --account-name <sa> -c data -f file.txt -n file.txt --auth-mode login
az storage container generate-sas --account-name <sa> -n data --permissions rl --expiry 2026-12-31 --as-user --auth-mode login
az storage account update -g labo-rg1 -n <sa> --sku Standard_GRS
azcopy copy ./dossier "https://<sa>.blob.core.windows.net/data?<SAS>" --recursive
```

---

## 7. Advanced Networking

**UDR**
- Une route table s'associe à un **subnet**. Next hops possibles : *Virtual appliance*, *Virtual network gateway*, *Internet*, *VNet*, *None*.
- La route la plus spécifique gagne (*longest prefix*). À préfixe égal : UDR > BGP > système.
- ⚠️ Pour une NVA, il faut activer l'**IP forwarding** sur la NIC **et** dans l'OS.

**Peering**
- ⚠️ **Non transitif**. ⚠️ Espaces **sans chevauchement**. À configurer **dans les deux sens**.
- Peut être régional ou global. Pour partager une passerelle VPN : *Allow gateway transit* (hub) + *Use remote gateways* (spoke).

**Accès aux services PaaS**
| | **Service endpoint** | **Private endpoint** |
|---|---|---|
| Niveau | Subnet | NIC avec une **IP privée** dans ton VNet |
| Le service garde son IP publique | Oui | Non (accès par IP privée) |
| Accessible depuis on-prem / peering | Non | **Oui** |
| DNS | Rien | Zone `privatelink.*` |
- **Private Link Service** : exposer **ton** service (derrière un LB Standard) à d'autres VNets ou tenants.
- **Resource firewall** (storage, SQL) : accès depuis des VNets ou IP choisis, avec l'exception *trusted Microsoft services*.

**Hybride**
- **VPN Gateway** :
  - Subnet **`GatewaySubnet`** (/27 conseillé). Déploiement de 30 à 45 min.
  - **S2S** : demande un *local network gateway* (IP publique et préfixes on-prem) et une *connection*.
  - **P2S** : postes clients individuels.
  - **VNet-to-VNet**.
  - Type **route-based** (standard) vs policy-based.
- **ExpressRoute** : lien privé via un opérateur, **pas par Internet** et ⚠️ **non chiffré** par défaut.
- **Virtual WAN** : hub-and-spoke managé à grande échelle.

**DNS**
- **Zone publique** : il faut **déléguer les NS** chez le registrar.
- **Zone privée** : liée aux VNets par des *virtual network links*.
  - **Auto-registration** : enregistre les VMs automatiquement. ⚠️ Un VNet ne peut avoir qu'**une** zone d'enregistrement, mais peut en résoudre plusieurs.
- **Alias record** : pointe vers une ressource Azure (IP publique, Front Door…).

```bash
az network route-table create -g labo-rg1 -n rt1
az network route-table route create -g labo-rg1 --route-table-name rt1 -n default \
  --address-prefix 0.0.0.0/0 --next-hop-type VirtualAppliance --next-hop-ip-address 10.10.1.4
az network vnet subnet update -g labo-rg1 --vnet-name vnet1 -n subnet1 --route-table rt1
az network vnet peering create -g labo-rg1 -n v1-v2 --vnet-name vnet1 --remote-vnet vnet2 --allow-vnet-access
az network private-dns link vnet create -g labo-rg1 -z lab.local -n link1 -v vnet1 -e true
```

---

## 8. App Hosting (App Service)

**App Service plan**
- C'est le plan qui définit le calcul. Toutes les apps d'un même plan partagent ses ressources.
| Tier | Retenir |
|---|---|
| Free / Shared | Tests, pas de domaine SSL personnalisé |
| Basic | Domaine personnalisé + SSL, scale manuel |
| **Standard** | **Autoscale**, **deployment slots** (5), backups |
| Premium | Plus de slots (20), plus de puissance, zone redundancy |
| Isolated | App Service Environment, dans ton VNet |

**Scaling**
- **Scale up** = changer de tier. **Scale out** = ajouter des instances.

**Déploiement**
- Méthodes : zip deploy, GitHub Actions, Azure DevOps, conteneur.
- **Slots** : on déploie en staging, puis **swap**, sans interruption et avec rollback possible.
  ⚠️ Les réglages cochés *deployment slot setting* (« sticky ») ne suivent pas le swap.

**Réseau**
- **Entrant** : *access restrictions* ou **private endpoint**.
- **Sortant** : **VNet integration**, qui demande un subnet délégué.

**Sécurité**
- HTTPS only, certificat managé gratuit, *Easy Auth* (authentification intégrée), managed identity.

**Backups**
- À partir du tier Standard, avec une sauvegarde personnalisée vers un storage account.

```bash
az appservice plan create -g labo-rg1 -n plan1 --sku S1 --is-linux
az webapp create -g labo-rg1 -p plan1 -n app-lab$RANDOM --runtime "PYTHON:3.12"
az webapp deployment slot create -g labo-rg1 -n <app> --slot staging
az webapp deployment slot swap -g labo-rg1 -n <app> --slot staging --target-slot production
```

---

## 9. Container Hosting

**ACR (Azure Container Registry)**
- SKUs Basic / Standard / **Premium** (géo-réplication, private link).
- ⚠️ L'*admin user* est désactivé par défaut : utiliser les rôles RBAC **AcrPull** / **AcrPush**.
- **ACR Tasks** : `az acr build` construit l'image dans Azure, sans Docker en local.

**ACI (Azure Container Instances)**
- Conteneurs serverless, facturés à la seconde.
- **Container group** : plusieurs conteneurs qui partagent cycle de vie, IP et volumes (Linux uniquement pour le multi-conteneur). Déploiement en YAML.
- Restart policy : Always / OnFailure / Never. Volume possible sur Azure Files.

**AKS (Azure Kubernetes Service)**
- Plan de contrôle managé (gratuit). **Node pools** : system ou user.
- Scaling : **cluster autoscaler** pour les nœuds, **HPA** pour les pods.
- Réseau : kubenet ou **Azure CNI** (overlay).
- Liaison au registre : `--attach-acr`.

**Container Apps**
- Serverless sur Kubernetes (KEDA, Dapr). Gère les **révisions**, l'ingress et le **scale to zero**.

```bash
az acr create -g labo-rg1 -n acrlab$RANDOM --sku Basic
az acr build -r <acr> -t web:v1 .
az container create -g labo-rg1 -n aci1 --image mcr.microsoft.com/azuredocs/aci-helloworld --ports 80 --ip-address Public
az aks create -g labo-rg1 -n aks1 --node-count 2 --attach-acr <acr> --generate-ssh-keys
```

---

## 10. Load Balancing & Connectivity

| | **Load Balancer** | **Application Gateway** |
|---|---|---|
| Couche | L4 (TCP/UDP) | L7 (HTTP/S) |
| Routage | Hash 5-tuple (ou affinité IP client) | **Par chemin** (`/images`), **multi-site** (host header) |
| Extra | Inbound NAT rules, outbound rules | Terminaison TLS, **WAF**, subnet dédié |

**Load Balancer**
- Composants : frontend IP → **backend pool** → **health probe** → **LB rule**.
- ⚠️ Seule la version **Standard** existe désormais : fermée par défaut (il faut un NSG qui autorise), et les VMs du backend doivent être dans le **même VNet**.
- Il peut être **public** ou **interne**.

**Mondial**
- **Traffic Manager** : routage DNS. **Front Door** : L7 global, avec CDN et WAF.

**Bastion**
- Subnet **`AzureBastionSubnet`, /26 minimum**. RDP/SSH via le portail en HTTPS (443), sans IP publique sur les VMs.
- SKUs : Developer (gratuit) / Basic / Standard (client natif, connexion par IP) / Premium.

**Azure Firewall**
- Pare-feu managé **stateful**, dans le subnet **`AzureFirewallSubnet`** (/26).
- Règles : **DNAT** → **Network** → **Application** (FQDN), évaluées dans cet ordre.
- Pour y faire passer le trafic : une **UDR** `0.0.0.0/0` → IP privée du firewall.
- La version Premium ajoute l'inspection TLS et l'IDPS.

```bash
az network lb create -g labo-rg1 -n lb1 --sku Standard --frontend-ip-name fe --backend-pool-name be --public-ip-address lb-pip
az network lb probe create -g labo-rg1 --lb-name lb1 -n http --protocol Http --port 80 --path /
az network lb rule create -g labo-rg1 --lb-name lb1 -n web --protocol Tcp --frontend-port 80 --backend-port 80 \
  --frontend-ip-name fe --backend-pool-name be --probe-name http
az network bastion create -g labo-rg1 -n bastion1 --vnet-name vnet1 --public-ip-address bastion-pip --sku Basic
```

---

## 11. Governance & Security

**Management groups**
- Arbre sous un *Root MG*, **6 niveaux max**. Un abonnement n'a qu'**un seul parent**. Les Policy et RBAC sont hérités.

**Tags**
- Paires clé/valeur, **50 max** par ressource. ⚠️ **Non hérités** : utiliser une Policy *inherit tag*.

**Locks**
- `CanNotDelete` / `ReadOnly`, hérités. Ils s'appliquent **même à l'Owner**, qui doit retirer le lock avant d'agir.
- ⚠️ Un `ReadOnly` sur un storage account bloque *List keys*. Sur une VM, il bloque le start et le stop.

**Coûts**
- **Budgets** : alertes et action groups. ⚠️ Un budget **n'arrête pas** les ressources.
- Leviers d'économie : Cost analysis, Advisor, **Reservations** (1 ou 3 ans), Savings plans, **Azure Hybrid Benefit** (licences existantes), **Spot VMs**.

**Azure Policy**
- Policy = *quoi* est autorisé. RBAC = *qui* peut agir.
- Une **initiative** est un groupe de policies. On **assigne** au scope voulu, avec exclusions et paramètres.
- Effets : `Deny` · `Audit` · `Append` · `Modify` · `AuditIfNotExists` · `DeployIfNotExists` · `Disabled`.
- ⚠️ L'existant non conforme n'est pas corrigé automatiquement : il faut une **remediation task** (pour Modify/DINE, avec une managed identity).

**ARM / Bicep**
- Structure ARM : `parameters`, `variables`, `resources`, `outputs`.
- Mode de déploiement : **incremental** par défaut, ou **complete**. ⚠️ Le mode complete **supprime** ce qui n'est pas dans le template.
- **Nested** (inline) vs **Linked** (URI accessible, souvent avec une SAS).
- **what-if** : aperçu des changements avant déploiement.

**Déplacer des ressources**
- Entre RG ou abonnements du même tenant. Les **IDs changent**, et la source et la cible sont verrouillées pendant le move.
- Les ressources dépendantes bougent ensemble (VM + disques + NIC).
- Pour changer de **région** : **Resource Mover**.
- ⚠️ Certaines ressources ne sont pas déplaçables.

**Key Vault**
- Contient des secrets, des clés et des certificats. Permissions : **RBAC** (recommandé) ou access policies.
- **Soft delete** (7 à 90 j) + **purge protection**. ⚠️ La purge protection est **irréversible** une fois activée.
- Standard vs Premium (HSM). Une VM y accède via sa **managed identity**.

```bash
az account management-group create -n mg-lab
az tag update --resource-id <id> --operation merge --tags env=lab
az lock create -n nodel -g labo-rg1 --lock-type CanNotDelete
az policy assignment create -n allowed-loc --policy e56962a6-4747-49cd-b67b-bf8b01975c4c -g labo-rg1 \
  --params '{"listOfAllowedLocations":{"value":["belgiumcentral"]}}'
az deployment group what-if -g labo-rg1 --template-file main.bicep
az resource move --destination-group rg2 --ids <id1> <id2>
az keyvault create -g labo-rg1 -n kv-lab$RANDOM --enable-rbac-authorization true
```

---

## 12. Backup & Site Recovery

| Vault | Protège |
|---|---|
| **Recovery Services vault** | VMs, SQL/SAP dans une VM, **Azure Files**, MARS, MABS/DPM, **ASR** |
| **Backup vault** | Blobs, Managed disks, PostgreSQL, AKS |

**Règles générales**
- ⚠️ Le vault doit être dans la **même région** que la VM.
- ⚠️ La redondance du vault (LRS/ZRS/GRS) se choisit **avant la 1re sauvegarde**. La **Cross Region Restore** demande GRS.

**Sauvegarde de VM**
- Politique **Standard** (quotidienne) ou **Enhanced** (plusieurs fois par jour). ⚠️ Enhanced est **obligatoire pour Trusted Launch**.
- Snapshots *instant restore* conservés 1 à 5 jours.
- Options de restauration : nouvelle VM, remplacement des disques, restauration des disques, ou **restauration de fichiers** (script de montage).

**MARS agent**
- Sauvegarde fichiers, dossiers et system state, sur **Windows uniquement**. Pour les applis (SQL, Exchange) : MABS ou DPM.

**Supprimer un vault**
- Il faut d'abord arrêter les sauvegardes et supprimer leurs données (soft delete de **14 jours**), puis retirer les serveurs enregistrés et les private endpoints.

**App Service**
- Sauvegardes disponibles à partir du tier Standard.

**ASR (Azure Site Recovery)**
- Réplication continue vers une autre région ou depuis on-prem (Hyper-V, VMware, physique).
- **Recovery plan** : orchestre l'ordre de bascule des machines.
- **Test failover** : se fait dans un VNet isolé, sans impact sur la production.
- Cycle complet : **failover** → **commit** → **re-protect** → **failback**.
- Notions : **RPO** = perte de données acceptable · **RTO** = durée d'interruption acceptable.

```bash
az backup vault create -g labo-rg1 -n rsv-lab -l belgiumcentral
az backup protection enable-for-vm -g labo-rg1 --vault-name rsv-lab --vm vm0 --policy-name DefaultPolicy
az backup protection backup-now -g labo-rg1 --vault-name rsv-lab -c vm0 -i vm0 --backup-management-type AzureIaasVM
```

---

## 13. Monitoring

**Azure Monitor**
- **Metrics** : séries temporelles, conservées **93 jours**.
- **Logs** : stockés dans un **Log Analytics workspace**, interrogés en **KQL**.
- Les métriques internes à l'OS de la VM demandent l'**Azure Monitor Agent** et une **DCR** (Data Collection Rule). Voir aussi **VM insights**.

**Activity Log**
- Événements du **plan de contrôle** (qui a créé ou supprimé quoi), au niveau de l'abonnement, conservés **90 jours**.
- Pour les garder plus longtemps : *diagnostic settings* vers Log Analytics, un storage account ou un Event Hub.
- ⚠️ Les logs **des ressources** ne sont pas collectés par défaut : il faut configurer des **diagnostic settings** sur chaque ressource.

**Alertes**
- Une **alert rule** = scope + condition (métrique, log ou activity log) + **action group** (email, SMS, webhook, runbook, Logic App).
- **Alert processing rules** : supprimer ou rediriger des alertes, par exemple pendant une maintenance.
- Sévérité de 0 (critique) à 4.

**Network Watcher** (activé automatiquement par région)
| Outil | Usage |
|---|---|
| **IP flow verify** | Quelle règle NSG autorise ou bloque un flux |
| **Next hop** | Où part le trafic (diagnostic d'UDR) |
| **Effective security rules** | NSG réellement appliqués sur une NIC |
| **Connection troubleshoot / Connection monitor** | Test ponctuel ou continu d'une connexion |
| **Packet capture** | Capture réseau (extension sur la VM) |
| **VNet flow logs** + **Traffic Analytics** | Journal des flux (remplace les NSG flow logs) + analyse dans Log Analytics |

```bash
az monitor log-analytics workspace create -g labo-rg1 -n law-lab
az monitor action-group create -g labo-rg1 -n ag-mail --action email admin <mail>
az monitor metrics alert create -g labo-rg1 -n cpu-high --scopes <vm-id> \
  --condition "avg Percentage CPU > 80" --action ag-mail
az network watcher test-ip-flow -g labo-rg1 --vm vm0 --direction Inbound --protocol TCP \
  --local 10.10.10.4:3389 --remote <mon-ip>:50000
az network watcher show-next-hop -g labo-rg1 --vm vm0 --source-ip 10.10.10.4 --dest-ip 8.8.8.8
```

```kusto
// KQL : VMs avec CPU moyen > 80 % sur la dernière heure
Perf
| where TimeGenerated > ago(1h) and CounterName == "% Processor Time"
| summarize avg(CounterValue) by Computer
| where avg_CounterValue > 80
```

---

## Les 15 pièges les plus fréquents
1. RBAC Azure ≠ rôles Entra · control plane ≠ data plane.
2. Les groupes dynamiques, Conditional Access et SSPR demandent P1. PIM et ID Protection demandent P2.
3. Changer le tenant d'un abonnement → les RBAC sont perdus.
4. 5 IP réservées par subnet.
5. Peering **non transitif**, sans chevauchement, dans les deux sens.
6. NSG à deux niveaux : il faut que les deux autorisent.
7. NAT Gateway = sortant seulement, sur le subnet. Une IP publique va sur une NIC ou un LB, jamais sur un subnet.
8. Un availability set ne se choisit qu'à la création de la VM. Le disque temporaire est perdu au deallocate.
9. Archive = hors ligne, réhydratation nécessaire. Un failover géo → le compte repasse en LRS.
10. Object replication = versioning + change feed.
11. Une stored access policy permet de révoquer une SAS. Régénérer la clé tue toutes les SAS.
12. Bastion /26 `AzureBastionSubnet` · `GatewaySubnet` · `AzureFirewallSubnet`.
13. Tags non hérités · un lock s'impose même à l'Owner · un budget n'arrête rien.
14. Le vault Recovery Services est dans la même région que la VM, et sa redondance se fixe avant la 1re sauvegarde.
15. Les logs des ressources demandent des diagnostic settings. L'activity log est conservé 90 jours.
