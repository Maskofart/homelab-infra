# Homelab – Architektur & Technologie-Stack

## Status

Dieses Homelab befindet sich in aktiver Weiterentwicklung.

Der aktuelle Stand bildet eine segmentierte Infrastruktur mit Active Directory, Monitoring, IoT-Integration und Automatisierung ab.  
Die Architektur folgt einem Default-Deny-Prinzip mit expliziten Inter-VLAN-Freigaben.

Weitere Härtungsmaßnahmen und Optimierungen sind geplant (siehe Roadmap).

---

# 1. Virtualisierung & Hardware

## Proxmox VE

- Zentrale Virtualisierungsplattform (KVM & LXC)
- SSD/NVMe-basierter Primär-Storage
- Management-Zugriff ausschließlich über VLAN 99
- Proxmox-Webinterface nicht im Server-VLAN erreichbar

## Netzwerk

- Managed Layer-2 Switch mit VLAN-Support
- pfSense als:
  - Firewall
  - Router (Inter-VLAN-Routing)
  - DHCP-Server
  - DNS-Resolver
  - NTP-Server
- pfSense-Management ausschließlich aus VLAN 99 erreichbar
- Switch-Management-IP im VLAN 99

---

# 2. Netzwerkdesign

## VLAN-Struktur

- VLAN 10 – Server  
  Windows- und Linux-Server, Automation-VM

- VLAN 20 – Observability  
  Monitoring-, Metrics- und Visualisierungsdienste

- VLAN 30 – IoT  
  Sensoren und MQTT-Kommunikation

- VLAN 99 – Management  
  Administrativer Zugriff (Admin-PC, Proxmox, pfSense, Switch)

---

## Firewall-Konzept (Default-Deny)

Inter-VLAN-Kommunikation ist standardmäßig blockiert.  
Freigaben erfolgen explizit, hostbasiert und portbasiert.

### Management

VLAN 99 → Infrastruktur

- TCP 443 (Webinterfaces)
- TCP 22 (SSH)
- TCP 3389 (RDP)
- TCP 8006 (Proxmox)

Nur definierte Admin-IP erlaubt.

---

### DNS & DHCP

Alle VLANs → pfSense Gateway-IP

- UDP/TCP 53 (DNS)
- UDP 67/68 (DHCP)

Domain-Mitglieder nutzen primär DC01 als DNS.  
DC01 forwardet externe Anfragen an pfSense.

---

### Pull-Monitoring

VLAN 20 → VLAN 10

- TCP 6556 (Checkmk Agent)
- TCP 9100 (Node Exporter)

Monitoring-VM initiiert die Verbindung.  
Stateful Firewall erlaubt Antwortverkehr automatisch.

---

### SNMP-Monitoring

VLAN 20 → VLAN 99 (Management-IPs)

- UDP 161 (SNMP)

Überwachte Systeme:
- pfSense
- Switch

SNMP ist read-only konfiguriert.

---

### MQTT

VLAN 20 → VLAN 30

- TCP 1883 (MQTT Broker)

Node-RED fungiert als Client.  
Broker läuft isoliert im IoT-VLAN.

MQTT-Authentifizierung aktiviert.  
TLS (8883) optional in Roadmap.

---

### Automatisierung (Ansible)

Automation-VM (VLAN 10) → definierte Hosts

- VLAN 10 → TCP 22
- VLAN 20 → TCP 22
- VLAN 30 → TCP 22

Nur Automation-VM-IP erlaubt.

---

# 3. DNS-Konzept

- DC01 ist autoritativer DNS-Server für die AD-Domain
- Domain-Mitglieder nutzen primär DC01 als DNS
- DC01 forwardet Internet-Anfragen an pfSense
- Linux- und IoT-Systeme nutzen pfSense als Resolver
- Conditional Forwarding in pfSense für AD-Domain

Damit bleiben AD-SRV-Records konsistent.

---

# 4. Virtuelle Maschinen & Services

## DC01 (VLAN 10)

- Windows Server 2022
- Active Directory Domain Services
- DNS (autoritativer AD-DNS)
- Gruppenrichtlinienverwaltung

