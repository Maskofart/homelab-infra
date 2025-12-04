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

### **VLANs**
- **VLAN 10 – Server**
- **VLAN 20 – Monitoring**
- **VLAN 30 – IoT**

### **Firewall-Regeln**
- Strikte Trennung der Netzwerksegmente
- Nur definierte Inter-VLAN-Kommunikation
- DHCP-Konfiguration pro VLAN
- NAT/Portweiterleitungen für interne Tests

---

## 3. Virtuelle Maschinen & Services

### **Monitoring-VM (Docker-Host)**
- **Grafana** – Dashboards und Visualisierung  
- **InfluxDB** – Zeitreihen-Datenbank  
- **Prometheus** – System- und Service-Metrics  
- **Node-RED** – Automatisierungs- und Logikflows  

### **IoT-VM**
- **Mosquitto MQTT-Broker**
- Verarbeitung und Weiterleitung von Sensordaten (ESP32, DHT22, Reed-Sensoren)

### **Automation-VM**
- **Ansible** für Infrastructure as Code  
- **Git** für Versionskontrolle  
- Playbooks für Systeminstallation, Baseline-Konfiguration und Service-Deployments  
- CI/CD-Anbindung (GitHub/GitLab)

---

## 4. Automatisierung (Infrastructure as Code)

- Provisionierung neuer Systeme über Ansible
- Rollen für:
  - Baseline (Benutzer, Updates, SSH, Hardening)
  - Docker-Hosts
  - Monitoring-Dienste
- Reproduzierbare Deployments durch Versionierung in Git und CI/CD

---

## 5. IoT-Integration

### **Hardware**
- ESP32
- ESP32 - cam
- DHT22-Sensoren
- Reed-Sensoren

### **Datenfluss**
**MQTT → Node-RED → InfluxDB → Grafana**

Sensorwerte werden in Echtzeit verarbeitet und visualisiert.
