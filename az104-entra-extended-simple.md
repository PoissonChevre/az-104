# AZ-104 – Entra ID Extended Services (version simple)

> Le chapitre en langage simple. ⚠️ = piège d'examen.
> Image de base : ton entreprise a **un annuaire au bureau (AD)** et **un annuaire dans le cloud (Entra ID)**.

---

## 1. Entra Connect — relier le bureau au cloud

**Le problème** : l'entreprise a déjà ses comptes dans l'AD du bureau. Elle ne veut pas tout recréer à la main dans le cloud.

**La solution** : Entra Connect **copie les comptes du bureau vers le cloud**, toutes les 30 minutes. L'employé garde le même identifiant et le même mot de passe partout.

**Comment le mot de passe est vérifié ?** 3 façons :
| Méthode | En clair |
|---|---|
| **PHS** (Password Hash Sync) | Une version brouillée du mot de passe est copiée dans le cloud. Le cloud vérifie seul. **Le plus simple, recommandé.** Marche même si le bureau est en panne. |
| **PTA** (Pass-Through Auth) | Le cloud demande à chaque fois au bureau « ce mot de passe est bon ? ». Si le bureau est en panne → plus de connexion. |
| **Fédération** (AD FS) | Le bureau gère toute la connexion. Lourd et complexe. |

- **Cloud Sync** = la version légère d'Entra Connect.
- **Writeback** = le sens inverse, du cloud vers le bureau. Sert surtout pour le mot de passe changé via SSPR (voir §10).
- ⚠️ Un compte copié depuis le bureau **se modifie au bureau**, pas dans le portail Azure.

---

## 2. Entra Domain Services — un « AD tout fait » dans Azure

**Le problème** : tu déplaces une vieille application dans Azure. Elle a besoin d'un AD classique (domaine, GPO, Kerberos…), mais Entra ID ne sait pas faire ça.

**La solution** : Microsoft te loue un **AD prêt à l'emploi**, qu'il entretient lui-même. Tu n'as pas de serveur contrôleur de domaine à gérer.

- ⚠️ Il lui faut **son propre subnet**.
- ⚠️ Tu n'es **pas Domain Admin**. Tu as seulement le groupe *AAD DC Administrators*.
- ⚠️ Les comptes créés dans le cloud doivent **changer leur mot de passe une fois** avant de pouvoir s'y connecter.

**À ne pas confondre** : Entra Connect **copie** ton AD existant. Domain Services te **fournit** un AD neuf dans Azure.

---

## 3. External Identities — faire entrer des gens de l'extérieur

### B2B = des partenaires dans TON annuaire
- Tu invites un consultant ou un fournisseur. Il garde **son propre compte** (celui de son entreprise, Gmail, Microsoft…).
- Chez toi, il apparaît comme **Guest** (invité), avec `#EXT#` dans son identifiant.
- Tu choisis qui a le droit d'inviter, et quels domaines sont autorisés ou bloqués : *External collaboration settings*.
- Il peut se connecter avec son **compte Google** (fédération Google, démo du cours).

### B2C / External ID = les clients de ton appli
- Tu fais un site ou une appli grand public : les **clients** créent un compte ou se connectent avec Google/Facebook.
- Ça se passe dans un **annuaire séparé**, pas dans celui des employés.

⚠️ **Retenir** : B2B = **partenaires**, dans ton annuaire · B2C = **clients**, dans un annuaire à part.

---

## 4. Governance — qui a accès à quoi, et pour combien de temps

Les 4 outils suivants servent à éviter que les gens gardent des accès dont ils n'ont plus besoin. ⚠️ Ils demandent tous la licence **P2** (ou *Entra ID Governance*).

---

## 5. Entitlement Management — le « kit d'accès »

**Le problème** : un nouvel employé ou un consultant a besoin de 5 groupes, 2 applis et un site SharePoint. Tout donner à la main, puis tout retirer à la fin, c'est long et on oublie.

**La solution** : on prépare un **Access Package**, un « kit d'arrivée » qui regroupe tout.
- La personne le **demande**, un responsable **approuve**, et elle reçoit tout d'un coup.
- Le kit a une **date d'expiration** : tout est retiré automatiquement à la fin.
- Les kits sont rangés dans un **Catalog** (un classeur).
- Ça marche aussi pour des externes (B2B).

---

## 6. PIM — les droits d'admin « sur demande »

**Le problème** : quelqu'un qui est Global Admin 24h/24, c'est dangereux. Si son compte est piraté, le pirate a tous les pouvoirs.

**La solution** (PIM = *Privileged Identity Management*) : la personne est admin **seulement quand elle en a besoin**.
- **Eligible** = elle *peut* devenir admin, mais ne l'est pas par défaut.
- Quand elle en a besoin, elle **active** le rôle : pour un temps limité (ex. 2 h), avec MFA, une justification et parfois l'accord d'un chef.
- **Active** = le rôle est donné en permanence (le mode classique).

Image : c'est une **clé qu'on emprunte** à l'accueil et qu'on rend, au lieu d'avoir la clé sur soi tout le temps.

---

## 7. Access Reviews — le grand ménage régulier

