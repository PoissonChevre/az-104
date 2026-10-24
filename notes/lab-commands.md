# Lab commands

General Azure CLI commands for the labs. The commands of each topic are in its chapter file.

```bash
# Resource group
az group create -n labo-rg1 -l belgiumcentral      # Templates deploy in the RG region by default
az group delete -n labo-rg1 --yes --no-wait        # Delete everything at the end of the lab

# Templates (ARM / Bicep)
az deployment group create -g labo-rg1 --template-file templates/win-client-full.json
az deployment group create -g labo-rg1 --template-file templates/win-client-full.json \
  --parameters createPublicIp=true                 # Override a parameter
az group export -n labo-rg1 > export.json          # Export what exists as ARM
az bicep decompile --file template.json            # ARM JSON → Bicep

# Sizes and quotas
az vm list-usage -l belgiumcentral -o table        # vCPU quotas per VM family (0 = can't use it)
az vm image list-skus -l belgiumcentral \
  -p MicrosoftWindowsDesktop -f windows-11 -o table  # Windows 11 images available

# VM
az vm deallocate -g labo-rg1 -n vm0                # Stop + no more compute billing
az vm start -g labo-rg1 -n vm0

# RDP from Linux
xfreerdp /v:<ip> /u:azureuser /dynamic-resolution /cert:ignore
```

**Remember**

- `SkuNotAvailable` = no capacity in this region · `QuotaExceeded` = VM family quota reached.
- `deallocate` stops compute billing. A shutdown from inside Windows does not.
