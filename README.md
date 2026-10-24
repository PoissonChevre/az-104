# AZ-104: Microsoft Azure Administrator — CloudLee

[Course page](https://cloudleeio.teachable.com/p/az-104-microsoft-azure-administrator) · **Instructor:** James Lee (CloudLee) · **Status:** in progress (started October 2026)

## About

The AZ-104 exam validates the skills to manage an Azure environment: identities, governance, storage, compute, networking, monitoring and backup.

Exam domains (Microsoft study guide):

| Domain | Weight |
|---|---|
| Manage Azure identities and governance | 20–25 % |
| Implement and manage storage | 15–20 % |
| Deploy and manage Azure compute resources | 20–25 % |
| Implement and manage virtual networking | 15–20 % |
| Monitor and maintain Azure resources | 10–15 % |

## Course progress

- [x] 1. Entra ID Essentials
- [x] 2. Identity and Access Management (IAM)
- [x] 3. Entra ID Extended Services
- [ ] 4. Virtual Networking (in progress)
- [ ] 5. Virtual Machines
- [ ] 6. Storage
- [ ] 7. Advanced Networking
- [ ] 8. App Hosting (App Service)
- [ ] 9. Container Hosting
- [ ] 10. Load Balancing and Connectivity
- [ ] 11. Governance and Security
- [ ] 12. Backup and Site Recovery
- [ ] 13. Monitoring
- [ ] Exam

## Repository content

One note per chapter: key points, exam traps (⚠️) and commands.

| Folder / file | Content |
|---|---|
| [notes/01-entra-id-essentials.md](./notes/01-entra-id-essentials.md) | Tenant, domains, users, groups, administrative units, app and managed identities |
| [notes/02-identity-access-management.md](./notes/02-identity-access-management.md) | Azure RBAC vs Entra roles, key roles, custom roles |
| [notes/03-entra-id-extended-services.md](./notes/03-entra-id-extended-services.md) | Hybrid identity, B2B/B2C, Conditional Access, SSPR, governance, licenses, quiz |
| [notes/04-virtual-networking.md](./notes/04-virtual-networking.md) | VNets, subnets, IP addresses, NAT Gateway, NSG, ASG |
| [notes/lab-commands.md](./notes/lab-commands.md) | General Azure CLI commands for the labs |
| [templates/](./templates/) | ARM templates to deploy lab VMs (based on [CloudLee click-to-deploy](https://github.com/jamesdplee/cloudlee-click2deploy)) |
| [images/study-topics-mindmap.png](./images/study-topics-mindmap.png) | Mind map of the exam topics |

## Study method

After each chapter: CloudLee quiz + one lab with the Azure CLI. Then the Microsoft Practice Assessment. Only work again on topics under 80 %.

## Resources

- [CloudLee: AZ-104 course](https://cloudleeio.teachable.com/p/az-104-microsoft-azure-administrator)
- [Microsoft: AZ-104 study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104)
- [Microsoft: Azure Administrator Associate certification](https://learn.microsoft.com/en-us/credentials/certifications/azure-administrator/)
- [CloudLee click-to-deploy templates (GitHub)](https://github.com/jamesdplee/cloudlee-click2deploy)
- [John Savill: AZ-104 Study Cram (YouTube)](https://www.youtube.com/watch?v=0Knf9nub4-k)
- [Tutorials Dojo: Azure cheat sheets](https://tutorialsdojo.com/microsoft-azure-cheat-sheets/)
- [Tutorials Dojo: AZ-104 study guide eBook](https://portal.tutorialsdojo.com/product/tutorials-dojo-study-guide-ebook-az-104-microsoft-azure-administrator/)