**Le problème** : avec le temps, les gens accumulent des accès qu'ils n'utilisent plus.

**La solution** : tous les X mois, on demande à quelqu'un (le chef, le propriétaire du groupe, ou la personne elle-même) : « Cette personne a-t-elle **encore besoin** de cet accès ? »
- Si la réponse est **non**, ou s'il n'y a pas de réponse, l'accès peut être **retiré automatiquement**.
- Ça s'applique aux groupes, aux applis et aux rôles admin.

---

## 8. ID Protection — le détecteur de connexions suspectes

**Le principe** : Microsoft surveille les connexions et repère ce qui est bizarre.
- **Sign-in risk** = *cette connexion* est louche. Exemple : connexion depuis Paris puis depuis le Japon 10 minutes plus tard, ou depuis un réseau de pirates.
- **User risk** = *ce compte* est probablement piraté. Exemple : son mot de passe a été trouvé dans une fuite sur Internet.

**Ce qu'on fait** : on crée des règles. Si le risque est moyen → demander la MFA. Si le compte est à risque → forcer le changement de mot de passe, ou bloquer.

---

## 9. Conditional Access — le videur à l'entrée

**Le principe** : une règle **SI… ALORS…**.
- **SI** : qui (user/groupe), quelle appli, d'où (pays, adresse IP), quel appareil, quel niveau de risque…
- **ALORS** : bloquer, ou laisser passer **à condition** de faire la MFA, d'avoir un PC de l'entreprise, etc.

Exemples :
- « Les admins doivent **toujours** faire la MFA. »
- « Personne ne se connecte **depuis l'étranger**. »
- « Pour l'appli RH : seulement depuis un **PC de l'entreprise**. »

- ⚠️ Licence **P1**.
- ⚠️ Il faut **désactiver les Security defaults** avant (la protection de base gratuite de Microsoft).
- Mode **Report-only** : la règle ne bloque rien, elle note juste ce qu'elle *aurait* fait. Pour tester sans se bloquer soi-même.

---

## 10. SSPR — « j'ai oublié mon mot de passe »

**Le principe** : l'employé réinitialise **lui-même** son mot de passe, sans appeler le support.
1. Il clique sur « Mot de passe oublié ? ».
2. Il prouve que c'est lui (code par SMS, appli Authenticator, mail…).
3. Il choisit un nouveau mot de passe.

- ⚠️ Licence **P1**.
- Activé pour : **personne / un groupe / tout le monde**.
- **1 ou 2** preuves demandées.
- L'utilisateur doit d'abord **enregistrer** son téléphone ou son mail.
- Comptes venant du bureau → il faut le **writeback** pour que le nouveau mot de passe marche aussi au bureau.

---

## 11. App Proxy — ouvrir une appli du bureau vers l'extérieur, sans danger

**Le problème** : l'entreprise a une appli web interne (intranet, outil RH) sur un serveur au bureau. Les employés en télétravail veulent y accéder. Ouvrir un port vers Internet, c'est risqué.

**La solution** :
- On installe un petit programme, le **connector**, sur un serveur du bureau.
- Ce connector **appelle lui-même** Microsoft (connexion sortante) : **aucun port à ouvrir** vers l'intérieur.
- L'employé passe par une adresse Microsoft, se connecte avec son compte Entra (et la MFA ou le Conditional Access si on veut), puis arrive sur l'appli.

- ⚠️ Licence **P1**.
- Image : au lieu d'ouvrir la porte du bureau, c'est un **coursier** du bureau qui va chercher les visiteurs à la gare.

---

## Récap licences

| **Free** (gratuit) | **P1** | **P2** |
|---|---|---|
| Users, groupes simples, B2B, Security defaults | Conditional Access, SSPR (+ writeback), App Proxy, groupes dynamiques | PIM, ID Protection, Access Reviews, Entitlement Management (Access Packages) |

**Astuce** : P2 = tout ce qui parle de **risque** ou de **gouvernance** (accès temporaires, revues, kits).

---

## Quel outil pour quelle situation ? (type examen)

| Situation | Réponse |
|---|---|
| Les comptes de l'AD du bureau doivent exister dans le cloud | **Entra Connect** (PHS de préférence) |
| Une vieille appli dans Azure a besoin d'un domaine, LDAP, Kerberos | **Entra Domain Services** |
| Un consultant externe doit accéder à une appli avec son propre compte | **B2B** (invité Guest) |
| Les clients d'une appli doivent créer un compte | **B2C / External ID** |
| Donner un lot d'accès qui expire automatiquement | **Access Package** (Entitlement Management) |
| Un admin ne doit avoir ses droits que quand il en a besoin | **PIM** (rôle *eligible*) |
| Vérifier chaque trimestre qui a encore besoin d'un accès | **Access Review** |
| Bloquer un compte dont le mot de passe a fuité | **ID Protection** (user risk) |
| Forcer la MFA hors du bureau | **Conditional Access** |
| Les users doivent changer leur mot de passe seuls | **SSPR** |
| Le mot de passe changé par SSPR ne marche pas au bureau | Activer le **writeback** |
| Accéder à l'intranet du bureau depuis chez soi sans VPN ni port ouvert | **App Proxy** |
