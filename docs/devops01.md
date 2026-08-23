# devops01

## Zweck

`devops01` ist die zentrale Linux- und DevOps-Arbeits-VM des Homelabs.

Von dieser VM aus werden GitHub, Azure, Terraform, Docker und später Kubernetes verwaltet.

## VM-Konfiguration

- Proxmox VM-ID: 107
- Betriebssystem: Ubuntu Server 24.04.4 LTS
- CPU: 2 vCPU
- Arbeitsspeicher: 4 GB
- Festplatte: 80 GB auf `sn850-vm`
- Firmware: UEFI/OVMF
- Machine Type: Q35
- QEMU Guest Agent: installiert

## Netzwerk

- Bridge: `vmbr1`
- VLAN: 10
- Aktuelle DHCP-Adresse: `10.0.10.102/24`
- Gateway: `10.0.10.1`
- Kein direkter Zugriff aus dem Internet

## Aktueller Stand

- Ubuntu installiert und aktualisiert
- SSH-Zugriff funktioniert
- Git installiert
- GitHub-Zugriff per SSH-Schlüssel eingerichtet
- Repository `homelab-infra` lokal geklont

## Geplanter Einsatz

1. Git und Linux
2. Azure CLI und PowerShell
3. Terraform
4. Docker und Compose
5. CI/CD
6. kubectl und Helm
7. Bash und Python
8. Ansible und ArgoCD