Single Domain Controller (Lab-Setup).  
Kein Redundanz-DC implementiert.

---

## Monitoring-VM (VLAN 20)

Ubuntu Server 22.04 – Docker-Host

Container:

- Checkmk (Infrastructure Monitoring & Alerting)
- Prometheus (Metrics Collection)
- InfluxDB (Zeitreihen-Datenbank)
- Grafana (Visualisierung)
- Node-RED (IoT-Datenverarbeitung)

Checkmk ist primäre Alert-Instanz.  
Prometheus dient primär zur Metriksammlung.

---

## IoT-VM (VLAN 30)

- Debian 12 (unprivileged LXC)
- Mosquitto MQTT Broker
- Anonymous Access deaktiviert
- Passwortbasierte Authentifizierung

---

## Automation-VM (VLAN 10)

- Ubuntu Server 22.04
- Ansible
- Git
- SSH-Key-basierte Authentifizierung
- Playbooks für:
  - Baseline-Konfiguration
  - Monitoring-Agent-Deployment
  - Docker-Services

---

# 5. Windows Domain Infrastructure

## OU-Struktur

- Users
- Groups
- Workstations
- Servers
- ServiceAccounts

---

## AGDLP-Modell

Accounts → Global → Domain Local → Permissions


User  
→ GG_IT  
→ DL_RDP_Server  
→ RDP-Berechtigung

Single-Domain-Umgebung, daher kein AGUDLP erforderlich.

NTFS-Rechte werden ausschließlich an Domain-Local-Gruppen vergeben.  
Keine direkten Benutzerberechtigungen auf Ressourcen.

---

## Gruppenrichtlinien

- Passwort-Policy (Domain-Level)
- Account-Lockout-Policy
- Windows-Firewall-Baseline
- Bildschirmsperre (Client-OU)
- Administrative Richtlinien pro OU (Server vs. Workstations getrennt)

---

# 6. Monitoring & Observability

## Checkmk

Überwachung von:

- Proxmox
- Windows Server
- Linux Server
- pfSense (SNMP)
- Switch (SNMP)

Pull-Verfahren.  
Zentrale Alert-Instanz.

---

## Prometheus

- Node Exporter
- Docker Metrics
- Application-Metriken

Keine eigenständige Alertmanager-Integration (derzeit).

---

## InfluxDB

Speichert:

- IoT-Sensordaten
- Langzeitmetriken

---

## Grafana

Visualisiert Daten aus:

- Prometheus
- InfluxDB

---

## Node-RED

- MQTT-Client
- Daten-Transformation
- Speicherung in InfluxDB

---

# 7. Backup & Recovery

## Aktueller Stand

- Externes Backup-Storage eingerichtet
- Manuelle Test-Backups durchgeführt
- Restore-Test validiert
- Snapshots nur für kurzfristige Tests

## Zielkonzept

- Tägliche automatisierte Proxmox-Backups
- Retention:
  - 7 tägliche Generationen
  - 4 wöchentliche Generationen

## Zielwerte

- RPO: 24 Stunden
- RTO: < 2 Stunden (Lab-Ziel)

---

# 8. Linux-User-Management

Linux-Server sind bewusst nicht AD-joined.

Zentrale Verwaltung über:

- SSH-Key-Authentifizierung
- Ansible-Deployment von authorized_keys
- Public Keys versioniert im Git
- Private Keys lokal gespeichert
- PasswordAuthentication deaktiviert

Optionale AD-Integration via SSSD in Roadmap.

---

# 9. Automatisierung

- Ansible-Playbooks versioniert in Git
- Manuelle Ausführung
  
---

# 10. IoT-Datenfluss

Sensor  
→ MQTT Broker (VLAN 30)  
→ Node-RED (VLAN 20)  
→ InfluxDB  
→ Grafana

Kommunikation ausschließlich über definierte Ports.

---

# Roadmap

- Weitere Firewall-Granularität (Host- statt Netz-Regeln)
- Backup-Automatisierung & Retention-Validierung
- Zweiter Domain Controller (Redundanz)
- MQTT TLS (8883) Integration
- Optionale Alertmanager-Integration für Prometheus
- MFA für Management-Zugriffe
