# Homelab – Architektur & Technologie-Stack

---

## 1. Virtualisierung & Hardware

### **Proxmox VE**
- Virtualisierung von KVM- und LXC-Instanzen
- Zentrale Verwaltung aller VMs
- Lokaler SSD-Speicher für schnelle IO

### **Netzwerkhardware**
- Managed Switch (Layer 2) mit VLAN-Support
- pfSense Firewall für Routing, DHCP und Sicherheitsregeln

---

## 2. Netzwerkdesign

### VLANs
- **VLAN 10 – Server**  
  Infrastruktur- und Automationssysteme
- **VLAN 20 – Monitoring**  
  Monitoring- und Observability-Dienste
- **VLAN 30 – IoT**  
  Sensoren, MQTT und IoT-Kommunikation

### Firewall-Regeln
- Strikte Trennung der Netzwerksegmente
- Nur explizit erlaubte Inter-VLAN-Kommunikation
- Eigene DHCP-Konfiguration pro VLAN
- NAT- und Portweiterleitungen für interne Tests

---

## 3. Virtuelle Maschinen & Services

### Monitoring-VM
- **Virtualisierung:** KVM
- **Betriebssystem:** Ubuntu Server 22.04 LTS
- **Netzwerk:** VLAN 20 – Monitoring
- **Rolle:** Docker-Host für Observability

**Services:**
- Grafana – Dashboards und Visualisierung  
- InfluxDB – Zeitreihen-Datenbank  
- Prometheus – System- und Service-Metriken  
- Node-RED – Automatisierungs- und Logikflows  

---

### IoT-VM / LXC
- **Virtualisierung:** LXC 
- **Betriebssystem:** Debian 12
- **Netzwerk:** VLAN 30 – IoT
- **Rolle:** IoT-Datenaufnahme und Weiterleitung

**Services:**
- Mosquitto MQTT-Broker
- Verarbeitung und Weiterleitung von Sensordaten (ESP32, DHT22, Reed-Sensoren)

---

### Automation-VM
- **Virtualisierung:** KVM
- **Betriebssystem:** Ubuntu Server 22.04 LTS
- **Netzwerk:** VLAN 10 – Server
- **Rolle:** Infrastructure as Code & Deployment

**Services:**
- Ansible für Infrastructure as Code  
- Git für Versionskontrolle  
- Playbooks für Systeminstallation, Baseline-Konfiguration und Service-Deployments  
- CI/CD-Anbindung (GitHub/GitLab)

---

## 4. Automatisierung (Infrastructure as Code)

- Provisionierung neuer Systeme über Ansible
- Verwendung von Rollen für:
  - Baseline (Benutzer, Updates, SSH, Security-Hardening)
  - Docker-Hosts
  - Monitoring-Dienste
- Reproduzierbare Deployments durch Versionskontrolle in Git und CI/CD

---

## 5. IoT-Integration

### Hardware
- ESP32
- ESP32-CAM
- DHT22-Sensoren
- Reed-Sensoren

### Datenfluss
**MQTT → Node-RED → InfluxDB → Grafana**

Sensorwerte werden in Echtzeit verarbeitet, gespeichert und visualisiert.
