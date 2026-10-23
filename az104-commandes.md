# AZ-104 – Commandes essentielles (labs)

```bash
# ── Resource group ──
az group create -n labo-rg1 -l belgiumcentral      # Crée le RG (sa région = celle des ressources)
az group delete -n labo-rg1 --yes --no-wait        # Supprime tout en fin de lab

# ── Templates ARM / Bicep ──
az deployment group create -g labo-rg1 --template-file template-win-client.json
az deployment group create -g labo-rg1 --template-file template-win-client.json \
  --parameters createPublicIp=true                 # Surcharge un paramètre
az group export -n labo-rg1 > export.json          # Exporte l'existant en ARM
az bicep decompile --file template.json            # ARM JSON → Bicep

# ── Tailles & quotas ──
az vm list-usage -l belgiumcentral -o table        # Quotas vCPU par famille (0 = inutilisable)
az vm image list-skus -l belgiumcentral \
  -p MicrosoftWindowsDesktop -f windows-11 -o table  # Images Windows 11 dispo

# ── VM ──
az vm deallocate -g labo-rg1 -n vm0                # Arrêt + fin de facturation compute
az vm start -g labo-rg1 -n vm0

# ── IP publique sur la NIC (RDP) ──
az network public-ip create -g labo-rg1 -n vm0-ip --sku Standard
az network nic ip-config update -g labo-rg1 --nic-name vm0-nic -n ipconfig1 \
  --public-ip-address vm0-ip                       # Attache l'IP à la NIC
az network nic ip-config update -g labo-rg1 --nic-name vm0-nic -n ipconfig1 \
  --remove publicIpAddress                         # La détache

# ── NAT Gateway (sortant uniquement, au niveau du subnet) ──
az network public-ip create -g labo-rg1 -n nat-pip --sku Standard
az network nat gateway create -g labo-rg1 -n natgw1 --public-ip-addresses nat-pip
az network vnet subnet update -g labo-rg1 --vnet-name vnet1 -n subnet1 --nat-gateway natgw1

# ── RDP depuis Linux ──
xfreerdp /v:<ip> /u:azureuser /dynamic-resolution /cert:ignore
```

**À retenir**
- Région du déploiement = région du **resource group**.
- `SkuNotAvailable` = pas de capacité dans la région · `QuotaExceeded` = quota de la famille atteint.
- IP publique → sur la **NIC** · NAT Gateway → sur le **subnet**, sortant seulement.
- `deallocate` stoppe la facturation, un arrêt depuis Windows non.
